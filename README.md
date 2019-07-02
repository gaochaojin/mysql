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


##### 索引：索引是数据结构

- ##### 索引是什么

  - ##### 二分查找：折半查找法，用来查找一组有序的记录数组中的耨已记录

    将记录按有序化（递增或递减）排列，在查找过程中采用跳跃式方式查找，即先以有序数列的中点位置作为比较位置，如果要找的元素小于该中点元素，则将待查序列所需为左半部分，否则为右半部分，通过一次比较，将查找区间缩小一半。

    ```markdown
    # 实例，升序，查找数字48
    数据：5， 10， 19， 21， 31， 37， 42， 48， 50， 52 
    下标：0， 1，  2，  3，  4，  5，  6，  7，  8，  9 
    
    步骤一：设置数据下标，设low为下标最小值0，high为下标最大值9
    步骤二：通过low和high得到mid，mid=（low+high）/2
    步骤三：mid=4对应的数据为31，31<48
    步骤四：通过二分查找，将low设置为31对应下标为4，此时mid=6
    步骤五：mid=6对应的数据为42，42<48
    步骤六：通过二分查找，将low设置为42对应下标为6，此时mid=7
    步骤七：mid=7对应的数据为48，48==48，查找结束
    # 通过3次二分查找，找到了所要查找的数字，而顺序查找需要8次
    ```

  - ##### 二叉树（Binary Tree）

    每个节点至多只有二棵子树；

    二叉树的子树有左右之分，次序不能颠倒；

    一棵深度为k，且有2^k -1节点，称为满二叉树（Full Tree）；

    一棵深度为k，且root到k-1层的节点树都达到最大，第k层的所有节点都连续集中在最左边，此时为完全二叉树（Complete Tree）

    - 平衡二叉树（AVL-树）

      - 左子树和右子树都是平衡二叉树；

      - 左子树和右子树的高度差绝对值不超过1；

        ![img](./images/平衡二叉树.png)

    - 平衡二叉树的遍历

      - 前序：6，3，2，5，7，8（Root节点在开头，中-左-右顺序）
      - 中序：2，3，5，6，7，8（中序遍历即为升序，左-中-右顺序）
      - 后序：2，5，3，8，7，6（Root节点在结尾，左-右-中顺序）

    - 平衡二叉树的旋转

      需要通过旋转（左旋，右旋）来维护平衡二叉树的平衡，在添加和删除时需要有额外的开销

      ![img](./images/平衡二叉树旋转.jpg)

  - ##### B+树（在数据结构与算法中）

- ##### 索引的分类

  - ##### 普通索引：即一个索引只包含单个列，一个表可以有多个单列索引

  - ##### 唯一索引：索引列的值必须唯一，但允许有空值

  - ##### 复合索引：即一个索引包含多个列

  - ##### 聚簇索引：并不是一种单独的索引类型，而是一种数据存储方式。具体细节取决于不同的实现，InnoDB的聚簇索引其实就是在同一个结构中保存了B-Tree索引和数据行

  - ##### 非聚簇索引：不是聚簇索引，就是非聚簇索引

- ##### 基础语法

  ```mysql
  -- 查看索引
  show index from table_name;
  -- 创建索引
  create [unique] index index_name on table_name(column_name(length));
  alter table table_name add [unique] index [index_name] (column_name);
  -- 删除索引
  drop index [index_name] on table_name;
  ```

##### 执行计划

- ##### 定义：使用explain关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句的，分析你的查询语句或是表结构的性能瓶颈。

- ##### 作用

  - 表的读取顺序
  - 数据读取操作的操作类型
  - 哪些索引可以使用
  - 哪些索引被实际使用
  - 表之间的引用
  - 每张表有多少行被优化器查询

- ##### 语法

  ```mysql
  explain select * from account where id = 1;
  ```

