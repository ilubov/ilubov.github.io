---
title: mysql使用函数递归查找父级或子级id
date: 2020-11-13 21:22:17
tags:
    - 数据库
    - mysql
categories:
    - 数据库
---
### mysql使用函数递归查找父级或子级id

#### 父级
```mysql
CREATE FUNCTION `getParentList`(rootId bigint)
  RETURNS varchar(1000)
  BEGIN
    DECLARE fid varchar(100) default '';
    DECLARE str varchar(1000) default rootId;
    WHILE rootId is not null do
      SET fid = (SELECT parent_id FROM table_name WHERE id = rootId);
      IF fid is not null AND fid > 0 THEN SET str = concat(str, ',', fid);
        SET rootId = fid;
      ELSE
        SET rootId = fid;
      END IF;
    END WHILE;
  RETURN str;
END
```

#### 子级
```mysql
CREATE FUNCTION `getChildList`(rootId bigint) 
  RETURNS VARCHAR(1000) 
  BEGIN
   DECLARE str VARCHAR(1000); 
   DECLARE cid VARCHAR(1000); 
   DECLARE k INT DEFAULT 0;
   SET cid = CAST(rootId AS CHAR);
   SET str = cid; 
   WHILE cid IS NOT NULL DO 
        IF k > 0 THEN SET str = CONCAT(str, ',', cid);
        END IF;
        SELECT GROUP_CONCAT(id) INTO cid FROM table_name WHERE FIND_IN_SET(parent_id, cid) > 0;
        SET k = k + 1;
   END WHILE; 
   RETURN str; 
END
```