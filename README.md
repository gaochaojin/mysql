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