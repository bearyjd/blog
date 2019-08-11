# Overview  

我们都知道 linux kernel 启动完成以后，会启动 /sbin/initd 作为第一个用户空间的进程。同样地，OpenWrt 也遵循这个规则，不同的是 OpenWrt 的主服务器进程是 procd，而 procd 是系统启动过程由 initd 切换过来的。 
<!--more-->


# Checkout source  

使用以下命令克隆 procd 源码到本地：  
```bash  
# git clone git://git.openwrt.org/project/procd.git
```   


# Initd process  

initd 进程是在内核启动完成后启动的第一个用户空间进程，它的主程序是 init.c，主要做了以下的一些工作：  

1. mount filesystem, proc & sysfs & cgroup & tmpfs & devpts  
2. create essential temporary directory, /tmp/run & /tmp/lock & /tmp/state  
3. set PATH environment, PATH="/usr/sbin:/sbin:/usr/bin:/bin"  
4. set debug level by finding key is "init_debug" from /proc/cmdline  
5. register timeout callback, which call watchdog  
6. fork one child process to execute "/sbin/kmodloader /etc/modules-boot.d/", which will load kernel modules, and parent process continue  
7. process management: procd and preinit. For procd, fork a child process to run as hotplug daemon with command "/sbin/procd -h /etc/hotplug-preinit.json", the parent process register plugd_proc to uloop_process. For preinit, fork a child process to execute "/bin/sh /etc/preinit", and the parent process register preinit_proc to uloop_process  
8. run loop  

<font color="#FF0000">注意：这里的第7点，用到了 uloop 的进程管理。对于调用者，只需要关心 struct uloop_process 的 pid 和 cb 成员，其中，cb 是回调，当子进程退出时，父进程会收到 SIGCHLD 信号而调用。通过 uloop_process_add 把 struct uloop_process 添加到 uloop 事件循环中。</font>  


# Initd to Procd  

我们透过简化代码分析 initd 进程是如何切换到 procd 进程：    
```c  
/* global various */
static struct uloop_process preinit_proc;

void preinit(void)
{
    char *init[] = { "/bin/sh", "/etc/preinit", NULL };

    preinit_proc.cb = spawn_procd;
	preinit_proc.pid = fork();
	if (!preinit_proc.pid) {
		execvp(init[0], init);
		ERROR("Failed to start preinit\n");
		exit(-1);
	}
	if (preinit_proc.pid <= 0) {
		ERROR("Failed to start new preinit instance\n");
		return;
	}
	uloop_process_add(&preinit_proc);
}
```  

根据上面提到 uloop 的进程管理可知，一旦执行完 `/bin/sh /etc/preinit`，则会执行 spawn_procd 回调。spawn_procd 的代码段：  
```c  
static void spawn_procd(struct uloop_process *proc, int ret)
{
	char *argv[] = { "/sbin/procd", NULL};
	struct stat s;

	if (plugd_proc.pid > 0)
		kill(plugd_proc.pid, SIGKILL);

	if (!stat("/tmp/sysupgrade", &s))
		while (true)
			sleep(1);

	execvp(argv[0], argv);
}
```  

spawn_procd 作为 initd 进程中 uloop_run 的回调函数，其 pid = 1(initd)，通过执行 execvp 拉起一进程取代原进程，这里是 initd 进程被 /sbin/procd 进程取代。至此，procd 成为 OpenWrt 系统的主服务进程(pid = 1)。  


# Procd process  

TODO
