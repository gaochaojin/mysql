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
  grant all privileges on *.* to 'root'@'%' identified by 'root' WITH GRANT OPTION;
  # 权限生效
  flush privileges;
  # 查看防火墙状态centos7及以上
  systemctl status firewalld
  # 关闭防火墙
  systemctl stop firewalld
  ```

- ##### 多实例安装过程

  - ##### 创建配置文件/etc/my.cnf
  
    ```shell
    [mysqld]
    sql_mode = "STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER"
    
    [mysqld_multi]
    mysqld = /usr/local/mysql/bin/mysqld_safe
    mysqladmin = /usr/local/mysql/bin/mysqladmin
    log = /var/log/mysqld_multi.log
    user=root
    pass=root
    
    [mysqld1] 
    server-id = 11
    socket = /tmp/mysql.sock1
    port = 3307
    datadir = /data1
    user = mysql
    performance_schema = off
    innodb_buffer_pool_size = 32M
    skip_name_resolve = 1
    log_error = error.log
    pid-file = /data1/mysql.pid1
    [mysqld2]
    server-id = 12
    socket = /tmp/mysql.sock2
    port = 3308
    datadir = /data2
    user = mysql
    performance_schema = off
    innodb_buffer_pool_size = 32M
    skip_name_resolve = 1
    log_error = error.log
    pid-file = /data2/mysql.pid2
    ```
  
  - ##### 安装过程
  
    ```shell
    # 创建数据文件夹data1/data2
    mkdir /data1
    mkdir /data2
    # 改变文件夹data1/data2的拥有者
    chown mysql.mysql /data{1..2}
    # 配置开机启动
    cp /usr/local/mysql/support-files/mysqld_multi.server /etc/init.d/mysqld_multid
    
    # 在/etc/init.d/mysqld_multid文件中添加配置
    export PATH=/usr/local/mysql/bin:$PATH
    /usr/local/mysql/support-files/mysqld_multi.server start
    
    chkconfig mysqld_multid on
    # 查看状态
    mysqld_multi report
    # 安装perl环境
    yum -y install perl perl-devel
    # 启动多实例
    mysqld_multi start
    # 登录并修改密码，允许远程链接
    mysql -u root -S /tmp/mysql.sock1 -p -P3307 
    mysql -u root -S /tmp/mysql.sock2 -p -P3308
    
    set password = 'root1234%';
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' with grant option;
    flush privileges; 
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

- ##### JSON类型

  - ##### 数据类型json

    ```mysql
    -- 创建表
    CREATE TABLE json_user (
    	uid INT auto_increment,
    	DATA json,
    	PRIMARY KEY (uid)
    );
    -- 插入数据
    insert into json_user values (
    null, '{
       "name":"lison",
    "age":18,
    "address":"enjoy"
       }' );
    
     insert into json_user values (
     null,
     '{
      "name":"james",
      "age":28,
      "mail":"james@163.com"
     }');
    
    ```

  - ##### json函数

    ```mysql
    -- json_extract 抽取
    SELECT json_extract ('[10,20,[30,40]]', '$[1]'); -- 20
    SELECT
    	json_extract (DATA, '$.name'),
    	json_extract (DATA, '$.address')
    FROM
    	json_user;
    
    -- json_object 将对象转为json
    select json_object("name","enjoy","email","enjoy.com","age","25");-- {"age": "25", "name": "enjoy", "email": "enjoy.com"}
    
    insert into json_user values ( null,json_object("name", "enjoy", "email", "enjoy.com", "age",35));
    
    -- json_insert 插入数据
    set @json = '{"a":1,"b":[2,3]}'; -- 局部变量 @@a：系统变量
    
    select json_insert(@json,'$.a',10,'$.c','[true,false]');
    
    UPDATE json_user set data = json_insert(data,'$.address_2','xiangxue') where uid =1;
    
    -- json_merge 合并数据并返回
    select json_merge('{"name":"enjoy"}','{"id":47}');
    
    select json_merge(
    json_extract(data,'$.address'),
    json_extract(data,'$.address_2')) 
    from json_user where uid = 1;
    
    -- json_unquote 
    set @j = '"abc"';
    select json_unquote(@j); -- abc
    ```

