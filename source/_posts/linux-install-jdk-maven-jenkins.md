---
title: linux环境安装
date: 2021-01-24 22:08:15
tags:
    - linux
categories:
    - linux
---

### 1、Jdk安装

#### [下载](https://www.oracle.com/java/technologies/javase-downloads.html)

#### 解压
```
tar -zxvf jdk-8u91-linux-x64.tar.gz /opt/java
```

#### 环境变量
```
vim /etc/profile

export JAVA_HOME=/opt/java/jdk1.8.0_91
export PATH=$PATH:$JAVA_HOME/bin
export PATH

source /etc/profile
```

#### 查看版本
```
java -version
``` 

### 2、Maven安装

#### [下载](http://maven.apache.org/download.cgi)

#### 解压
```
tar -zxvf apache-maven-3.6.3-bin.tar.gz /opt/server
```

#### 环境变量
```
vim /etc/profile

export MAVEN_HOME=/opt/server/apache-maven-3.6.3
export PATH=$PATH:$MAVEN_HOME/bin
export PATH

source /etc/profile
```

#### 查看版本
```
mvn -v
```

### 3、Jenkins安装

#### 下载rpm包
```
wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/jenkins-2.263.2-1.1.noarch.rpm
```

#### 安装
```
sudo yum install jenkins-2.263.2-1.1.noarch.rpm
```

#### 修改端口
```
vim /etc/sysconfig/jenkins

JENKINS_PORT="8080"
```

#### 启动
```
service jenkins start
```

#### 如果报错
```
vim /etc/init.d/jenkins

/usr/bin/java
/opt/java/jdk1.8.0_91/bin/java
```

#### 初始密码
```
cat /var/lib/jenkins/secrets/initialAdminPassword
```

#### 卸载
```
service jenkins stop
yum clean all
yum -y remove jenkins
rm -rf /var/cache/jenkins
rm -rf /var/lib/jenkins/
```