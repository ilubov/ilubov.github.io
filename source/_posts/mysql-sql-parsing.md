---
title: Mysql sql解析
date: 2021-03-05 23:12:45
tags:
    - java
categories:
    - java
---

### Mysql sql解析
![image](/images/mysql/mysql-sql-parsing.png)
#### 示例语句
```
SELECT DISTINCT
 < select_list >
FROM
 < left_table > < join_type >
JOIN < right_table > ON < join_condition >
WHERE
 < where_condition >
GROUP BY
 < group_by_list >
HAVING
 < having_condition >
ORDER BY
 < order_by_condition >
LIMIT < limit_number >
```

#### 执行顺序
```
1 FROM <left_table>
2 ON <join_condition>
3 <join_type> JOIN <right_table>  第二步和第三步会循环执行
4 WHERE <where_condition>  第四步会循环执行，多个条件的执行顺序是从左往右的。
5 GROUP BY <group_by_list>
6 HAVING <having_condition>
7 SELECT 分组之后才会执行SELECT
8 DISTINCT <select_list>
9 ORDER BY <order_by_condition>
10 LIMIT <limit_number> 前9步都是SQL92标准语法。limit是MySQL的独有语法
```

#### sql
```
select 
	d.name, count(d.id) cnt
from dept d 
left join user u on d.id = u.dept_id 
where u.age > 10
group by d.name
having count(d.id) > 0
order by cnt desc
limit 10
```

#### 流程分析
* 1.FROM: 将2个表联合查询得到他们的笛卡尔积CROSS JOIN，产生虚表VT1。
* 2.ON: 对虚表VT1进行ON筛选，只有那些符合的行才会被记录在虚表VT2中。
* 3.LEFT JOIN: 保留左表中未匹配的行就会作为外部行，添加到虚拟表VT2中，产生虚拟表VT3。
* 4.WHERE: 对虚拟表VT3进行WHERE条件过滤，只有符合的记录才会被插入到虚拟表VT4中。
* 5.GROUP BY: 根据GROUP BY子句中的列，对VT4中的记录进行分组操作，产生虚拟表VT5 。
* 6.HAVING: 对虚拟表VT5应用HAVING过滤，只有符合的记录才会被插入到虚拟表VT6中。
* 7.SELECT: 对SELECT子句中的元素进行处理，生成VT7表。
* 8.ORDER BY: 根据ORDER BY子句的条件对结果进行排序，生成VT8表。
* 9.LIMIT: LIMIT子句从上一步得到的VT8虚拟表 中选出从指定位置开始的指定行数据。

#### 流程说明
* 单表查询：根据 WHERE 条件过滤表中的记录，形成中间表（这个中间表对用户是不可见的）；
然后根据SELECT 的选择列选择相应的列进行返回最终结果。

* 两表连接查询：对两表求积（笛卡尔积：行相乘、列相加）并用 ON 条件和连接连接类型进行过滤形成中间表；
然后根据WHERE条件过滤中间表的记录，并根据 SELECT 指定的列返回查询结果。

* 多表连接查询：先对第一个和第二个表按照两表连接做查询，然后用查询结果和第三个表做连接查询，以此类推，
直到所有的表都连接上为止，最终形成一个中间的结果表，然后根据WHERE条件过滤中间表的记录，并根据SELECT指定的列返回查询结果。