- ##### JSON索引

  JSON类型数据本身无法直接创建索引，需要将需要索引的JSON数据重新生成虚拟列之后，对该列进行索引

  ```mysql
  create table test_index_1(
    data json,
    gen_col varchar(10) generated always as (json_extract(data, '$.name')), 
  index idx (gen_col) 
   );
  
  insert into test_index_1(data) values ('{"name":"king", "age":18, "address":"cs"}');
  insert into test_index_1(data) values ('{"name":"peter", "age":28, "address":"zz"}');
  
  select * from test_index_1;
  
  select json_extract(data,"$.name") as username from test_inex_1 where gen_col='"king"';-- 查询的时候需要为"king"
  
  -- 虚拟列，这个查询的时候不需要加上''符号即可查到数据
  create table test_index_2 (
   data json,
   gen_col varchar(10) generated always as (
   json_unquote( 
   json_extract(data, "$.name")
   )),
   key idx(gen_col)
   );
  
  insert into test_index_2(data) values ('{"name":"king", "age":18, "address":"cs"}');
  insert into test_index_2(data) values ('{"name":"peter", "age":28, "address":"zz"}');
  
  select json_extract(data,"$.name") as username from test_index_2 where gen_col="king";
  ```

##### mysql架构

- ##### 体系

  ![1560994734245](.\images\体系.png)

  - ##### 连接层

    ![](.\images\连接层1.png)

    当mysql启动（mysql服务器就是一个进程），等待客户端链接，每一个客户端链接请求，服务器斗湖i新建一个线程处理（如果是线程池的话，则是分配一个空的线程），每个线程独立，拥有各自的呢次村处理空间。

    ![1560995571509](.\images\连接层2.png)

    连接到服务器，服务器需要对其进行验证，也就是用户名、ip、密码验证，一旦链接成功，还要验证是否具有某个特定查询的权限

  - ##### SQL处理层

    ![1560995758601](.\images\SQL处理层.png)

    这一层主要功能有：SQL语句的解析、优化、缓存的查询，mysql内置函数的实现，跨存储引擎功能（所谓跨存储引擎就是每个引擎都是提供的功能）

    1. 如果是查询语句（select语句），首先会查询缓存是否已有相应结果，有则返回结果，无则进行下一步（如果不是查询语句，同样调到下一步）
    2. 解析查询，创建一个内部数据结构（解析数），这个解析数主要用来SQL语句的语义与语法解析
    3. 优化，优化SQL语句，例如重写查询，决定表的读取顺序，以及选择需要的索引等。这一阶段用户是可以查询的，插叙服务器是如何进行优化的，便于用户重构查询和修改相应的配置，达到最优化。这一阶段还涉及到存储引擎，优化器会询问存储引擎。

    - ##### 缓存

      ```mysql
      show variables like  '%query_cache_type%';   -- 默认不开启
      show variables like  '%query_cache_size%';  -- 默认值1M
      SET GLOBAL query_cache_type = 1; -- 会报错
      -- query_cache_type只能配置在my.cnf文件中，这大大限制了qc的作用
      
      -- 在生产环境建议不开启，除非经常有sql完全一模一样的查询(但是这种情况可以在应用层做缓存redis)
      
      -- QC严格要求2次SQL请求要完全一样，包括SQL语句，连接的数据库、协议版本、字符集等因素都会影响
      
      ```

    - ##### 解析查询

      ```mysql
      SELECT DISTINCT
      	<select_list>
      FROM
      	<left_table> <join_type>
      JOIN <right_table> ON <join_condition>
      WHERE
      	<where_condition>
      GROUP BY
      	<group_by_list>
      HAVING
      	<having_condition>
      ORDER BY
      	<order_by_condition>
      LIMIT <limit_number>
      ```

      ![1560997001596](.\images\解析查询.png)

    - ##### 优化

      ```mysql
      explain
      select * from account where name = ''; -- using where
      
      explain
      select * from account where 1=1; -- null
      
      explain 
      select * from account where id is null; -- impossible where
      
      -- 以下两个sql式一样的
      select * from account t where t.id  in (select t2.id from account t2);
      select t.* from account t join account t2 on t.id = t2.id;
      ```

      

- ##### 逻辑架构

  ![1560997611430](.\images\逻辑架构.png)

  在mysql中其实还有schema的概念，没有太多作用，只是为了兼容其他数据库，   database和schema是等价的

  ```mysql
  create database demo;
  show databases;
  drop schema demo;
  show databases;
  ```

