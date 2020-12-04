---
title: Linux 创建自定义命令
date: 2020-11-01 10:22:17
tags:
    - 服务器
    - linux
categories:
    - 服务器
---
###Linux 创建自定义命令

#### 语法
```shell script
alias[别名]=[指令名称]
```

#### 使用
* 临时生效
```shell script
# 直接在终端使用命令
alias ls='top'
```
```shell script
# 取消自定命令
unalias ls
```

* 当前用户永久生效
```shell script
vim ~/.bashrc
# 文件内容尾行添加 
alias show="netstat -npl | grep $1"
# 立即生效
source /etc/bashrc
```
    
* 所用用户永久生效
```shell script
vim /etc/bashrc
# 文件内容尾行添加 
alias show="netstat -npl | grep $1"
# 立即生效
source /etc/bashrc
```