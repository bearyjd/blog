# Introduce  

OpenWrt 缺省控制 DHCP 的进程如下：  
- udhcpc - DHCP IPV4 client  
- odhcp6c - DHCP IPV6 client  
- DNSmasq - DHCP IPV4 server  
- odhcpd - DHCP IPV6 server  
<!--more-->

使用 ps 命令查看 udhcpc 运行参数情况：  
```bash  
# ps -w | grep udhcpc
```  

udhcpc 运行的参数类似这样：  
```bash  
 2339 root      1204 S    udhcpc -p /var/run/udhcpc-eth0.2.pid -s /lib/netifd/dhcp.script -f -t 0 -i eth0.2 -x hostname GL-MT300N-V  
 2674 root      1204 S    udhcpc -p /var/run/udhcpc-apcli0.pid -s /lib/netifd/dhcp.script -f -t 0 -i apcli0 -C -O 121  
```  

对于我的路由器，当前系统 apcli0 和 eth0.2 网络接口使用了 DHCP 协议，当有 DHCP 事件被触发，会执行 `/lib/netifd/dhcp.script` 脚本。  

# Udhcpc startup  

通过 grep 搜索，发现启动 udhcpc 的代码在 `/lib/netifd/proto/dhcp.sh` 脚本中，proto_dhcp_setup 函数代码段：  
```bash  
proto_run_command "$config" udhcpc \
         -p /var/run/udhcpc-$iface.pid \
         -s /lib/netifd/dhcp.script \
         -f -t 0 -i "$iface" \
         ${ipaddr:+-r $ipaddr} \
         ${hostname:+-x "hostname:$hostname"} \
         ${vendorid:+-V "$vendorid"} \
         $clientid $broadcast $release $dhcpopts
```  

这里调用了 proto_run_command 函数，同样地，搜索到它是 `/lib/netifd/netifd-proto.sh` 提供的脚本 api，代码段：  
```bash  
proto_run_command() {
    local interface="$1"; shift

    json_init
    json_add_int action 1
    json_add_array command
    while [ $# -gt 0 ]; do
        json_add_string "" "$1"
        shift
    done
    json_close_array
    [ -n "$_EXPORT_VARS" ] && {
        json_add_array env
        for var in $_EXPORT_VARS; do
            eval "json_add_string \"\" \"\${$var}\""
        done
        json_close_array
    }
    _proto_notify "$interface"
}
```  

proto_run_command 函数就是初始化了json object，往 json object 添加了 string 类型的 action 和 array 类型的 command 字段，把传进来的参数全部填充到 command array中，最后调用了_proto_notify。  

_proto_notify 函数代码段：  
```bash  
_proto_notify() {
    local interface="$1"
    local options="$2"
    json_add_string "interface" "$interface"
    ubus $options call network.interface notify_proto "$(json_dump)"
}  
```  

由此可以看出，最终使用 ubus 把 json 格式的内容发送到 ubusd 处理，network.interface 对象的 notify_proto 方法。  

我们可以通过命令行查看 network.interface 对象注册了哪些方法以及怎么使用：  
```bash  
# ubus -v list network.interface
```  

notify_proto 方法在背后做了哪些工作呢？我们可以通过 netifd 源码来寻找答案。notify_proto 的定义在 ubus.c 文件中，netifd 源码：  
```bash  
# git clone git://git.openwrt.org/project/netifd.git
```  

```c
static struct ubus_method iface_object_methods[] = {
	{ .name = "up", .handler = netifd_handle_up },
	{ .name = "down", .handler = netifd_handle_down },
	{ .name = "renew", .handler = netifd_handle_renew },
	{ .name = "status", .handler = netifd_handle_status },
	{ .name = "prepare", .handler = netifd_handle_iface_prepare },
	{ .name = "dump", .handler = netifd_handle_dump },
	UBUS_METHOD("add_device", netifd_iface_handle_device, dev_link_policy ),
	UBUS_METHOD("remove_device", netifd_iface_handle_device, dev_link_policy ),
	{ .name = "notify_proto", .handler = netifd_iface_notify_proto },
	{ .name = "remove", .handler = netifd_iface_remove },
	{ .name = "set_data", .handler = netifd_handle_set_data },
};  
```  