- ##### 物理存储结构

  - ##### 数据库的数据库

    ```mysql
    -- 数据库存放位置
    show variables like '%datadir%'; -- /usr/local/mysql/data/
    ```

  - ##### 数据库

    创建了一个数据库后，会在datadir目录新建一个子文件夹（文件夹名为数据库名称）

  - ##### 表文件

    进入数据库文件里面，表文件和具体的存储引擎相关，但有个共同的就是都有一个frm文件，存放的是表的数据格式。

    ```mysql
    -- 使用mysqlfrm可以通过frm文件查看表结构
    mysqlfrm -diagnostic /usr/local/mysql/data/mall/account.frm
    ```

  - ##### mysql utilities 安装

    安装包：链接: https://pan.baidu.com/s/1X13Iw05BxAVmjZhAbdvFfQ 提取码: ct15

    ```shell
    -- 将安装包mysql-utilities-1.6.5.tar.gz放到路径/opt/soft/下
    tar -zxvf mysql-utilities-1.6.5.tar.gz
    cd mysql-utilities-1.6.5
    python ./setup.py build
    python ./setup.py install
    ```

##### 存储引擎

```mysql
-- mysql提供的储存引擎
show engines;
-- 查看mysql当前默认存储引擎
show variables like '%storage_engine%';
```

- ##### MyISAM

  **Mysql5.5**之前默认的存储引擎，MyISAM存储引擎由MYD和MYI组成

  ```
  -- 创建表指定存储引擎为myisam
  create table testmysam (
    id int PRIMARY key
  ) ENGINE=myisam;
  insert into testmysam  VALUES(1),(2),(3);
  ```

  - ##### 表组成

    testmysam.frm----结构文件

    testmysam.MYD----数据文件

    testmysam.MYI----索引文件

  - ##### 表压缩

    ```shell
    -- 压缩命令
    myisampack -b -f /usr/local/mysql/data/mall/testmysam
    -- 压缩后的表不能插入数据
    -- 压缩后需要修复数据
    myisamchk -r -f /usr/local/mysql/data/mall/testmysam.MYI
    ```

  - ##### 适用场景（由于现在innodb越来越强大，myisam已经停止维护，绝大多数场景都不适合）

    - 非事务型应用（数据仓库，报表，日志数据）
    - 只读类应用
    - 空间类应用（空间函数，坐标）

- ##### Innodb

  - innodb是一种事务型存储引擎

  - 完全支持事务的ACID特性（原子性，一致性，隔离性，持久性）

  - Redo Log和Undo Log

  - innodb支持行级锁，并发程度高

  - 查看innodb_log_buffer_size

    ```mysql
    show VARIABLES like 'innodb_log_buffer_size';
    ```

  - 与MyISAM的区别

    ![1561081273656](.\images\Innodb和MyIsam的区别.png)

- ##### CSV

  ```mysql
  create table mycsv(id int not null,c1 VARCHAR(10) not null,c2 char(10) not null) engine=csv;
  insert into mycsv values(1,'aaa','bbb'),(2,'cccc','dddd');
  -- 可以直接修改文本文件mycsv.CSV,然后需要flush tables才能生效
  flush TABLES;
  select * from mycsv;
  -- 不支持索引
  create index idx_id on mycsv(id);
  ```

  - 以csv格式进行数据存储
  - 所有列都不能为null
  - 不支持索引（不适合达标，不适合在线处理）
  - 可以对数据文件直接编辑（保存文本文件内容）

- ##### Archive

  ```mysql
  create table myarchive(id int auto_increment not null,c1 VARCHAR(10),c2 char(10), key(id)) engine = archive;
  -- 不允许在非自增ID列上加索引
  create index idx_c1 on myarchive(c1);
  INSERT into myarchive(c1,c2) value('aa','bb'),('cc','dd');
  -- 只支持insert和select操作
  delete from myarchive where id = 1;
  update myarchive set c1='aaa' where id = 1;
  ```

  - 组成：以zlib对表数据进行压缩，磁盘I/O更少，数据存储在ARZ为后缀的文件中
  - 特点：只支持insert和select操作；只允许在自增ID列上加索引。

- ##### Memory

  ```mysql
  -- 不支持大字段Blog和text等
  create table mymemory(id int,c1 varchar(10),c2 char(10),c3 text) engine = memory;
  
  create table mymemory(id int,c1 varchar(10),c2 char(10)) engine = memory;
  -- 创建hash索引（默认值）
  create index idx_c1 on mymemory(c1);
  -- 创建btree索引
  create index idx_c2 using btree on mymemory(c2);
  show index from mymemory;
  -- 查看表状态
  show TABLE status LIKE 'mymemory';
  ```

  - 特点

    - 文件系统存储特点，也称HEAP存储引擎，所以数据保存在内存中
    - 支持hash索引和btree索引
    - 所有字段都是固定长度 varchar(10)=char(10)
    - 不支持Blog和Text等大字段
    - Memory存储引擎使用表级锁
    - 最大大小由max_heap_table_size参数决定

  - 与临时表的区别

    ![](.\images\memory与临时表的区别.png)

  - 使用场景

    - hash索引用于查找或者是映射表（邮编和地区的对应表）
    - 用于保存数据分析中产生的中间表
    - 用于混村周期性聚合数据的结果表

