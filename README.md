#### Mysql

##### 安装

- ##### 安装包

  链接: https://pan.baidu.com/s/1xQ84o2otnQw_NkBQ6IeRYg 提取码: 2198

- ##### 单实例安装过程

  ```shell
  # 创建用户组
  groupadd mysql
  # 创建用户
  useradd -r -g mysql mysql
  # 将安装包mysql-5.7.9-linux-glibc2.5-x86_64.tar.gz放到路径 /usr/local目录下
  cd /usr/local
  # 解压安装包
  tar -zxvf mysql-5.7.9-linux-glibc2.5-x86_64.tar.gz
  # 安装需要的依赖
  yum install -y libaio
  # 命令链接
  ln -s mysql-5.7.9-linux-glibc2.5-x86_64 mysql
  # 进入mysql目录
  cd mysql
  # 创建目录
  mkdir mysql-files
  # 授权
  chmod 770 mysql-files
  # 改变文件拥护者为mysql
  chown -R mysql .
  # 改变文件所属组为mysql
  chgrp -R mysql .
  # 创建data目录用于存放初始化的文件
  mkdir data
  # 初始化mysql(会得到root的初始化密码VP!N,1T)dyuy)
  bin/mysqld --initialize --user=mysql
  # 安装ssl
  bin/mysql_ssl_rsa_setup
  chown -R root .
  chown -R mysql data mysql-files
  # 查看my.cnf
  /usr/local/mysql/bin/mysqld --verbose --help |grep -A 1 'Default options' 
  # /etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf 
  # 删除文件 /etc/my.cnf
  rm -rf /etc/my.cnf
  # 后台启动mysql
  bin/mysqld_safe --user=mysql &
  # 设置开机启动
  cp support-files/mysql.server /etc/init.d/mysql.server
  # 查看开机启动清单以及开机启动mysql
  chkconfig --list
  chkconfig mysql.server on
  # 配置环境变量
  vim /etc/profile
  export PATH=/usr/local/mysql/bin:$PATH # 在文件最后一行添加
  # 让配置文件生效
  source /etc/profile
  # 登录mysql
  mysql -uroot -p'VP!N,1T)dyuy'
  # 修改root密码
  set password='root'
  # 设置权限允许远程登录
  grant all privileges on *.* to 'root'@'%' identified by 'root';
  # 权限生效
  flush privileges;
  # 查看防火墙状态centos7及以上
  systemctl status firewalld
  # 关闭防火墙
  systemctl stop firewalld
  ```

- ##### 多实例安装过程

  ```
  odoOCvSrc8;a
  
  Jq(po(7*hppf
  ```

##### 权限

- ##### 授权

  ```shell
  # 给用户dev，设置密码为123，并授权IP段为192.168.244.*，数据库select权限(这个需要在服务器段执行)
  grant SELECT on mall.* TO 'dev'@'192.168.244.%' IDENTIFIED BY '123' WITH GRANT OPTION;
  
  # 查看dev的权限
  show GRANTS for 'dev'@'192.168.244.%';
  
  # 给用户dev授权：查询，字段为id，name(这个需要在服务器段执行)
  grant select(id,name) on mall.account to 'dev'@'192.168.244.%';
  
  # 回收权限
  REVOKE select on mall.* from 'dev'@'192.168.244.%';
  
  # 授权之后记得
  flush privileges;
  ```

- ##### 用户标识

  在mysql中的权限不是赋予给用户的，而是赋予给**用户+IP**的，即用户标识为**用户+IP**

- ##### 用户权限所涉及的表

  - mysql.user：user的一行记录代表一个用户的标识
  - mysql.db：db的一行记录代表对数据库的权限
  - mysql.table_priv：table_priv的一行记录代表对表的权限
  - mysql.column_priv：column_priv的一行记录代表堆某一列的权限

- ##### 角色授权

  ```shell
  # 查看角色代理的参数
  SHOW VARIABLES like '%proxy%';
  # 设置以下两个参数为true，5.7之后才有
  set GLOBAL check_proxy_users = 1;
  set GLOBAL mysql_native_password_proxy_users = 1;
  # 创建角色dev_role
  create user 'dev_role';
  # 创建用户deer/enjoy/james，无密码
  create user 'deer';
  create user 'enjoy';
  create user 'james';
  # 查看deer的权限
  show GRANTS for 'deer';
  # 将角色的deer/enjoy/james的授权角色dev_role
  GRANT proxy on 'dev_role' to 'deer';
  GRANT proxy on 'dev_role' to 'enjoy';
  GRANT proxy on 'dev_role' to 'james';
  # 在服务器端执行远程链接可以授权代理
  GRANT PROXY ON ''@'' TO 'root'@'%' WITH GRANT OPTION;
  # 授权角色dev_role,select权限，字段id,name
  GRANT select(id,name) on mall.account to 'dev_role';
  # 权限生效
  FLUSH PRIVILEGES;
  ```