通过跟踪代码，调用顺序大致是这样：  
```bash  
netifd_iface_notify_proto --–>
  iface->proto->notify --->  
    回调的注册：
    state->proto.notify = proto_shell_notify; --->
      proto_shell_run_command --->  
        netifd_start_process --->  
          创建子进程调用execvp执行脚本命令  
```  

至此，我们可以知道，刚开始分析的 udhcpc 作为脚本 api proto_run_command 的传入参数，最终通过 ubus 通讯，由 netifd 执行 execvp 带起 udhcpc。  

# Example  

以重新获取 wan 口的 ip 地址为例演示上面的过程，`/lib/netifd/proto/dhcp.sh` 的 proto_dhcp_renew 函数可以重新租约 ip 地址，proto_dhcp_renew 函数代码段：  
```bash  
proto_dhcp_renew() {
    local interface="$1"
    # SIGUSR1 forces udhcpc to renew its lease
    local sigusr1="$(kill -l SIGUSR1)"
    [ -n "$sigusr1" ] && proto_kill_command "$interface" $sigusr1
}  
```  

proto_dhcp_renew 函数需要接收一个 interface( wan | wan6 | lan | wwan | ... ) 参数，重新租约 wan 口的 ip：  
```bash  
proto_dhcp_renew "wan"
```  

<font color="#FF0000">注意：这里只是为了演示过程，实际上不能直接调用的。</font>  

proto_dhcp_renew 调用了 proto_kill_command，代码段：  
```bash  
_proto_notify() {
    local interface="$1"
    local options="$2"
    json_add_string "interface" "$interface"
    ubus $options call network.interface notify_proto "$(json_dump)"  
} 

proto_kill_command() {
        local interface="$1"; shift
                         
        json_init                                          
        json_add_int action 2
        [ -n "$1" ] && json_add_int signal "$1"
        _proto_notify "$interface"
}   
```  

前面分析可知，这里是构造了一个 json 格式消息，然后通过调用 _proto_notify 把消息发送到 ubusd 处理，这里的 json_dump 内容为：  
```json  
{
    "interface": "wan",
    "action": 2,
    "signal": 16
}
```  

`proto_dhcp_renew "wan"` 其实等价于命令行执行：  
```bash  
# ubus call network.interface notify_proto '{"action":2,"signal":16,"interface":"wan"}'
```  

# Troubleshoot  

前面提到入口在 `/lib/netifd/proto/dhcp.sh` 脚本，我们分析一下这个脚本是怎样被执行的？  

netifd 主程序是 main.c，netifd 在启动的时候，调用了 proto_shell_init 函数。proto_shell_init 的实现代码以及分析注释：  
```c  
void proto_shell_init(void)
{
    /* 获取 /lib/netifd/proto 文件描述符 */
	proto_fd = netifd_open_subdir("proto");
	if (proto_fd < 0)
		return;

	netifd_init_script_handlers(proto_fd, proto_shell_add_handler);
}  
```  

netifd_init_script_handlers 的实现代码：  
```c  
void netifd_init_script_handlers(int dir_fd, script_dump_cb cb)
{
	glob_t g;
	int i, prev_fd;

	prev_fd = netifd_dir_push(dir_fd);
	if (glob("./*.sh", 0, NULL, &g))
		return;

	for (i = 0; i < g.gl_pathc; i++)
		netifd_parse_script_handler(g.gl_pathv[i], cb);
	netifd_dir_pop(prev_fd);

	globfree(&g);
}
```  

`netifd_parse_script_handler` 函数执行 `/lib/netifd/proto` 目录下的所有.sh文件，dump 作为传入参数。对于dhcp.sh，在脚本执行：  
```bash  
init_proto "dump"
add_protocol dhcp
```  

其中，init_proto 和 add_protocol 为 netifd-proto.sh 的函数接口。  

```bash  
add_protocol --->
  proto_dhcp_init_config
```  

并且构造一个 json 格式消息作为返回，netifd 的 `netifd_parse_script_handler` 函数将执行 dhcp.sh 脚本的返回值封装成 json 对象。  

```bash  
netifd_init_script_handlers --->
  netifd_parse_script_handler --->
    netifd_init_script_handler -->
      proto_shell_add_handler --->
        add_proto_handler --->
          avl_insert
```  

`proto_shell_add_handler` 是 `netifd_init_script_handlers` 注册的回调函数，在 `netifd_init_script_handler` 被调用。`proto_shell_add_handler` 函数把 json 解析到 struct proto_handler 对应的成员变量中，然后把它插入到 avl 树中。