- ##### Federated

  ```shell
  -- federated引擎默认禁止的，启用需要再启动时增加federated参数
  vi /etc/my.cnf
  
  -- 然后在[mysqld]下添加federated，然后重启即可
  ```

  ```mysql
  -- 创建数据库local和remote
  create database local;
  create database remote;
  
  -- 在remote数据库中创建表及插入数据（表引擎为innode）
  create table remote_fed(id int auto_increment not null,c1 varchar(10) not null default '',c2 char(10) not null default '',primary key(id)) engine = INNODB；
  
  INSERT into remote_fed(c1,c2) values('aaa','bbb'),('ccc','ddd'),('eee','fff');
  
  -- 在local数据库中创建表（表引擎为federated）
  CREATE TABLE `local_fed` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `c1` varchar(10) NOT NULL DEFAULT '',
    `c2` char(10) NOT NULL DEFAULT '',
    PRIMARY KEY (`id`)
  ) ENGINE=FEDERATED CONNECTION ='mysql://root:root@127.0.0.1:3306/remote/remote_fed';
  
  -- 可以通过local数据库查询到remote数据库的数据
  select * from local_fed;
  
  delete from local_fed where id =  2;
  
  select * from remote.remote_fed;
  ```

  - 特点

    - 提供了访问远程mysql服务器上表的方法

    - 本地不存储数据，数据全部放到远程服务器上

    - 本地需要保存表结构和远程服务器的连接信息

  - 使用场景

    偶尔的统计分析及手工查询（某些游戏行业）

##### 锁

- ##### 锁的简介

  - ##### 锁的概念

    锁是计算机协调多个进程或线程并发访问某一资源的机制。

    在数据库中，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。

    锁对数据库而言显得尤其重要，也更加复杂。

  - ##### Mysql中的锁

    - 表级锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。
    - 行级锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。
    - 页面锁（gap锁，间隙锁）：开锁和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。

  - ##### 表锁与行锁的使用场景

    表级锁更适合于以查询为主，只要少量按索引条件更新数据的应用，如OLAP系统（在线分析处理）；

    行级锁则更适合于有大量按索引条件并发更新少量不同数据，同事又有并发查询的应用，如OLTP系统（在线事务处理）

- ##### MyISAM锁

  MySQL的表级锁有两种模式：

  表共享读锁（Table Read Lock）

  表独占写锁（Table Write Lock）

  ![1561102488417](.\images\表共享锁和独占锁.png)

  - 共享读锁

    **语法**：lock table 表名 read

    ```mysql
    -- 以下实验在5.7中不能成功，在5.6中可以成功
    -- 对表testmysqm进行共享读锁
    lock table testmysam read;
    
    -- 在同一session中可以查询；在另一个session中可以查询
    select * from testmysam;-- success
    
    -- 在同一session中会报错；在另一个session中处于等待wait状态
    insert into testmysam value (4); -- error 
    update testmysam set id = 5 where id = 1; -- error 
    
    -- 在同一session中会报错；在另一个session中成功success
    insert into account value(4,'aa',123); -- error 
    select  * from account  ; -- error 
    select s.* from testmysam s; -- error 
    ```

  - 独占写锁

    ```mysql
    -- 对表testmysam进行设置写锁
    lock table testmysam WRITE;
    
    -- 同一session中可以成功；另一个session中等待wait
    insert testmysam value(4);
    delete from testmysam where id = 3;
    select * from testmysam;
    
    -- 同一session中对不同的表报错
    select s.* from  testmysam s;
    insert into account value(4,'aa',123);
    ```

  - ##### 总结

    - 读锁，对MyISAM表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求
    - 读锁，对MyISAM表的读操作，不会阻塞当前session对表读，当对表进行修改会报错
    - 读锁，一个session使用lock table命令给表加了读锁，这个session可以查询锁定表中的记录，但更新或访问其他表都会提示错误；
    - 写锁，对MyISAM表的写操作，则会阻塞其他用户对同一表的读和写操作
    - 写锁，对MyISAM表的写操作，当前session可以对本表做CRUD，但对其他表进行操作会报错

