## docker部署MySQL8.0

```shell script
docker run --name mysql --restart=always -d -e MYSQL_ROOT_PASSWORD=Umisen@123 -p 3306:3306 mysql:8.0.20

grant all privileges on *.* to 'root'@'%' identified by 'password';

CREATE USER 'root'@'%' IDENTIFIED BY 'password'; 
GRANT ALL ON *.* TO 'root'@'%'; 
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
FLUSH PRIVILEGES;


mysql> CREATE USER 'root'@'%' IDENTIFIED BY 'password';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;

```

## `binlog`处理

### 临时删除

### 查看
```
show variables like "%expire_logs_days%";
```

### 设置保留7天

mysql5.x

```
set global expire_logs_days=7;
```

mysql8

```
set global binlog_expire_logs_seconds=604800;

set global binlog_expire_logs_seconds=259200;
259200

```

```shell script
purge master logs before'2021-09-05 17:20:00';

set global max_binlog_size=268435456;

set global max_binlog_size=1073741824;
```

### 配置文件

永久有效，需重启mysqld服务才生效

```
vim /etc/my.cnf
mysqld模块
expire_logs_days=7
```


## 导出

导出表

```shell script
mysqldump -u root -p RUNOOB runoob_tbl > dump.txt
```


拷贝其他主机

```shell script
mysqldump -h 172.16.xxx.xxx -P 13306 -u root -pyhblsqt@123 clouds-manager > clouds-manager.sql
```

## 导入

use 数据库名称

source 文件名称


## Mysql利用sqoop转hive

```shell script
sudo -H -u hive bash -c "sqoop import --connect jdbc:mysql://172.16.xx.xx:13306/test?useSSL=false --username root --password xxxx --table mission_result --fields-terminated-by '\t' --lines-terminated-by '\n'  --direct --delete-target-dir --num-mappers 1 --hive-import  --hive-overwrite --hive-database log --hive-table mission_result"


sudo -H -u hive bash -c "sqoop import --connect jdbc:mysql://172.16.xx.xx:3306/test?useSSL=false --username root --password xxxx --table account_view --fields-terminated-by '\t' --lines-terminated-by '\n'  --direct --delete-target-dir --num-mappers 1 --hive-import  --hive-overwrite --hive-database log --hive-table account"


sudo -H -u hive bash -c "sqoop import --connect jdbc:mysql://172.16.xx.xx:3307/yq?useSSL=false --username root --password xxxx#root --table task_single_dislike  --fields-terminated-by '\t' --lines-terminated-by '\n'  --direct --delete-target-dir --num-mappers 1 --hive-import  --hive-overwrite --hive-database log --hive-table task_single_dislike"

# mysql转hive binary

sudo -H -u hive bash -c "sqoop import --connect jdbc:mysql://172.16.xx.xx:13306/test?useSSL=false --username root --password xxxx --table account --fields-terminated-by '\t' --lines-terminated-by '\n'  --direct --delete-target-dir --num-mappers 1 --hive-import  --hive-overwrite --hive-database log --hive-table account --map-column-hive id=binary,user_id=binary"
```
