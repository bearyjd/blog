## Formatting with printf  

awk的格式化输出，和C语言的printf没什么区别。让我们来看一下以下的格式化输出示例：  

```  
echo 123.4567 | awk '{printf "%.3f\n", $1}'
123.457

# 不满8字符向左填充空格，并且只保留一位小数
echo 123.4567 | awk '{printf "%8.1f\n", $1}'
   123.5

# 第一个元素不满20字符向右填充空格，并且保留七位小数；第二个元素以整形输出
echo 123.4567 55.2 | awk '{printf "%-20.7f  %d\n" , $1 , $2}'
123.4567000           55
```  

Here is a list of the format-control letters:  

```  
`c'    This prints a number as an ASCII character. Thus, `printf "%c", 65' outputs the letter `A'. 
          The output for a string value is the first character of the string. 

`d'    This prints a decimal integer. 

`i'    This also prints a decimal integer. 

`e'    This prints a number in scientific (exponential) notation. 
           For example, printf "%4.3e", 1950  prints `1.950e+03', 
           with a total of four significant figures of which three follow the decimal point. 
           The `4.3' are modifiers, discussed below. 

`f'    This prints a number in floating point notation. 

`g'    This prints a number in either scientific notation or floating point notation, whichever uses fewer characters. 

`o'    This prints an unsigned octal integer. 

`s'    This prints a string. 

`x'    This prints an unsigned hexadecimal integer. 

`X'    This prints an unsigned hexadecimal integer. 
          However, for the values 10 through 15, it uses the letters `A' through `F' instead of `a' through `f'. 

`%'    This isn't really a format-control letter, but it does have a meaning when used after a `%': the sequence `%%' outputs one `%'. 
         It does not consume an argument. 
```  

## BEGIN/END  

在读取第一个输入记录之前，BEGIN规则仅执行一次。同样，在读取所有输入之后，END规则仅执行一次。例如： 

```  
# awk -F":" 'BEGIN{print "Show all users:"} {print $1} END{print "====> Successful!"}' /etc/passwd
Show all users:
root
daemon
ftp
network
nobody
dnsmasq
stubby
====> Successful!
```  

## Built-in Variables  

### FS Example: 输入字段分隔符变量

awk默认以空白字符（空格/tab）来解析从输入读取的每一行，并设置变量$1，$2等，FS变量用于设置字段分隔符，它可以设置为任何单个字符或正则表达式。我们可以通过以下两种方式来使用FS变量：  

1. 命令行使用-F选项；  
2. 直接设置FS变量；  

```  
Syntax:

awk -F'FS' 'commands' inputfilename

(or)

awk 'BEGIN{FS="FS";}'
``` 

这是一个awk FS示例，以":"作为分隔符处理 */etc/passwd* 文件。为了易读，这里使用了awk脚本的形式。  

```  
$ cat passwd.awk
# 处理前
BEGIN{
	FS=":";
	print "Name\tUserID\tGroupID\tHomeDirectory";
}
# 处理中
{
	print $1"\t"$3"\t"$4"\t"$6;
}
# 处理后
END {
	print NR,"Records Processed";
}
```  

```  
# awk -f passwd.awk /etc/passwd
Name	UserID	GroupID	Home
root	0	0	/root
daemon	1	1	/var
ftp	55	55	/home/ftp
network	101	101	/var
nobody	65534	65534	/var
dnsmasq	453	453	/var/run/dnsmasq
stubby	410	410	/var/run/stubby
7 Records Processed
```  

### OFS Example: 输出字段分隔符变量  

OFS是FS变量的输出等价物，默认值是空格。下面是使用OFS缺省值的示例：  

```  
# awk -F':' '{print $3,$4;}' /etc/passwd
0 0
1 1
55 55
101 101
65534 65534
453 453
410 410
```  
print语句中的","表示两个参数使用空格连接起来，也就是OFS的默认值。OFS的值将被插入到输出字段间，示例如下：  

```  
# awk -F':' 'BEGIN{OFS="=";} {print $3,$4;}' /etc/passwd
0=0
1=1
55=55
101=101
65534=65534
453=453
410=410
```  

### RS Example: 输入的记录分隔符变量  

RS定义了行，awk缺省情况是以换行符为分隔符一行一行读取的。  

让我们把软件包信息存到一个文件，每个软件信息以一个空行作为分割，每个字段以换行符分隔。  

```  
# cat opkg.info 
Package: iwinfo
Version: 2018-07-24-94b1366d-2
Depends: libc, libiwinfo
Status: install user installed
Architecture: mips_24kc
Installed-Time: 1553831708
 
