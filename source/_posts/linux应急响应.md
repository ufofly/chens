---
title: linux应急响应
date: 2022-04-07 11:28:37
tags: linux
---
![](https://raw.githubusercontent.com/ufofly/picgo/master/%E5%BA%94%E6%80%A5%E5%93%8D%E5%BA%94.png)

# 1. 账户安全

先查看基础用户信息文件(/etc/passwd，/etc/shadow，/etc/group)

**/etc/shadow 字段说明**

`登录名:加密口令:最后一次修改时间:最小时间间隔:最大时间间隔:警告时间:不活动时间:失效时间:标志`


1）“登录名”是与/etc/passwd文件中的登录名相一致的用户账号
2）“口令”字段存放的是加密后的用户口令字，如果为空，则对应用户没有口令，登录时不需要口令；   
星号代表帐号被锁定；
双叹号表示这个密码已经过期了。
$6$开头的，表明是用SHA-512加密的，
$1$ 表明是用MD5加密的
$2$ 是用Blowfish加密的
$5$ 是用 SHA-256加密的。 
3）“最后一次修改时间”表示的是从某个时刻起，到用户最后一次修改口令时的天数。时间起点对不同的系统可能不一样。例如在SCOLinux中，这个时间起点是1970年1月1日。
4）“最小时间间隔”指的是两次修改口令之间所需的最小天数。
5）“最大时间间隔”指的是口令保持有效的最大天数。
6）“警告时间”字段表示的是从系统开始警告用户到用户密码正式失效之间的天数。
7）“不活动时间”表示的是用户没有登录活动但账号仍能保持有效的最大天数。
8）“失效时间”字段给出的是一个绝对的天数，如果使用了这个字段，那么就给出相应账号的生存期。期满后，该账号就不再是一个合法的账号，也就不能再用来登录了。





1、查询特权用户特权用户(uid 为0),sudo特权用户
awk -F: '$3==0{print $1}' /etc/passwd
more /etc/sudoers | grep -v "^#\|^$" | grep "ALL=(ALL)"

2、查询可以远程登录的帐号信息
awk '/\$1|\$6/{print $1}' /etc/shadow

3、除root帐号外，其他帐号是否存在sudo权限。如非管理需要，普通帐号应删除sudo权限
more /etc/sudoers | grep -v "^#\|^$" | grep "ALL=(ALL)"

4、查询空口令账号
awk -F: 'length($2)==0 {print $1}' /etc/shadow

5、禁用或删除多余及可疑的帐号
usermod -L user    #禁用帐号，帐号无法登录，/etc/shadow第二栏为!开头
userdel -r user    #将删除user用户，并且将/home目录下的user目录一并删除
passwd -l username  #禁用账号，在shadow密码前面加两个感叹号 -u解除禁用
修改/etc/passwd，把表示密码的第二个字段中X改为其他任意的字符，该账号就不能登录。
修改/etc/passwd，把最后一个shell字段修改为/sbin/nologin。
修改/etc/shadow，在第二个密码字段前面加上一个！或者两个！！，该账号就不能登录，这个效果和锁定账号一样
修改/etc/shadow，在最后两个冒号之间加上数字“1”，表示该账号的密码自1970年1月1日起，过一天后立即过期，当然现在自然就不能登录了。

6、查询登录用户及登录时长
who     # 查看当前登录系统的所有用户（tty 本地登陆  pts 远程登录）
w       # 显示已经登录系统的所用用户，以及正在执行的指令
uptime  # 查看登陆多久、多少用户，负载状态
whoami  #查看与当前有效用户ID关联的用户名
lastlog #查看所有用户最后一次登录的时间：
last    #查看所有用户的登录注销信息及系统的启动、重启及关机事件
lastb   #查看用户错误的登录列表

last
#显示logged in表示用户还在登录
# pts表示从SSH远程登录
# tty表示从控制台登录，就是在服务器旁边登录

lastb
# ssh表示从SSH远程登录
# tty表示从控制台登录



命令排查黑客什么时间登录的有的黑客登录时，会将/var/log/wtmp文件删除或者清空，这样就无法使用last命令获得有用的信息了。在做系统加固的时候可以使用 chattr +a 对 /var/log/wtmp 文件进行锁定，避免被黑客删除。



1. linux的密码/etc/passwd中,只显示一个x,真正的密码放在/etc/shadow中
2. 密码的三种状态
`$`长度34个字符的经过md5混编的不可逆密码
`!!` :两个叹号: 表示这个帐号目前没有密码，也不能用来登录，通常为一些系统帐号
`*` : 星号,使此账号无法登入
3. 如何临时关闭一个账号?
临时关闭（锁定）一个用户帐号，并不需要修改该用户的密码。只需要在/etc/shadow文件里属于该用户的行的第二个字段（密码）前面加上星号`*`就可以了。星号`*`指的是该用户不允许登录。当你想要把该用户恢复正常，只需要把星号“*”去掉就可以了，用户就可以恢复正常。

# 2.端口、进程、服务

`使用netstat -antulp网络连接命令，分析可疑端口、IP、PID`


ls -l /proc/$PID/exe 查看下pid所对应的进程文件路径
file /proc/$PID/exe  查看下pid所对应的进程文件路径
lsof -i:xx 查看某个端口是哪个进程打开的
fuser -n tcp xxx 查看端口xxx对应的进程pid
lsof -p xxx  根据pid查看进程
lsof -c sshd  通过服务名查看该进程打开的文件
ps -p PID -o lstart 查看pid的启动事件点
kill -9  xxx  根据pid强行停止程
pkill -kill -t pts/1  根据连接终端类型踢掉用户
netstat -nat | awk '{print $6}'| sort | uniq -c | sort -rn 统计tcp连接状态
netstat -anlp | grep 80 | grep tcp | awk ‘{print $5}’ | awk -F: ‘{print $1}’ | sort | uniq -c | sort -nr | head -n 20  查看tcp 80端口请求的ip
netstat -ntlp | grep 62333 | awk '{print $7}' | cut -d/ -f1 根据端口显示进程号






# 3.linux持久化驻留方法

## 检查是否存在可疑定时任务

枚举定时任务：crontab-l      

查看anacron异步定时任务：cat /etc/anacrontab

## 检查是否存在可疑服务

枚举主机所有服务，查看是否有恶意服务：

`centos6 service --status-all`

`centos7 systemctl list-units`

## 扫描是否存在恶意驱动

枚举/扫描系统驱动：lsmod

安装chkrootkit 、rkhunter进行扫描

`http://ftp.pangeia.com.br/download.htm`

`https://sourceforge.net/projects/rkhunter/`

## ssh弱口令

## 检查是否有被攻击痕迹

查询log主机登陆日志：

```
grep "Accepted " /var/log/secure* | awk '{print $1,$2,$3,$9,$11}'
```

定位有爆破的源IP：

```
grep "Failed password" /var/log/secure|grep -E -o "(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)"|uniq -c
```

爆破日志的用户名密码：

```
grep "Failed password" /var/log/secure|perl -e 'while($_=<>){ /for(.*?) from/; print "$1\n";}'|uniq -c|sort -nr
```

SSH爆破是Linux病毒最常用的传播手段，若存在弱密码的主机很容易被其他感染主机SSH爆破成功，从而再次感染病毒。

# 4.日志分析

```bash
日志默认存放位置：/var/log/
日志配置位置：/etc/rsyslog.conf


日志文件             说明
/var/log/cron         记录了系统定时任务相关的日志
/var/log/mailog     记录邮件信息
/var/log/auth.log   包含系统授权信息，包括用户登录和使用的权限机制等
/var/log/userlog    记录所有等级用户信息的日志
/var/log/vsftpd.log 记录Linux ftp 日志
/var/log/message     记录系统重要信息的日志。这个日志文件中会记录Linux系统的绝大多数重要信息，如果系统出现问题时，首先要检查的就应该是这个日志文件
/var/log/lastlog     记录系统中所有用户最后一次登录时间的日志，这个文件是二进制文件，不能直接vi，而要使用lastlog命令查看
/var/log/btmp       记录错误登录日志，这个文件是二进制文件，不能直接vi查看，而要使用lastb命令查看
/var/log/wtmp         永久记录所有用户的登录、注销信息，同时记录系统的启动、重启、关机事件。同样这个文件也是一个二进制文件，不能直接vi，而需要使用last命令来查看
/var/run/utmp         记录当前已经登录的用户信息，这个文件会随着用户的登录和注销不断变化，只记录当前登录用户的信息。同样这个文件不能直接vi，而要使用w,who,users等命令来查询
/var/log/secure     记录验证和授权方面的信息，只要涉及账号和密码的程序都会记录，比如SSH登录
/var/log/faillog    记录系统登录不成功的账户信息，一般会被黑客删除。
```

```bash
1、定位有多少IP在爆破主机的root帐号：    
grep "Failed password for root" /var/log/secure | awk '{print 11}' | sort | uniq -c | sort -nr | more
定位有哪些IP在爆破：
grep "Failed password" /var/log/secure|grep -E -o "(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)"|uniq -c
爆破用户名字典是什么？
grep "Failed password" /var/log/secure|perl -e 'while(_=<>){ /for(.*?) from/; print "1\n";}'|uniq -c|sort -nr
2、登录成功的IP有哪些：   
grep "Accepted " /var/log/secure | awk '{print11}' | sort | uniq -c | sort -nr | more
登录成功的日期、用户名、IP：
grep "Accepted " /var/log/secure | awk '{print 1,2,3,9,$11}' 
3、增加一个用户kali日志：
Jul 10 00:12:15 localhost useradd[2382]: new group: name=kali, GID=1001
Jul 10 00:12:15 localhost useradd[2382]: new user: name=kali, UID=1001, GID=1001, home=/home/kali
, shell=/bin/bash
Jul 10 00:12:58 localhost passwd: pam_unix(passwd:chauthtok): password changed for kali
#grep "useradd" /var/log/secure 
4、删除用户kali日志：
Jul 10 00:14:17 localhost userdel[2393]: delete user 'kali'
Jul 10 00:14:17 localhost userdel[2393]: removed group 'kali' owned by 'kali'
Jul 10 00:14:17 localhost userdel[2393]: removed shadow group 'kali' owned by 'kali'
# grep "userdel" /var/log/secure
5、su切换用户：
Jul 10 00:38:13 localhost su: pam_unix(su-l:session): session opened for user good by root(uid=0)
sudo授权执行:
sudo -l
Jul 10 00:43:09 localhost sudo:    good : TTY=pts/4 ; PWD=/home/good ; USER=root ; COMMAND=/sbin/shutdown -r now
```

utmp、wtmp、btmp文件

Linux用户登录信息放在三个文件中：

/var/run/utmp：记录当前正在登录系统的用户信息，默认由who和w记录当前登录用户的信息，uptime记录系统启动时间；

/var/log/wtmp：记录当前正在登录和历史登录系统的用户信息，默认由last命令查看；

/var/log/btmp：记录失败的登录尝试信息，默认由lastb命令查看。

相关命令介绍

查看这三个日志文件的命令，分别是lastlog、last、lastb、ac、who、w、users、utmpdump。其中last、lastb、who、utmpdump可以通过指定参数而查看三个中的任意一个文件。

**1、lastlog**

列出所有用户最近登录的信息，或者指定用户的最近登录信息。lastlog引用的是/var/log/lastlog文件中的信息，包括login-name、port、last login time。

```
# lastlog
```

**2、last**

列出当前和曾经登入系统的用户信息，它默认读取的是/var/log/wtmp文件的信息。输出的内容包括：用户名、终端位置、登录源信息、开始时间、结束时间、持续时间。注意最后一行输出的是wtmp文件起始记录的时间。当然也可以通过last -f参数指定读取文件，可以是/var/log/btmp、/var/run/utmp。

```
# last
```

**3、lastb**

列出失败尝试的登录信息，和last命令功能完全相同，只不过它默认读取的是/var/log/btmp文件的信息。当然也可以通过last -f参数指定读取文件，可以是/var/log/wtmp、/var/run/utmp。

```
# lastb
# lastb -F -f /var/log/wtmp
# lastb -F -f /var/log/utmp
```



**4、who**

查看当前登入系统的用户信息。其中who -m等效于who am i。

who命令强大的一点是，它既可以读取utmp文件也可以读取wtmp文件，默认没有指定FILE参数时，who查询的是utmp的内容。当然可以指定FILE参数，比如who -aH /var/log/wtmp,则此时查看的是wtmp文件。

```
# who
# who -rH
# who -q
```

**5、w**

查看当前登入系统的用户信息及用户当前的进程（而who命令只能看用户不能看进程）。该命令能查看的信息包括字系统当前时间，系统运行时间，登陆系统用户总数及系统1、5、10分钟内的平均负载信息。后面的信息是用户，终端，登录源，login time，idle time，JCPU，PCPU，当前执行的进程等。

w的信息来自两个文件：用户登录信息来自/var/run/utmp，进程信息来自/proc/.

**6、users**

显示当前正在登入统的用户名。语法是users \[OPTION\]… \[FILE\]。如果未指定FILE参数则默认读取的是/var/run/utmp，当然也可以指定通用相关文件/var/log/wtmp，此时输出的就不是当前用户了。

```
# users
```

**7、utmpdump**

utmpdump用于转储二进制日志文件到文本格式的文件以便查看，同时也可以修改二进制文件！！包括/var/run/utmp、/var/log/wtmp、/var/log/btmp。语法为：utmpdump \[options\] \[filename\]。修改文件实际就可以抹除系统记录，所以一定要设置好权限，防止非法入侵。

例子：修改utmp或wtmp。由于这些都是二进制日志文件，你不能像编辑文件一样来编辑它们。取而代之是，你可以将其内容输出成为文本格式，并修改文本输出内容，然后将修改后的内容导入回二进制日志中。如下： 

```
# utmpdump /var/log/utmp > tmp_output.txt          #导出文件信息
# utmpdump -r tmp_output.txt > /var/log/utmp       #导入到源文件中
```

----