- ##### 执行计划详解（通过执行计划来看sql，而不是通过sql来猜测执行计划）

  ```mysql
  -- 实例sql
  DROP TABLE IF EXISTS `t1`;
  CREATE TABLE `t1` (
    `id` int(11) NOT NULL,
    `other_column` char(20) DEFAULT NULL,
    `col1` char(20) DEFAULT NULL,
    `col2` varchar(20) DEFAULT NULL,
    `col3` varchar(20) DEFAULT NULL,
    PRIMARY KEY (`id`),
    KEY `idx_col1_col2_col3` (`col1`,`col2`,`col3`) USING HASH
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
  
  -- ----------------------------
  -- Records of t1
  -- ----------------------------
  INSERT INTO `t1` VALUES ('1', 'asd', 'ac', 'ac', 'ac');
  INSERT INTO `t1` VALUES ('2', '2', 'ac', 's', 'a');
  INSERT INTO `t1` VALUES ('3', 'ac', 'sc', 'a', 'a');
  
  -- ----------------------------
  -- Table structure for t2
  -- ----------------------------
  DROP TABLE IF EXISTS `t2`;
  CREATE TABLE `t2` (
    `id` int(11) NOT NULL,
    `other_column` char(20) DEFAULT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
  
  -- ----------------------------
  -- Records of t2
  -- ----------------------------
  INSERT INTO `t2` VALUES ('1', '');
  
  -- ----------------------------
  -- Table structure for t3
  -- ----------------------------
  DROP TABLE IF EXISTS `t3`;
  CREATE TABLE `t3` (
    `id` int(11) NOT NULL,
    `other_column` char(20) DEFAULT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
  
  -- ----------------------------
  -- Records of t3
  -- ----------------------------
  INSERT INTO `t3` VALUES ('1', '');
  ```

  

  ```markdown
  # 包含如下几列
  | id | select_type | table   | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra 
  ```

  - ##### ID列：描述select查询的序列号，包含一组数字，表示查询中执行select字句或操作表的顺序；根据ID的数值可以分为以下三种情况：

    - id相同：执行顺序由上而下

      ```mysql
      EXPLAIN SELECT
      	t2.*
      FROM
      	t1,
      	t2,
      	t3
      WHERE
      	t1.id = t2.id
      AND t1.id = t3.id;
      ```

    - id不同：如果是子查询，id序号会递增，id值越大优先级越高，越先被执行

      ```mysql
      EXPLAIN
      select * from t2 where id = (
      select id from t1 where id = (select id from t3 where other_column = ''));
      ```

    - id相同不同：同时存在

      ```mysql
      EXPLAIN
      select t2.* from (
       select id from t1 where id =  (select t3.id from t3 where t3.other_column='')
      ) s1 ,t2 where s1.id = t2.id;
      ```

      

  - ##### select_type列：查询类型，用于区分普通查询/联合查询/子查询等的复合查询

    | 类型         | 描述                                                         |
    | ------------ | ------------------------------------------------------------ |
    | SIMPLE       | 简单的select查询，查询中不包含子查询或者UNION                |
    | PRIMARY      | 查询中若包含任何复杂的字部份，最外层查询则被标记             |
    | SUBQUERY     | 在select或where列表中包含子查询                              |
    | DERIVED      | 在from列表中包含的子查询被标记为DERIVED（衍生）。MySQL会递归执行这些子查询，把结果放在临时表里。 |
    | UNION        | 若第二个select出现在UNION之后，则被标记为UNION；若UNION包含在from子句的子查询中，外层select将被标记为DERIVED |
    | UNION RESULT | 从UNION表获取结果的select                                    |

    - SIMPLE

      ```mysql
      explain select * from t1;
      ```

    - PRIMARY与SUBQUERY

      ```mysql
      explain select t1.*,(select t2.id from t2 where t2.id = 1) from t1;
      ```

    - DERIVED

      ```mysql
      -- 在mysql5.6中t2显示为derived；但是在mysql5.7中显示为simple 这个和数据库版本和数据量有关
      EXPLAIN select t1.* from t1 ,(select t2.* from t2 where t2.id = 1 ) s2  where t1.id = s2.id;
      ```

    - UNION与UNION RESULT

      ```mysql
      explain 
      select * from t2 
      UNION
      select * from t2;
      ```

  - ##### table列：显示这一行的数据是关于哪张表的

  - ##### Type列：访问类型，是较为重要的一个指标，结果值从最好到最坏依次是：

    system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

    ##### 常用顺序：system > const > eq_ref > ref > range > index > ALL；一般来说，得保证查询至少达到range级别，最好能达到ref

    - ##### System与const

      System：表只有一行记录（等于系统表），这是const类型的特列；

      Const：表示通过索引一次就找到了；Const用于比较primary key或者unique索引，因为只匹配一行数据，所以很快

      ```mysql
      EXPLAIN
      select * from t2 where id = 1;-- id为primary key
      ```

    - ##### eq_ref：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描

      ```mysql
      EXPLAIN
       SELECT * from t1,t2 where t1.id = t2.id；-- t1:eq_ref
      ```

    - ##### Ref：非唯一性索引扫描，返回匹配某个单独值的所有行；本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以它应该属于查找和扫描的混合体

      ```mysql
      EXPLAIN
      select count(DISTINCT col1) from t1 where col1 = 'ac';
      ```

    - ##### range：只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引，一般就是where语句中出现between、<、<、in等的查询这种范围扫描索引扫描比全表扫描要好，因为它只要开始于索引的某一点，而结束于另一个点，不用扫描全部索引

      ```mysql
      EXPLAIN select * from t1 where id BETWEEN 30 and 60;
      
      EXPLAIN select * from t1 where id in(1,2);
      ```

    - ##### Index：当查询的结果全为索引列的时候，虽然也是全部扫描，但是只查询的索引库，而没有区查询数据

      ```mysql
      EXPLAIN select c2 from testdemo;
      ```

    - ##### ALL：将遍历全表以找到匹配的行

      ```mysql
      EXPLAIN select * from t1;
      ```

  - ##### possible_keys与key

    possible_keys：可能使用的key

    key：实际使用的索引，如果为NULL，则没有使用索引

    查询中若使用了覆盖索引，则该索引和查询的select字段重叠

    ```mysql
    EXPLAIN select col1,col2 from t1;
    ```

  - ##### key_len： 表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好；显示的值为索引字段的最大可能长度，并非实际使用长度，即ken_len是根据表定义计算而得，不是通过表内检索出的

    - ken_len表示索引使用的字节数

    - 根据这个值，就可以判断索引使用情况，特别是在组合索引时，判断所有的索引字段是否都i被查询用到

    - char和varchar跟字符编码也有密切的联系

    - latin1占1个字节，gbk占用2个字节，utf8占用3个字节

      ##### 字符类型：真正建立所有的类型常用情况只是char、varchar

      - 字符类型-索引字段为char类型+不可为Null时

        ```mysql
        CREATE TABLE `s1` (
          `id` int(11) NOT NULL AUTO_INCREMENT,
          `name` char(10) NOT NULL,
          `addr` varchar(20) DEFAULT NULL,
          PRIMARY KEY (`id`),
          KEY `name` (`name`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
         
         -- ken_len=3*10+0=30
        explain select * from s1 where name='james';
        ```

      - 字符类型-索引字段为char类型+允许为Null时

        ```mysql
        CREATE TABLE `s2` (
          `id` int(11) NOT NULL AUTO_INCREMENT,
          `name` char(10) DEFAULT NULL,
          `addr` varchar(20) DEFAULT NULL,
          PRIMARY KEY (`id`),
          KEY `name` (`name`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
         
         -- ken_len=3*10+1=31
        explain select * from s2 where name='james';
        ```

      - 字符索引-索引字段为varchar类型+不可为Null时

        ```mysql
        CREATE TABLE `s3` (
          `id` int(11) NOT NULL AUTO_INCREMENT,
          `name` varchar(10) NOT NULL,
          `addr` varchar(20) DEFAULT NULL,
          PRIMARY KEY (`id`),
          KEY `name` (`name`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
         
         -- ken_len=3*10+2=32  varchar类型要加2
        explain select * from s3 where name='james';
        ```

      - 字符索引-索引字段为varchar类型+允许为Null时

        ```mysql
        CREATE TABLE `s4` (
          `id` int(11) NOT NULL AUTO_INCREMENT,
          `name` varchar(10) DEFAULT NULL,
          `addr` varchar(20) DEFAULT NULL,
          PRIMARY KEY (`id`),
          KEY `name` (`name`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
         
         -- key_len=3*10+1+2=33
        explain select * from s3 where name='james';
        ```

      ##### 数字类型

      ```mysql
      CREATE TABLE `numberKeyLen ` (
      `c0`  int(255) NOT NULL ,
      `c1`  tinyint(255) NULL DEFAULT NULL ,
      `c2`  smallint(255) NULL DEFAULT NULL ,
      `c3`  mediumint(255) NULL DEFAULT NULL ,
      `c4`  int(255) NULL DEFAULT NULL ,
      `c5`  bigint(255) NULL DEFAULT NULL ,
      `c6`  float(255,0) NULL DEFAULT NULL ,
      `c7`  double(255,0) NULL DEFAULT NULL ,
      PRIMARY KEY (`c0`),
      INDEX `index_tinyint` (`c1`) USING BTREE ,
      INDEX `index_smallint` (`c2`) USING BTREE ,
      INDEX `index_mediumint` (`c3`) USING BTREE ,
      INDEX `index_int` (`c4`) USING BTREE ,
      INDEX `index_bigint` (`c5`) USING BTREE ,
      INDEX `index_float` (`c6`) USING BTREE ,
      INDEX `index_double` (`c7`) USING BTREE 
      )
      ENGINE=InnoDB
      DEFAULT CHARACTER SET=utf8 COLLATE=utf8_general_ci
      ROW_FORMAT=COMPACT;
      
      -- key_len = 数字类型所占字节数+Null（1）
      EXPLAIN
      select * from  numberKeyLen where c1=1;-- key_len=1+1=2
      
      EXPLAIN
      select * from  numberKeyLen where c2=1;-- key_len=2+1=3
      
      EXPLAIN
      select * from  numberKeyLen where c3=1;-- key_len=3+1=4
      
      EXPLAIN
      select * from  numberKeyLen where c4=1;-- key_len=4+1=5
      
      EXPLAIN
      select * from  numberKeyLen where c5=1;-- key_len=8+1=9
      
      EXPLAIN
      select * from  numberKeyLen where c6=1;-- key_len=4+1=5
      
      EXPLAIN
      select * from  numberKeyLen where c7=1;-- key_len=8+1=9
      ```

      ##### 日期和时间
    
      ```mysql
      CREATE TABLE `datatimekeylen ` (
      `c1`  date NULL DEFAULT NULL ,
      `c2`  time NULL DEFAULT NULL ,
      `c3`  year NULL DEFAULT NULL ,
      `c4`  datetime NULL DEFAULT NULL ,
      `c5`  timestamp NULL DEFAULT NULL ,
      INDEX `index_date` (`c1`) USING BTREE ,
      INDEX `index_time` (`c2`) USING BTREE ,
      INDEX `index_year` (`c3`) USING BTREE ,
      INDEX `index_datetime` (`c4`) USING BTREE ,
      INDEX `index_timestamp` (`c5`) USING BTREE 
      )
      ENGINE=InnoDB
      DEFAULT CHARACTER SET=utf8 COLLATE=utf8_general_ci
      ROW_FORMAT=COMPACT;
      
      -- key_len = 日期和时间类型所占字节数+Null（1）
      EXPLAIN
      SELECT  * from datatimekeylen where c1 = 1;-- key_len=3+1=4
      
      EXPLAIN
      SELECT  * from datatimekeylen where c2 = 1;-- key_len=3+1=4
      
      EXPLAIN
      SELECT  * from datatimekeylen where c3 = 1;-- key_len=1+1=2
      
      EXPLAIN
      SELECT  * from datatimekeylen where c4 = 1;-- key_len=5+1=6
      
      EXPLAIN
      SELECT  * from datatimekeylen where c5 = 1;-- key_len=4+1=5
      -- datetime类型在5.6中字段长度为5个字节
      -- datetime类型在5.5中字段长度为8个字节
      ```
  
  - ##### Ref：显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列和常量被用于查找索引列上的值
  
    ```mysql
    EXPLAIN
    select * from s1 ,s2 where s1.id = s2.id and s1.name = 'enjoy';
    ```
  
  - ##### Rows：根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数
  
  - ##### Extra：包含不适合在其他类中显示但十分重要的额外信息
  
    - ##### Using filesort：说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。MySQL中无法利用索引完成的排序操作称为“文件排序”；当发现有Using filesort后，实际上就是发现了可以优化的地方
  
      ```mysql
      EXPLAIN select col1 from t1 where col1='ac' order by col3;
      
      -- 当排序增加了col2之后，就没有了
      EXPLAIN select col1 from t1 where col1='ac' order by col2,col3;
      ```
  
    - ##### Using temporary：使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常用于排序order by和分组查询group by
  
      ```mysql
      EXPLAIN select col1 from t1 where col1 in('ac','ab','aa') GROUP BY col2;
      -- Using where; Using index; Using temporary; Using filesort
      
      EXPLAIN select col1 from t1 where col1 in('ac','ab','aa') GROUP BY col1,col2;
      -- Using where; Using index 
      -- 在执行计划里面有using filesort而且还有Using temporary的时候，特别需要注意
      ```
  
    - ##### Using index：表示相应的select操作中使用了覆盖索引（Covering Index），避免访问了表的数据行，效率不错
  
      ```mysql
      -- 如果同时出现using where，表明索引被用来执行索引键值的查找
      EXPLAIN select col2 from t1 where col1 = 'ab';
      -- 如果没有同时出现using where，表明索引用来读取数据而非执行查找动作
      EXPLAIN select col2 from t1;
      ```
  
      ##### 覆盖索引：有两种理解方式
  
      - 就是select的数据列只用从索引中就能够取得，不必读取数据行，MySQL可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件，即查询列要被所建的索引覆盖
  
      - 索引是高效找到行的一个方法，但是一般数据库也能使用索引找到一个列的数据，因此它不必读取整个行。毕竟索引叶子节点存储了他们索引的数据，即可以通过读取索引得到想要的数据，就不需要读取行了。一个索引包含了满足查询结果的数据就是覆盖索引
  
        **注意：**如果使用覆盖索引，要注意select列表中只取出需要的列，不可select *，因为如果将所有字段一起做索引会导致索引文件过大，查询性能下降；
  
    - ##### Using where：使用了where过滤
  
    - ##### Using join buffer：使用了连接缓存
  
      ```mysql
      EXPLAIN select * from t1  JOIN t2  on t1.other_column = t2.other_column;
      
      -- 查看缓存大小
      show variables like '%join_buffer_size%';
      ```
  
    - ##### Impossible where：where字句的值总是false，不能用来获取任何元组
  
      ```mysql
      EXPLAIN select * from t1 where 1=2
      
      EXPLAIN select * from t1 where  t1.other_column ='kobe' and t1.other_column = 'james'
      ```
  
  ##### SQL优化
  
  - ##### 优化实战
  
    ```mysql
    -- 实例表
    CREATE TABLE `staffs`(
    	id int primary key auto_increment,
    	name varchar(24) not null default "" comment'姓名',
    	age int not null default 0 comment '年龄',
    	pos varchar(20) not null default ""  comment'职位',
    	add_time timestamp not null default current_timestamp comment '入职时间'
    	)charset utf8 comment '员工记录表';
     
    	
    insert into staffs(name,age,pos,add_time) values('z3',22,'manage',now());
    insert into staffs(name,age,pos,add_time) values('july',23,'dev',now());
    insert into staffs(name,age,pos,add_time) values('2000',23,'dev',now());
     
    alter table staffs add index idx_staffs_nameAgePos(name,age,pos);
    ```
  
    - ##### 尽量全值匹配
  
      ```mysql
      -- 当建立了索引列后，能在where条件中使用索引的尽量所用。
      EXPLAIN SELECT * FROM staffs WHERE NAME = 'July';
      EXPLAIN SELECT * FROM staffs WHERE NAME = 'July' AND age = 25;
      EXPLAIN SELECT * FROM staffs WHERE NAME = 'July' AND age = 25 AND pos = 'dev';
      ```
  
    - ##### 最佳左前缀法则
  
      ```mysql
      -- 查询从索引的最左前列开始并且不跳过索引中的列
      EXPLAIN SELECT * FROM staffs WHERE  age = 25 AND pos = 'dev';-- 全表扫描
      EXPLAIN SELECT * FROM staffs WHERE pos = 'dev';-- 全表扫描
      EXPLAIN SELECT * FROM staffs WHERE NAME = 'July';
      ```
  
    - ##### 不在索引列上做任何操作
  
      ```mysql
      -- 不在索引列上做任何操作（计算、函数、（自动或手动）类型转换），会导致索引失效而转向全表扫描
      EXPLAIN SELECT * FROM staffs WHERE NAME = 'July';
      
      EXPLAIN SELECT * FROM staffs WHERE left(NAME,4) = 'July';-- 全表扫描
      ```
  
    - ##### 范围查找放最后
  
      ```mysql
      -- 中间有范围查询会导致后面的索引列全部失效,故需要将范围查找放最后（不是sql中放到最后，而是将范围的该字段索引顺序放到最后）
      EXPLAIN SELECT * FROM staffs WHERE NAME = 'July'  and age >22 and pos='manager';
      ```
  
    - ##### 覆盖索引尽量用
  
      ```mysql
      -- 尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），减少select *
      EXPLAIN SELECT * FROM staffs WHERE NAME = 'July'  and age =22 and pos='manager';
      
      EXPLAIN SELECT name,age,pos FROM staffs WHERE NAME = 'July'  and age =22 and pos='manager';
      
      EXPLAIN SELECT * FROM staffs WHERE NAME = 'July'  and age >22 and pos='manager';
      
      EXPLAIN SELECT name,age,pos FROM staffs WHERE NAME = 'July'  and age >22 and pos='manager';
      ```
  
    - ##### “不等于”要慎用
  
      ```mysql
      -- mysql在使用不等于时，无法使用索引，会导致全表扫描
      EXPLAIN SELECT * FROM staffs WHERE NAME = 'July';
      
      EXPLAIN SELECT * FROM staffs WHERE NAME != 'July';
      
      EXPLAIN SELECT * FROM staffs WHERE NAME <> 'July';
      -- 如果必须使用不等于，用覆盖索引
      EXPLAIN SELECT name,age,pos FROM staffs WHERE NAME != 'July';
      
      EXPLAIN SELECT name,age,pos FROM staffs WHERE NAME <> 'July';
      ```
  
    - ##### Null/Not有影响
  
      ```mysql
      -- 自定义为Not Null
      EXPLAIN select * from staffs where name is null;
      
      EXPLAIN select * from staffs where name is not null;
      -- 在字段为not null的情况下，使用is null或is not null会导致索引失效
      -- 解决方式：覆盖索引
      EXPLAIN select name,age,pos from staffs where name is not null;
      
      -- 自定义为Null或者不定义
      -- 在字段为null或者不定义的情况下，使用is not null会导致索引失效
      -- 解决方式：覆盖索引
      ```
  
    - ##### Like查询要当心
  
      ```mysql
      -- like以通配符开头（'%abc...'），mysql索引失效会变成全表扫描的操作
      EXPLAIN select * from staffs where name ='july';
      
      EXPLAIN select * from staffs where name like '%july%';
      
      EXPLAIN select * from staffs where name like '%july';
      
      EXPLAIN select * from staffs where name like 'july%';-- 效率较高
      -- 解决方式：覆盖索引
      EXPLAIN select name,age,pos from staffs where name like '%july%';
      ```
  
    - ##### 字符类型加引号
  
      ```mysql
      -- 字符串不加单引号索引失效
      EXPLAIN select * from staffs where name = 917;
      -- 解决方式：加单引号
      EXPLAIN select * from staffs where name = '917'；
      ```
  
    - ##### OR改UNION效率高
  
      ```mysql
      EXPLAIN
      select * from staffs where name='July' or name = 'z3';
      
      -- 使用union代替or
      EXPLAIN
      select * from staffs where name='July' 
      UNION
      select * from staffs where  name = 'z3';
      -- 解决方式：覆盖索引
      EXPLAIN select name,age from staffs where name='July' or name = 'z3';
      ```
  
  - ##### 批量导入
  
    - ##### insert语句优化
  
      - 提交前关闭自动提交
      - 尽量使用批量insert语句
      - 可以使用MyISAM存储引擎
  
    - ##### LOAD DATA INFLIE
  
      ```mysql
      -- 使用load data infile比一般的insert语句快20倍
      -- 将表product_info中的数据导出到文件product.txt中
      select * into OUTFILE 'D:\\product.txt' from product_info;
      
      -- 将文件product.txt中的数据导入到表product_info中
      load data INFILE 'D:\\product.txt' into table product_info;
      
      -- 查看变量secure_file_priv
      show VARIABLES like 'secure_file_priv';
      
      -- secure_file_priv 为 NULL 时，表示限制mysqld不允许导入或导出。
      
      -- secure_file_priv 为 /tmp 时，表示限制mysqld只能在/tmp目录中执行导入导出，其他目录不能执行。
      
      -- secure_file_priv 没有值时，表示不限制mysqld在任意目录的导入导出。
      ```
  
      