Package: luci-mod-rpc
Version: git-18.196.56128-9112198-1
Depends: libc, luci-lib-json
Status: install user installed
Architecture: all
Installed-Time: 1553831708

Package: ubus                 
Version: 2018-01-16-5bae22eb-1
Depends: libc, libubus, libblobmsg-json, ubusd
Status: install ok installed
Architecture: mips_24kc
Installed-Time: 1553831711
Auto-Installed: yes
```  

以下是使用RS变量打印软件包名称的示例：  

```  
# cat opkg.awk 
BEGIN {
	RS="";
	FS=" ";
}
{
	print $2;
}

# awk -f opkg.awk opkg.info 
iwinfo
luci-mod-rpc
ubus
```  

在opkg.awk脚本中，RS被赋值为空，它将匹配到空行，因此把每个软件包详细信息作为单个记录读取，并且记录中的每一行都是一个字段，而分隔字段使用FS变量，为了提取软件包名称，这里使用了空格（" ")作为FS的分隔符。  

### ORS Example: 输出的记录分隔符变量  

ORS是RS变量的输出等价物，表示输出中的每条记录都将用这个分隔符打印。下面是使用ORS的示例：  

```  
# awk 'BEGIN{ORS="=>"; RS=""} {print;}' opkg.info 
Package: iwinfo
Version: 2018-07-24-94b1366d-2
Depends: libc, libiwinfo
Status: install user installed
Architecture: mips_24kc
Installed-Time: 1553831708
 =>Package: luci-mod-rpc
