### MySQL数据库导入导出操作

#### 一、导出数据库

##### 1、导出sql文件

```shell
mysqldump -uroot -proot after_loan_20200619 > D:\databaseData\after_loan_20200619.sql
```

##### 2、导出dump文件

```shell
mysqldump -uroot -proot after_loan_20200619 > D:\databaseData\after_loan_20200619.dump
```

##### 3、导出数据库结果

```shell
# -d：没有数据；--add-drop-table：create语句钱增加drop table
mysqldump -uroot -proot -d --add-drop-table after_loan_20200619 > D:\databaseData\after_loan_20200702.sql
```

#### 二、导入数据库

##### 1、导入数据库文件(sql)

```mysql
-- 需要登录mysql控制台
source D:\databaseData\after_loan_20200619.sql
```

##### 2、导入数据库文件(dump)

```shell
mysql -uroot -p after_loan_dump < D:\databaseData\after_loan.dump
```

