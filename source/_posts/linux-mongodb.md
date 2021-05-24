---
title: MongoDB 命令行操作
date: 2021-05-24 21:23:50
tags:
    - 数据库
    - linux
categories:
    - 数据库
---

### MongoDB 命令行操作

#### 安装
```
# 查找mongo
docker search mongo
# 拉取mongo最新镜像
docker pull mongo
# 查看本地镜像
docker images
# 运行
docker run -d  -p 27017:27017 --network somework --name mongodb \
-e MONGO_INITDB_ROOT_USERNAME=用户名 \
-e MONGO_INITDB_ROOT_PASSWORD=密码 \
mongo
```

#### 进入命令行
```
# 查看CONTAINER ID
docker ps
# 1 进入命令行
docker exec -it [CONTAINER ID | name] mongo
# 2 进入命令行
docker exec -it [CONTAINER ID | name] bash
# 用户名密码登录
mongo 127.0.0.1:27017 -u '用户名' -p '密码' --authenticationDatabase 'admin'
```

#### 创建用户
```
# 创建用户和密码
db.createUser({ user: 'admin', pwd: '123', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] }); 
# 验证
db.auth("admin","123");
```

#### 数据库操作
```
# 查看数据库
show dbs
# 切换数据库
use mydb
# 删除当前数据库
db.dropDatabase()
```

#### 集合操作
```
# 查看集合
show collections
# 删除集合
db.users.drop()
```

#### 文档查询
```
# 查询所有文档
db.test.find()
# 查询所有文档个数
db.test.find().count()
# 条件查询
db.test.find({
    # >
    'field': {$gt: 'value'},
    # <
    'field': {$lt: 'value'},
    # >=
    'field': {$gte: 'value'},
    # <=
    'field': {$lte: 'value'},
    # < field <
    'field': {$gt: 'value1', $lt: 'value2'},
    # =
    'field': 'value',
    # !=
    'field': {$ne: 'value'},
    # in
    'field': {$in: ['value1', 'value2', 'value3']},
    # not in
    'field': {$nin: ['value1', 'value2', 'value3']}
})
# 限制结果数量
db.test.find().limit(3)
# 从第3条记录开始，返回5条记录(limit 3, 5)
db.test.find().skip(3).limit(5)
# 升序（asc）
db.test.find().sort({value: 1});
# 降序（desc）
db.test.find().sort({value: -1});
```

#### 文档操作
```
# 插入文档
db.test.insert({
    'field1': 'value1',
    'field2': 'value2'
})
# 更新文档
db.test.update({
    'field1': 'value1'
}, {
    'field2': 'value2'   
})
# 删除所有
db.test.remove({})
# 条件删除
db.test.remove({
    'field': 'value'
})
```
