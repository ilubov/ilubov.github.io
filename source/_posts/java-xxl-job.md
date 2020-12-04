---
title: xxl job
date: 2020-11-12 23:01:19
tags:
    - java
categories:
    - java
---
### xxl job

#### 1、代码下载
* 码云地址：https://gitee.com/xuxueli0323/xxl-job
* gitHub地址：https://github.com/xuxueli/xxl-job

#### 2、执行SQL
在数据库执行   \xxl-job\doc\db\tables_xxl_job.sql

#### 3、修改配置文件
修改配置文件：xxl-job\xxl-job-admin\src\main\resources\application.properties
调整数据库连接、控制台用户名密码等内容

#### 4、在docker上部署xxl-job
* 打包xxl-job-admin
```
mvn clean package -Dmaven.test.skip=true
```
* 在服务器中创建xxl-job文件夹： mkdir /xworks/data/xxl-job
* 进入新创建的xxl-job文件夹，新建一个dockerfile文件，编辑文件
```dockerfile
# Docker image for springboot file run
# VERSION 0.0.1
# Author: ilubov
# 基础镜像使用java
FROM java:8
# 作者
MAINTAINER ilubov <o@ilubov.cn>
# VOLUME 指定了临时文件目录为/tmp。
# 其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp
VOLUME /tmp
# 将jar包添加到容器中并更名为app.jar，此处的jar包名称一定要与要部署的jar包名称一致
ADD xxl-job-admin-2.3.0-SNAPSHOT.jar app.jar
# 运行jar包
RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
* 将xxl-job-admin-2.3.0-SNAPSHOT.jar上传至xxl-job文件夹
* 构建新的镜像
```shell script
docker build -t xxl-job-admin .
```
* 使用xxl-job-admin镜像启动新容器
```shell script
docker run -d -p 8080:8080 xxl-job-admin
```
* 访问 
```
server {
  listen 80;
  server_name  xxl-job.xxx.com;
  index  index.php index.html index.htm;

  location / {
    proxy_pass  http://127.0.0.1:8080;
    proxy_set_header Host $proxy_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```
```
http://ip:8080/xxl-job-admin
http://xxl-job.xxx.com/xxl-job-admin
```

#### 5、spring boot使用
* 添加依赖
```xml
<dependency>
    <groupId>com.xuxueli</groupId>
    <artifactId>xxl-job-core</artifactId>
    <version>2.2.0</version>
</dependency>
```
* config
```java
import com.xxl.job.core.executor.impl.XxlJobSpringExecutor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Slf4j
@Configuration
public class XXlJobConfig {

    @Value("${xxl.job.admin.addresses}")
    private String adminAddresses;

    @Value("${xxl.job.executor.appname}")
    private String appName;

    @Value("${xxl.job.executor.ip}")
    private String ip;

    @Value("${xxl.job.executor.port}")
    private int port;

    @Value("${xxl.job.accessToken}")
    private String accessToken;

    @Value("${xxl.job.executor.logpath}")
    private String logPath;

    @Value("${xxl.job.executor.logretentiondays}")
    private int logRetentionDays;

    @Bean
    public XxlJobSpringExecutor xxlJobExecutor() {
        log.info("===> xxl-job config init.");
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAppname(appName);
        xxlJobSpringExecutor.setIp(ip);
        xxlJobSpringExecutor.setPort(port);
        xxlJobSpringExecutor.setAccessToken(accessToken);
        xxlJobSpringExecutor.setLogPath(logPath);
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);
        return xxlJobSpringExecutor;
    }
}
```
* application.yml
```yaml
# xxl job
xxl:
  job:
    # 执行器通讯TOKEN：非空时启用
    accessToken:
    executor:
      # 执行器的名称
      appname: xxl-job-executor-sample
      # ip地址 不填自动识别
      ip:
      # 执行器运行日志文件存储的磁盘位置，需要对该路径拥有读写权限
      logpath: /data/applogs/xxl-job/jobhandler
      # 执行器Log文件定期清理功能，指定日志保存天数，日志文件过期自动删除。限制至少保持3天，否则功能不生效
      # -1表示永不删除
      logretentiondays: -1
      # 端口号
      port: 8081
    #调度中心部署跟地址：如调度中心集群部署存在多个地址则用逗号分隔。
    #执行器将会使用该地址进行"执行器心跳注册"和"任务结果回调"。
    admin:
      addresses: http://localhost:8080/xxl-job-admin
```
* test
```java
    @XxlJob("test")
    public ReturnT<String> test(String param) {
        log.info("test: {}", param);
        return ReturnT.SUCCESS;
    }
```

