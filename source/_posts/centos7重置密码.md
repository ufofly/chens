---
title: centos7重置密码
date: 2022-03-26 19:50:34
tags:
---

**操作步骤**

1. 重启操作系统  
2. 当系统启动至引导界面，按任意键。然后将[光标]移动至待启动内核，按 e 编辑选中行。  
3. 移动光标至 kernel command line（linux16 开头的行）  
4. 移动至行末尾，增加 rd.break。  
5. 按 Ctrl +x 启动系统
6. 系统启动至 root shell 界面，此时，系统本身的 root 文件系统被以只读模式挂载到 / sysroot 目录下  
7. 以读写模式重新挂载 / sysroot

```
switch_root:/# mount -o remount,rw /sysroot
#检查/sysroot是否以读写模式挂载
switch_root:/# mount | grep sysroot
/dev/mapper/centos-root on /sysroot type xfs(rw,relatime,attr2,inode64,noquota)
```
8. 执行 chroot 命令，将 / sysroot 目录切换为根目录。
```
switch_root:/# chroot /sysroot
#chroot命令执行成功后，shell提示符将变为下述表示方法
sh-4.2# 
```

> chroot jail 是什么？在 linux 操作系统中，默认的根目录都是‘/’, 而 chroot 就是为改变正在运行的进程以及它的子进程的根目录而生。假设，某个程序的根目录从原先的默认的系统根目录‘/’, 被你修改到 / home 目录下，这个 / home 目录就变成这个程序的逻辑根目录，那么，这个被修改了根目录环境的程序，就不能进入这个逻辑根目录以外的路径。本质上，这就是限制某个程序所能进入的目录树，所以，被称为 chroot 监狱。因此，这个程序的活动范围就从本来的整个系统 "/", 到后来的逻辑根 “/home”。chroot(change root) 命令把根目录换成指定的目的目录。

9. 创建一个新的密码  
10. 根目录下创建. autorelabel 文件，确保在系统启动时，未标记的文件自动标记。

```
#针对已启用selinux的主机，该步骤的目的是重新生成文件的标记，必不可少。
sh-4.2# touch /.autorelabel
```
11. 键入两次 exit，第一次退出 chroot 环境，第二次退出 initramfs 调试 shell，系统会自动重启。  
12. 使用新密码正常登录。

