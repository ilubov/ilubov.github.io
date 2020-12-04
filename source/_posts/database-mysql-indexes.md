---
title: mysql索引失效
date: 2020-04-01 19:22:17
tags:
    - 数据库
    - mysql
categories:
    - 数据库
---
### mysql索引失效
* where语句中包含or时，可能会导致索引失效，尽量避免使用or语句，可以根据情况尽量使用union all或者in来代替
```mysql
-- 不走索引
select * from `user` where name = '1234' or age = 11;
```
* where语句中索引列使用了负向查询，可能会导致索引失效

    负向查询包括!=、<>、NOT IN、NOT LIKE等
```mysql
-- 不走索引
select * from `user` where name <> '1234';
```
* 索引字段可以为null，使用is null或is not null时，可能会导致索引失效
```mysql
-- 不走索引
select * from `user` where name is not null;
```
* like查询是以%开头，可改为右模糊
```mysql
-- 不走索引
select * from user where name like '%1234';
-- 走索引
select * from user where name like '1234%';
```
* 如果列类型是字符串，那一定要在条件中将数据使用引号引用起来，否则不使用索引
```mysql
-- 不走索引
select * from user where name = 1234;
-- 走索引
select * from user where name = '1234';
```
* 列与列对比，例如column1 = column2
```mysql
-- 不走索引
select * from user where name = full_name;
```
* 条件上使用函数或运算
```mysql
-- 不走索引
select * from user where DATE_ADD(last_login_time, INTERVAL 1 DAY) = 7;
select * from user where age - 10 > 10;
```
* 将name、age、phone顺序设置为联合索引，一定要注意顺序，mysql联合索引有最左匹配原则
```mysql
-- 不走索引
select * from user where age = 11 and name = '1234';
-- 走索引
select * from user where name = '1234';
select * from user where name = '1234' and age = 11;
select * from user where name = '1234' and age = 11 and phone = '186';
```
* mysql优化器判断全表扫描或者走索引哪个成本低，如果使用全表扫描要比使用索引快，则不使用索引