Version: git-18.196.56128-9112198-1
Depends: libc, luci-lib-json
Status: install user installed
Architecture: all
Installed-Time: 1553831708=>Package: ubus                 
Version: 2018-01-16-5bae22eb-1
Depends: libc, libubus, libblobmsg-json, ubusd
Status: install ok installed
Architecture: mips_24kc
Installed-Time: 1553831711
Auto-Installed: yes=>
```  

上面的awk表达式中，以每个软件包信息作为一条记录，并且记录的输出用"=>"分隔。  

### NR Example: 记录数变量  

NR表示已经读出的记录数或者行号，从1开始，如果有多个文件话，这个值也是不断累加中。  

在下面的NR示例中，NR变量记录当前的记录数，等同于行号，在END部分中，NR告诉我们文件中的记录总数。  

```  
# awk '{print "Processing Record - ",NR;}END {print NR, "Packages Records are processed";}' opkg.info 
Processing Record -  1
Processing Record -  2
Processing Record -  3
Processing Record -  4
Processing Record -  5
Processing Record -  6
Processing Record -  7
Processing Record -  8
Processing Record -  9
Processing Record -  10
Processing Record -  11
Processing Record -  12
Processing Record -  13
Processing Record -  14
Processing Record -  15
Processing Record -  16
Processing Record -  17
Processing Record -  18
Processing Record -  19
Processing Record -  20
Processing Record -  21
Processing Record -  22
Processing Record -  23
Processing Record -  24
Processing Record -  25
25 Packages Records are processed
```  

### NF Example: 当前记录中的字段数变量  

NF表示当前记录的字段总数，也就是列数。NF对于验证记录中是否存在所有字段非常有用。  

让我们来看一个学生成绩文件，其中有学生缺少一项成绩，如下所示。  

```  
cat student.txt 
Jones 78 84 77
Gondrol 56 58 45
RinRao 38 37
Edwin 78 67 45
Dayan 30 47
```  

以下的awk脚本，打印记录(行)号和该记录中的字段数，很容易发现哪些学生的成绩缺少了。

```  
# awk '{print NR,"->",NF}' student.txt 
1 -> 4
2 -> 4
3 -> 3
4 -> 4
5 -> 3
```  

### FILENAME Example: 当前输入的文件名  

以下的示例，在处理完后把当前的输入文件名打印出来：  

```  
# awk 'END{print "Current input file:", FILENAME}' student.txt 
Current input file: student.txt
```  

### FNR Example: 相对于当前输入文件的记录数  

与NR不同的是，这个值会是各个文件自己的行号，而NR对于多个文件，NR的值是不断累加的。  

我们处理单个文件的情况较多，多文件处理相对用的少。  

## Expressions as Patterns  

任何awk表达式都是合法的awk模式，如果表达式的值非零（对于整数）或者非空（对于字符串），则模式匹配。处理每条记录时，都会执行一次表达式。

### Comparison expression  

awk支持比较表达式，比较运算符：==, !=, >, <, >=, <=。  

以下示例通过比较表达式提取本机所有监听端口：  

```  
# netstat -nap | awk '$6 == "LISTEN" {print;}'
tcp        0      0 0.0.0.0:139             0.0.0.0:*               LISTEN      1833/smbd
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1652/lighttpd
tcp        0      0 0.0.0.0:83              0.0.0.0:*               LISTEN      1652/lighttpd
tcp        0      0 127.0.0.1:53            0.0.0.0:*               LISTEN      2858/dnsmasq
tcp        0      0 192.168.17.211:53       0.0.0.0:*               LISTEN      2858/dnsmasq
```  

### Pattern match  

awk跟grep一样，支持模式匹配。模式匹配格式：  

```  
var ~ /regex/ <=> /regex/
var !~ /regex/ <=> !/regex/
```    

把上面的条件表达式改成模式匹配，其等价于：  

```  
# netstat -nap | awk '$6 ~ /LISTEN/ {print;}'
tcp        0      0 0.0.0.0:139             0.0.0.0:*               LISTEN      1833/smbd
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1652/lighttpd
tcp        0      0 0.0.0.0:83              0.0.0.0:*               LISTEN      1652/lighttpd
tcp        0      0 127.0.0.1:53            0.0.0.0:*               LISTEN      2858/dnsmasq
tcp        0      0 192.168.17.211:53       0.0.0.0:*               LISTEN      2858/dnsmasq

(or)

# netstat -nap | awk '/LISTEN/ {print;}'
tcp        0      0 0.0.0.0:139             0.0.0.0:*               LISTEN      1833/smbd
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1652/lighttpd
tcp        0      0 0.0.0.0:83              0.0.0.0:*               LISTEN      1652/lighttpd
tcp        0      0 127.0.0.1:53            0.0.0.0:*               LISTEN      2858/dnsmasq
tcp        0      0 192.168.17.211:53       0.0.0.0:*               LISTEN      2858/dnsmasq
```  

### Example  

opkg.info文件记录了软件包信息，其中自动安装的软件包会包含"Auto-Installed: yes"字段。

```  
# cat opkg.info 
Package: iwinfo
Version: 2018-07-24-94b1366d-2
Depends: libc, libiwinfo
Status: install user installed
Architecture: mips_24kc
Installed-Time: 1553831708

Package: luci-mod-rpc
Version: git-18.196.56128-9112198-1
Depends: libc, luci-lib-json
Status: install user installed
Architecture: all
Installed-Time: 1553831708

Package: ubus                 
Version: 2018-01-16-5bae22eb-1
Depends: libc, libubus, libblobmsg-json, ubusd
Status: install ok installed
Architecture: mips_24kc
Installed-Time: 1553831711
Auto-Installed: yes
```  

下面的示例使用了模式匹配的取反规则来打印所有非自动安装的软件包名称：  

```  
# awk 'BEGIN{ORS=" ";RS=""};!/Auto-Installed/{print $2}' opkg.info 
iwinfo luci-mod-rpc
```  

## See also  

https://www.gnu.org/software/gawk/manual/html_node/Expression-Patterns.html
