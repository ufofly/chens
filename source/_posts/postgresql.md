---
title: postgresql
date: 2022-03-22 18:43:05
tags:
---

初次安装后，默认生成一个名为postgres的数据库和一个名为postgres的数据库用户。这里需要注意的是，同时还生成了一个名为postgres的Linux系统用户。这时相当于系统用户postgres以同名数据库用户的身份，登录数据库，这是不用输入密码的。如果一切正常，系统提示符会变为"postgres=#"，表示这时已经进入了数据库控制台。以下的命令都在控制台内完成。
切换到postgres
```
su - postgres
psql 

```
1.使用\password命令，为postgres用户设置一个密码。

```
\password  mima

```
2.创建数据库用户dbuser（刚才创建的是Linux系统用户），并设置密码。
`
CREATE USER dbuser WITH PASSWORD 'password';
`
3.创建用户数据库，这里为exampledb，并指定所有者为dbuser。
`
CREATE DATABASE exampledb OWNER dbuser;
`
4.将exampledb数据库的所有权限都赋予dbuser，否则dbuser只能登录控制台，没有任何数据库操作权限。
```
GRANT ALL PRIVILEGES ON DATABASE exampledb to dbuser;

```