- ##### InnoDB锁

  在mysql的InnoDB引擎支持行锁

  共享锁：读锁，当一个事务对某几行上读锁时，允许其他事务对这几行进行读操作，但不允许其进行写操作，也不允许其他事务给这几行上排它锁，但允许上读锁。

  排它锁：写锁，当一个事务对某几行上写锁时，不允许其他事务写，但允许读。更不允许其他事务给这几行上任何锁，包括写锁。

  - 语法

    ```mysql
    -- 共享锁语法
    lock in share mode
    select * from account where id = 1 lock in share mode;
    -- 排它锁语法
    for update
    select * from account where id = 2 for update;
    ```

  - 注意事项

    - 两个事务不能锁同一个索引

    - insert、delete、update在事务中都会自动默认加上排它锁

    - 行锁必须有索引才能实现，否则会自动锁全表，那么就不是行锁了

      ```mysql
      -- 创建表并插入数据
      CREATE TABLE testdemo (
      `id`  int(255) NOT NULL ,
      `c1`  varchar(300) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL ,
      `c2`  int(50) NULL DEFAULT NULL ,
      PRIMARY KEY (`id`),
      INDEX `idx_c2` (`c2`) USING BTREE 
      )
      ENGINE=InnoDB;
      
      insert into testdemo VALUES(1,'1',1),(2,'2',2);
      
      -- 开启写锁
      begin；
      select * from testdemo where id =1 for update;
      -- 在另外一个session
      update testdemo set c1 = '1' where id = 2; -- success
      update testdemo set c1 = '1' where id = 1; -- wait
      -- 在第一个session中执行以下任一命令，wait--》success
      commit;
      rollback;
      
      
      -- insert、delete、update在事务中都会自动默认加上排它锁
      BEGIN;
      update testdemo set c1 = '1' where id = 1;
      -- 在另外一个session中
      update testdemo set c1 = '1' where id = 1; -- wait
      
      
      -- 行锁必须有索引才能实现，否则会自动锁全表
      BEGIN;
      update testdemo set c1 = '1' where  c1 = '1';
      -- 在另外一个session中
      update testdemo set c1 = '2' where  c1 = '2';
      
      
      -- 在一个session中
      select * from testdemo where id = 1 for update;
      -- 第二个session中
      select * from testdemo where id = 1 lock in share mode; -- wait
      -- 回到第一个session中使用unlock tables不能解锁，可以使用commit,begin,rollback进行解锁
      
      -- 表锁
      lock table testdemo WRITE;
      -- 使用commit，rollback不能解锁，只能使用unlock tables，begin才能解锁
      ```

- ##### 锁的等待问题

  ```mysql
  -- 在第一个session中
  BEGIN;
  SELECT * FROM testdemo WHERE id = 1 FOR UPDATE;
  -- 在第二个session中
  BEGIN;
  SELECT * FROM testdemo WHERE id = 1 lock in share mode;-- wait
  
  -- 在这种情况下可以通过以下sql查看是那条数据被锁住了
  select * from information_schema.INNODB_LOCKS;
  -- 同一个数据有两个锁，X（写锁），S（读锁）
  -- 解决方法
  -- 5.7及以上版本
  select * from sys.innodb_lock_waits; -- 记录中有一个字段sql_kill_blocking_connection,获取他的值a，然后有以下命令杀死这个锁
  kill a;
  -- 5.6及之前版本
  SELECT
    r.trx_id waiting_trx_id,
    r.trx_mysql_thread_id waiting_thread,
    r.trx_query waiting_query,
    b.trx_id blocking_trx_id,
    b.trx_mysql_thread_id blocking_thread
  FROM
    information_schema.innodb_lock_waits w
  INNER JOIN
    information_schema.innodb_trx b ON b.trx_id = w.blocking_trx_id
  INNER JOIN
    information_schema.innodb_trx r ON r.trx_id = w.requesting_trx_id;
    
  -- 获取blocking_thread的值b，使用以下命令杀死锁
  kill b;
  ```

##### 事务

- ##### 支持事务的存储引擎

  ```mysql
  -- 查看数据库下面是否支持事务（InnoDB支持）
  show engines；
  
  -- 查看mysql当前默认存储引擎
  show variables like '%storage_engine%';
  
  -- 查看某张表的存储引擎
  show create table 表名;
  
  -- 对于表的存储结构修改
  create table table_name ...... type=InnoDB;
  alter table table_name type=InnoDB;
  ```