##### Mysql数据类型

- ##### Int类型

  | 类型      | 字节 | 范围                                      |
  | --------- | ---- | ----------------------------------------- |
  | tinyint   | 1    | 有符号（-128-127），无符号（0-255）       |
  | smallint  | 2    | 有符号（-32768-32767），无符号（0-65535） |
  | mddiumint | 3    | 有符号（-2^32-2^32-1），无符号（0-2^33）  |
  | int       | 4    | 有符号（-2^64-2^64-1），无符号（0-2^65）  |
  | bigint    | 5    | 有符号（-2^128-2^128-1），无符号（0-129） |

  - ##### 有无符号

    ```mysql
    CREATE TABLE test_unsigned (a INT UNSIGNED, b INT UNSIGNED);
    insert into test_unsigned values(1, 2);
    -- 使用UNSIGNED关键字之后，表示该字段为无符号的
    select b - a from test_unsigned; -- 1
    select a - b from test_unsigned; -- 运行报错
    ```

    **一般在项目中使用bigint，而且是有符号的**

  - ##### INT(N)

    ```mysql
    create table test_int_n(a int(4) zerofill);
    insert into test_int_n values(1);
    insert into test_int_n values(123456);
    -- 使用zerofill关键字，长度不够左边补0
    select * from test_int_n; -- 需要在服务端查询才能显示效果
    ```

    INT(N)中的N是显示宽度，不表示存储的数字的长度的上限；

    zerofill表示当存储的数字长度<N时，用数字0填充昨天，直至补满长度N；

    当存储数字的长度超过N时，按照实际存储的数字显示。

  - ##### 自动增长的问题

    ```
    -- 创建自动增长，必须设置主键
    create table test_auto_increment(a int auto_increment primary key);
    insert into test_auto_increment values(NULL);-- 1
    insert into test_auto_increment values(0);-- 0
    insert into test_auto_increment values(-1); -- 2
    insert into test_auto_increment values(null),(100),(null),(10),(null); -- 3 100 101 10 102
    ```

- ##### 字符类型

  | 类型          | 说明           | N的含义 | 是否有字符集 | 最大长度 |
  | ------------- | -------------- | ------- | ------------ | -------- |
  | CHAR(N)       | 定长字符       | 字符    | 是           | 255      |
  | VARCHAR(N)    | 变长字符       | 字符    | 是           | 16384    |
  | BINARY(N)     | 定长二进制字节 | 字节    | 否           | 255      |
  | VARBINARY(N)  | 变长二进制字节 | 字节    | 否           | 16384    |
  | TINYBLOB(N)   | 二进制大对象   | 字节    | 否           | 256      |
  | BLOB(N)       | 二进制大对象   | 字节    | 否           | 16K      |
  | MEDIUMBLOB(N) | 二进制大对象   | 字节    | 否           | 16M      |
  | LONGBLOB(N)   | 二进制大对象   | 字节    | 否           | 4G       |
  | TINYTEXT(N)   | 大对象         | 字节    | 是           | 256      |
  | TEXT(N)       | 大对象         | 字节    | 是           | 16K      |
  | MEDIUMTEXT(N) | 大对象         | 字节    | 是           | 16M      |
  | LONGTEXT(N)   | 大对象         | 字节    | 是           | 4G       |

- ##### 时间类型

  | 日期类型  | 占用空间 | 表示范围                                          |
  | --------- | -------- | ------------------------------------------------- |
  | DATETIME  | 8        | 1000-01-01 00:00:00 ~ 9999-12-31 23:59:59         |
  | DATE      | 3        | 1000-01-01 ~ 9999-12-31                           |
  | TIMESTAMP | 4        | 1970-01-01 00:00:00UTC ~ 2038-01-19   03:14:07UTC |
  | YEAR      | 1        | YEAR(2):1970-2070, YEAR(4):1901-2155              |
  | TIME      | 3        | -838:59:59 ~ 838:59:59                            |

  datatime与timestamp的区别：

  datatime与时区无关；timestamp与时区有关；时间范围不一样；

- 