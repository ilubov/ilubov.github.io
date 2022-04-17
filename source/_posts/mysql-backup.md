---
title: mysql数据库备份
date: 2022-04-17 23:22:38
tags:
    - 数据库
    - mysql
categories:
    - 数据库
---
### mysql数据库备份

#### python
```
#!/usr/bin/env python
# encoding: utf-8
import datetime
import os
import shutil
import subprocess
import time
import zipfile

# 数据库用户名
db_user = "root"
# 数据库密码
db_password = "password"
# 备份目录
backup_dir = "/data/mysql/backup"
# backup_prefix和backup_suffix分别为备份文件的前缀和后缀，
# 如database_backup_2020-06-18-16则代表该文件是在2020年6月18日的16点时备份的
backup_prefix = "database_backup"
backup_suffix = "%Y-%m-%d-%H"
# 备份数据库列表
backup_databases = [
    "database"
]
# 容器名
container_name = "mysql-prod"
# 过期小时，定期删除7个小时前的备份文件
expire_hour = 7


# 获取备份文件名
def get_backup_filename():
    t = time.strftime(backup_suffix, time.localtime())
    return "%s_%s" % (backup_prefix, t)


def get_backup_path():
    return "%s%s%s" % (backup_dir, os.sep, get_backup_filename())


# 获取过期时间戳
def get_expire_time():
    t = datetime.datetime.now() - datetime.timedelta(hours=expire_hour)
    return int(time.mktime(t.timetuple()))


def create_dir(dir_path):
    # 如果目录存在则退出
    if os.path.exists(dir_path):
        return
    os.mkdir(dir_path)


cmd_template = "docker exec -it {container_name} mysqldump -u{db_user} -p{db_password} {database} > {file_path}"


# 备份指定数据库
def backup_database(backup_path, database):
    file_path = os.sep.join([backup_path, "%s.sql" % database])
    d = {
        "container_name": container_name,
        "db_user": db_user,
        "db_password": db_password,
        "database": database,
        "file_path": file_path
    }
    cmd = cmd_template.format(**d)
    subprocess.call(cmd, shell=True)


def zip_dir(dir_path):
    file_path = '.'.join([dir_path, "zip"])
    if os.path.exists(file_path):
        os.remove(file_path)
    z = zipfile.ZipFile(file_path, 'w', zipfile.ZIP_DEFLATED)
    for root, directories, files in os.walk(dir_path):
        fpath = root.replace(dir_path, '')
        fpath = fpath and fpath + os.sep or ''
        for filename in files:
            z.write(os.path.join(root, filename), fpath + filename)
    z.close()


# 备份数据库
def backup():
    backup_path = get_backup_path()
    try:
        create_dir(backup_path)
        for database in backup_databases:
            backup_database(backup_path, database)
        zip_dir(backup_path)
    finally:
        shutil.rmtree(backup_path)


# 清理过期备份文件
def clean():
    expire_time = get_expire_time()
    for root, directories, files in os.walk(backup_dir):
        for file in files:
            if not file.startswith(backup_prefix):
                continue
            if not file.endswith(".zip"):
                continue
            file_path = os.sep.join([root, file])
            t = os.path.getctime(file_path)
            if t < expire_time:
                os.remove(file_path)


if __name__ == "__main__":
    try:
        backup()
    finally:
        clean()
```

#### sh
```
#!/bin/bash
FNAME=$(date +%Y%m%d%H%M%S)
cd /data/mysql/backup
mysqldump --add-drop-database -h 127.0.0.1 -uroot -ppassword --events --routines database | gzip > "mysql_backup_$FNAME.sql.gz"
find . -type f -name "*.sql.gz" -mtime +14 -exec rm -f {} \;
```

#### crontab
```
crontab -e
0 */6 * * * python /data/mysql/docker_mysql_backup.py
0 */6 * * * /bin/bash /data/mysql/mysqlbackup.sh >/dev/null 2>&1
```