- ##### 事务特性

  事务具有4个属性：原子性、一致性、隔离性、持久性，称为ACID

  - ##### 原子性（atomicity）

    一个事务必须被视为一个不可分割的最小单元，整个事务中的所有操作要么全部提交成功，要么全部失败，对于一个事务来说，不可能只执行其中的一部分操作。

  - ##### 一致性（consistency）

    一致性是指事务将数据库从一种一致性转换到另外一种一致性状态，在事务开始之前和事务结束之后数据库中的完整性没有被破坏。

  - ##### 隔离性（isolation）

    一个事务的执行不能被其他事务干扰。即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。（对数据库的并行执行，应该像串行执行一样）

  - ##### 持久性（durability）

    一旦事务提交，则其所做的修改就会永久保存到数据库中。此时即使系统崩溃，已提交的修改数据也不会丢失。

- ##### 事务的隔离级别

  **mysql默认的事务隔离级别为：REPEATABLE-READ**

  ```mysql
  show variables like '%tx_isolation%';
  ```

  - ##### 未提交读（READ UNCOMMITED）脏读

    ```mysql
    -- 设置事务隔离级别（分别在第一个和第二个session中）
    set session transaction isolation level read uncommitted;
    -- 开启事务（分别在第一个和第二个session中）
    start transaction;
    -- 在第一个session中执行以下操作 原始值balance = 900
    update account set balance = balance -50 where id = 1;
    select * from account;-- balance = 850
    -- 在第二个session中执行
    select * from account;-- balance = 850
    -- 在第一个session中执行
    rollback；
    -- 在第二个session中执行
    select * from account;-- balance = 900 
    
    -- 在第二个session中读取到了未提交的数据，这部分的数据为脏数据
    ```

  - ##### 已提交读（READ COMMITTED）不可重复读

    ```mysql
    -- 设置事务隔离级别
    set session transaction isolation level read committed;
    -- 开启事务
    start transaction;
    -- 在第一个session中执行以下操作 原始值balance=850
    update account set balance = balance -50 where id = 1;
    select * from account;-- balance = 800
    -- 在第二个session中执行
    select * from account;-- balance = 850
    -- 在第一个session中执行
    commit；
    -- 在第二个session中执行
    select * from account;-- balance = 800 
    
    -- 不可重复读解决了脏读问题
    ```

  - ##### 可重复读（REPEATED READ）

    ```mysql
    -- 设置事务隔离级别
    set session transaction isolation level repeatable read;
    -- 开启事务
    start transaction;
    -- 在第一个session中执行以下操作 原始值balance=750
    update account set balance = balance -50 where id = 1;
    select * from account;-- balance = 700
    -- 在第二个session中执行
    select * from account;-- balance = 750
    -- 在第一个session中执行
    commit；
    -- 在第二个session中执行
    select * from account;-- balance = 750 
    -- 在第二个session中执行
    commit;
    -- 在第二个session中执行
    select * from account; -- balance  = 700
    
    -- 可重复读解决了不可重复读，在mysql数据库中解决了幻读（其他数据库中为解决幻读）
    ```

  - ##### 可串行化（SERIALIZABLE）

    ```mysql
    -- 设置事务隔离级别
    set session transaction isolation level serializable;
    -- 开启事务
    begin;
    -- 在第一个session中执行
    select * from account; -- 5条数据
    -- 在第二个session中执行
    select * from account; -- 5条数据
    -- 在第二个session中执行
    insert into account  values (6,'ert','600');-- wait
    -- 在第一个session中执行
    commit; -- 第二个session中wait的语句--》success
    -- 在第一个session中执行
    select * from account; -- 5条数据
    -- 在第二个session中执行
    commit;
    -- 在第一个session中执行
    select * from account; -- 6条数据
    
    -- 可串行化可解决幻读
    ```
    
  - ##### 间隙锁（gap锁）

    在mysql中，可重复锁已经解决了幻读问题，借助的就是间隙锁
    
    ```mysql
    -- 实例1
    -- 查看事务隔离级别
    select @@tx_isolation;
    -- 创建表和插入数据
    create table t_lock_1 (a int primary key);
    insert into t_lock_1 values(10),(11),(13),(20),(40);
    -- 查看数据
    select * from t_lock_1;
    -- 开启事务 在第一个session中
    BEGIN;
    -- 写锁 在第一个session中
    select * from t_lock_1 where a<=13 for update; -- 10 11 13
    -- 在第二个session中插入数据
    insert into t_lock_1 values (21);-- success
    insert into t_lock_1 values (19);-- wait
    
    -- 在repeated read隔离级别中会扫描到当前值（13）的下一个值（20），并把这些数据全部加锁
    
    
    -- 实例2
    -- 创建表和插入数据
    create table t_lock_2 (a int primary key,b int, key (b));
    insert into t_lock_2 values(1,1),(3,1),(5,3),(8,6),(10,8);
    
    select * from t_lock_2;
    -- 开启事务 在第一个session中
    BEGIN;
    -- 添加写锁 在第一个session中
    select * from t_lock_2 where b = 3 for update;
    
    -- 在第二个session中
    select * from t_lock_2 where a = 5 lock in share mode;-- wait
    
    insert into t_lock_2 values (4,2); -- wait,b=2在（1，3）内
    
    insert into t_lock_2 values (6,5); -- wait,b=2在（3，6）内
    
    insert into t_lock_2 values (2,0); -- success
    
    insert into t_lock_2 values (6,7); -- success
    
    insert into t_lock_2 values (9,6); -- success
    
    insert into t_lock_2 values (7,6); -- wait
    
    -- 主键索引只锁住了a=5的这条记录
    -- 二级索引所著的范围是（1，3），（3，6）
    ```
    
    

