---
title: git1
date: 2022-03-23 10:45:45
tags:
---

# git显示中文乱码

git status 查看改动发现中文乱码
```
git config --global core.quotepath false
加上后在git配置文件.gitconfig会出现
[core]
        quotepath = false
```

# 
