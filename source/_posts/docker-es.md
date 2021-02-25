---
title: ElasticSearch环境搭建
date: 2021-02-23 20:10:18
tags:
    - linux
    - docker
categories:
    - linux
---

### Docker部署ElasticSearch（7.9.3）
[官网地址](https://www.elastic.co/)
#### 下载启动
```
# 下载镜像
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.9.3

# 启动
docker run -d --name es -p 9200:9200 -p 9300:9300 -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.9.3
```
#### 配置跨域
```
# 进入es容器内部
docker exec -it es /bin/bash

# 修改elasticsearch.yml文件
vi config/elasticsearch.yml

# 修改为以下内容
cluster.name: "docker-cluster"
network.host: 0.0.0.0
http.cors.enabled: true
http.cors.allow-origin: "*"
discovery.zen.minimum_master_nodes: 1

# 退出容器并重启es
exit
docker restart es
```

### Docker部署ElasticSearch-Head
```
# 下载镜像
docker pull mobz/elasticsearch-head:5
# 启动
docker run -d --name es_admin -p 9100:9100 mobz/elasticsearch-head:5
```

### Docker部署ik中文分词插件
[下载地址](https://github.com/medcl/elasticsearch-analysis-ik/releases)
```
# 进入es容器内部，/plugins下新建ik文件夹
docker exec -it es /bin/bash
cd plugins/
mkdir ik
cd ik/

# 下载并解压
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.9.3/elasticsearch-analysis-ik-7.9.3.zip
unzip elasticsearch-analysis-ik-7.9.3.zip

# 退出容器并重启es
exit
docker restart es

# postman测试
post: http://ip:9200/_analyze?pretty=true
{
	"analyzer": "ik_max_word",
	"text": "我是张三丰"
}
```

### sql支持
[下载地址](https://github.com/NLPchina/elasticsearch-sql)
```
# 进入es容器内部
docker exec -it es /bin/bash

# 执行
./bin/elasticsearch-plugin install https://github.com/NLPchina/elasticsearch-sql/releases/download/7.9.3.0/elasticsearch-sql-7.9.3.0.zip

# 退出容器并重启es
exit
docker restart es
```