- ##### 事务并发问题

  - 脏读：事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据。

  - 不可重复读：事务A多次读取同一数据，事务B在事务A多次读取的过程中，对数据做了更新并提交，导致事务A多次读取同一数据时，结果不一致。

  - 幻读：系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入了一条具体分数的记录，当系统管理员A改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就是幻读。

    **注意：不可重复读和幻读很容易混淆，不可重复读侧重于修改，幻读侧重于新增和删除。解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表。**
  
- ##### 事务语法

  ```mysql
  -- 开启事务
  begin;
  start transaction;
  begin work;
  
  -- 事务回滚
  rollback;
  
  -- 事务提交
  commit;
  
  -- 还原点
  savepoint;-- 语法
  show variables like '%autocommit%';-- 查看事务，默认为自动提交 ON
  
  set autocommit = 0;-- 设置事务不自动提交 OFF
  
  select * from testdemo;
  
  insert into testdemo values(5,5,5);
  savepoint s1;-- 还原点s1
  insert into testdemo values(6,6,6);
  savepoint s2;-- 还原点s2
  insert into testdemo values(7,7,7);
  savepoint s3;-- 还原点s3
  
  rollback to savepoint s2; -- 将事务还原到还原点s2，（7，7，7）数据没有了
  commit; -- 提交事务，提交事务之后，还原点就失效了
  ```

##### 逻辑设计

- ##### 范式设计

  - ##### 数据库设计第一范式：数据库表中的所有字段都只具有单一属性，单一属性的列是由基本数据类型所构成的。

  - ##### 数据库设计第二范式：要求表中只具有一个业务主键，即符合第二范式的表不饿能存在非主键列只对部分主键的依赖关系。

  - ##### 数据库设计第三范式：每一个非主属性既不部分依赖于也不传递于业务主键，也就是在第二范式的基础上相处了非主键对主键的传递依赖。

    **备注：**完全符合范式化的设计有时并不能得到良好的SQL查询性能

- ##### 反范式设计

  - ##### 所谓反范式化就是为了性能和读取效率的考虑而适当的对数据库设计范式要求进行违反。

  - ##### 允许存在少量的冗余，即反范式化就是使用空间来换取时间。

- ##### 范式化设计优缺点

  - 优点：可以尽量的减少数据冗余；范式化的更新操作比反范式化更快；范式化的表通常比反范式化的表更小。
  - 缺点：对于查询需要对多个表进行关联；更难进行索引优化

- ##### 反范式化设计优缺点

  - 优点：可以减少表的关联；可以更好的进行索引优化
  - 缺点：存在数据冗余及数据维护异常；对数据的修改徐奥更多的成本。

##### 物理设计

- ##### 命名规范

  - 数据库、表、字段的命名要遵守可读性原则（使用大小写来格式化的库对象名字以获得良好的可读性）
  - 数据库、表、字段的命名要遵守表意性原则（对象名字应该能够顾描述它所表示的对象）
  - 数据库、表、字段的命名要遵守长名要遵守长名原则（尽可能少使用或者不使用缩写）

- ##### 存储引擎选择

  ![1561449789233](.\images\引擎区别.png)

