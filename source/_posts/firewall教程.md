---
title: firewall教程
date: 2023-03-21 10:08:30
tags:
---
# firewall

#### 查看服务的状态

```
firewall-cmd --state
```

#### 永久配置存储位置

```
firewall的每个区域的永久配置存储在/usr/lib/firewalld/zones/ 目录中
```

#### 使新设置持久

```
firewall-cmd --runtime-to-permanent
```

#### 添加新端口

通过打开端口，系统可从外部访问，这代表了安全风险。通常，让端口保持关闭，且只在某些服务需要时才打开。

##### 1.列出所有允许的端口：

```
firewall-cmd --list-all
```

##### 2.在允许的端口中添加一个端口，以便为入站流量打开这个端口：

```
firewall-cmd --add-port=port-number/port-type
```

端口类型为 tcp、udp、sctp 或 dccp。这个类型必须与网络通信的类型匹配。

##### 3.使新设置具有持久性

```
firewall-cmd --runtime-to-permanent
```

端口类型为 tcp、udp、sctp 或 dccp。这个类型必须与网络通信的类型匹配。

#### 关闭端口

##### 1.列出要关闭的端口情况

```
firewall-cmd --list-all
```

##### 2.从允许的端口中删除端口，以便对传入的流量关闭

```
firewall-cmd --remove-port=port-number/port-type
```

##### 3.使新设置具有持久性

```
firewall-cmd --runtime-to-permanent
```

#### 列出区域

##### 1.查看系统中有哪些可用区：

```
firewall-cmd --get-zones
```

firewall-cmd --get-zones 命令显示系统上所有可用的区，但不显示特定区的详情。

##### 2.查看所有区的详细信息：

```
firewall-cmd --list-all-zones
```

##### 3.查看特定区的详细信息：

```
firewall-cmd --zone=zone-name --list-all
```

#### 更改特定区的 firewalld 设置

要在其他区域中工作，请使用 `--zone=*zone-name*` 选项。例如，要允许在区 *public* 中使用 `SSH` 服务：

```
firewall-cmd --add-service=ssh --zone=public
```

#### 更改默认区

##### 1.显示当前的默认区：

```
firewall-cmd --get-default-zone
```

##### 2.设置新的默认区：

```
firewall-cmd --set-default-zone zone-name
```

#### 将区分配给特定的接口

##### 1.列出活跃区以及分配给它们的接口

```
firewall-cmd --get-active-zones
```

##### 2.为不同的区分配接口

```
firewall-cmd --zone=zone_name --change-interface=interface_name --permanent
```

#### 在 ifcfg 文件中手动将区分配给网络连接

当连接由 **NetworkManager** 管理时，必须了解它使用的区。为每个网络连接指定区域，根据计算机有可移植设备的位置提供各种防火墙设置的灵活性。因此，可以为不同的位置（如公司或家）指定区域和设置。要为连接设置一个区，请编辑 `/etc/sysconfig/network-scripts/ifcfg-*connection_name*` 文件，并添加将区分配给这个连接的行：

```
ZONE=zone_name
```

#### 创建一个新区

##### 1.创建一个新区

```
firewall-cmd --permanent --new-zone=zone-name
```

##### 2.检查是否在您的永久设置中添加了新的区：

```
firewall-cmd --get-zones
```

##### 3.使新设置具有持久性

```
firewall-cmd --runtime-to-permanent
```

####  使用区目标设定传入流量的默认行为

对于每个区，您可以设置一种处理尚未进一步指定的传入流量的默认行为。此行为是通过设置区的目标来定义的。有四个选项：

- `ACCEPT` ：接受所有传入的数据包，除了特定规则禁止的。
- `REJECT` ：拒绝所有传入的数据包，除了特定规则允许的。当 `firewalld` 拒绝数据包时，会告知源机器有关拒绝的信息。
- `DROP` ：丢弃所有传入的数据包，除了特定规则允许的。当 `firewalld` 丢弃数据包时，不会告知源机器有关丢弃数据包的信息。
- `default` ：与 `REJECT` 的行为类似，但在某些情况下具有特殊含义。详情请查看 `firewall-cmd(1)` 手册页中的 `适应和查询区和策略的选项` 部分。

#### 流程

为区设置目标：

1. ##### 列出特定区的信息以查看默认目标：

   ```none
   firewall-cmd --zone=zone-name --list-all
   ```

2. ##### 在区中设置一个新目标：

   ```none
   firewall-cmd --permanent --zone=zone-name --set-target=<default|ACCEPT|REJECT|DROP>
   ```