- ##### 数据类型选择

  当一个可以选择多种数据类型时:

  - ##### 优先考虑数字类型

  - ##### 其次是日期、时间类型

  - ##### 最后是字符类型

  - ##### 对于相同级别的数据类型，应该优先选择占用空间小的数据类型

    浮点类型

    | 列类型  | 存储空间                            | 是否精确类型 |
    | ------- | ----------------------------------- | ------------ |
    | FLOAT   | 4个字节                             | 否           |
    | DOUBLE  | 8个字节                             | 否           |
    | DECIMAL | 每4个字节存9个数字，小数点占1个字节 | 是           |

    注意：float和double是非精确度类型，如果是和金额相关尽量用decimal

##### 慢查询

- ##### 概念

  就是查询慢的日志，是指mysql记录所有执行超过long_query_time参数设定的时间阈值的SQL语句的日志。该日志能为SQL语句的优化带来很好的帮助。默认情况下，慢查询日志是关闭的，要使用慢查询日志功能，首先要开启慢查询日志功能。

- ##### 慢查询配置

  - 慢查询基本配置

    - show_query_log：启动停止慢查询日志

    - show_query_log_file：指定慢查询日子和存储路径及文件（默认和数据文件放一起）

    - long_query_time：指定记录慢查询日志SQL执行时间的阈值（单位：秒，默认10秒）

    - log_queries_not_using_indexes：是否记录未使用索引的SQL

    - log_output：日志存放的地方（table，file，file-table）

      配置了慢查询后，它会记录符合条件的SQL，包括：查询语句、数据修改语句、已经回滚的SQL

      ```mysql
      show VARIABLES like '%slow_query_log%';
      
      show VARIABLES like '%slow_query_log_file%';
      
      show VARIABLES like '%long_query_time%';
      
      show VARIABLES like '%log_queries_not_using_indexes%';
      
      show VARIABLES like 'log_output';
      
      
      set global long_query_time=0;   -- 默认10秒，这里为了演示方便设置为0 
      
      set GLOBAL  slow_query_log = 1; -- 开启慢查询日志
      
      set global log_output='FILE,TABLE'  -- 项目开发中日志只能记录在日志文件中，不能记表中
      
      -- 慢查询日志文件路径为：/usr/local/mysql/data/localhost-slow.log
      ```

  - 慢查询解读

    ```mysql
    -- 查看日志文件/usr/local/mysql/data/localhost-slow.log里面的数据组成如下：
    -- 用户名、用户的IP信息、线程ID号
    # User@Host: root[root] @  [192.168.244.1]  Id:    30
    -- 执行花费的时间（单位：毫秒）、执行获取锁的时间、获得的结果行数
    # Query_time: 0.000298  Lock_time: 0.000120 Rows_sent: 0  
    -- 扫描的数据行数
    Rows_examined: 0
    -- SQL执行的具体时间
    SET timestamp=1561452234;
    -- 具体的SQL时间
    SELECT * FROM `mall`.`account` LIMIT 0;
    ```

- ##### 慢查询分析

  慢查询的日志记录非常多，通过一些工具辅助快速定位到需要优化的SQL语句，如下：

  - ##### mysqldumpslow：常用的慢查询日志分析工具，，汇总除查询条件外其他完全相同的SQL，并将分析结果按照参数中所制定的顺序输出

    ```mysql
    -- 语法
    mysqldumpslow -s r -t 10 localhost-slow.log
    -- 其中 -s order（c-总次数，t-总时间，l-锁的时间，r-总数据行，at-平均查询时间，al-平均锁定时间，ar-平均返回记录时间）-t top 执行去前面几条作为结果输出
    ```

  - ##### pt-query-digest：是用于分析mysql慢查询的一个工具

    - 安装

      ```shell
      -- pt-query-digest依赖包
      yum -y  install 'perl(Data::Dumper)';
      yum -y install perl-Digest-MD5;
      yum -y install perl-DBI;
      yum -y install perl-DBD-MySQL;
      
      -- 安装pt-query-digest文件
      yum -y install wget
      wget percona.com/get/pt-query-digest
      
      -- 执行慢查询命令
      perl ./pt-query-digest --explain h=127.0.0.1,u=root,p=root /usr/local/mysql/data/localhost-slow.log 
      -- 结果如下图：
      -- 总的查询时间
      -- 总的锁定时间
      -- 总的获取数据量
      -- 扫描的数据量
      -- 查询大小
      -- Response:总的查询时间；time：该查询在本次分析中总的时间占比；calls：执行次数，即本次分析总共有多少条这种类型的查询语句；R/Call：平均每次执行的响应时间；Item：查询对象
      ```

      ![pt-query-digest分析](F:\mysql\images\pt-query-digest分析.png)

    - 

- 