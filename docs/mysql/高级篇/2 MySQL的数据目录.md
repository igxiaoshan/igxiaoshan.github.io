# MySQL数据目录

## MySQL8.0的主要目录结构

```sh
## 使用命令,查看mysql的安装目录
find / -name mysql
```

### 数据库文件的存放路径

> MySQL数据库文件的存放路径: `/var/lib/mysql`

```sql
-- 也可以使用mysql服务命令查看数据目录
SHOW BARIABLES LIKE 'datadir';
```

### 相关命令目录

> 相关命令目录: `/usr/bin (mysqladmin, mysqlbinlog, mysqldump 等命令) 和 /usr/bin`

```sh
## 进入到bin目录下,查找mysql服务相关的文件
cd /usr/bin
find . -name "mysql*"
```

### 配置文件目录

> 配置文件目录: `/usr/share/mysql-8.0(命令及配置文件), /etc/my.cnf (配置文件)` 

![](E:\igsshan_learn\mysql\images\share-mysql8.png)

## 数据库和文件系统的关系

### 查看默认数据库

```SQL
-- 查看mysql服务数据库
SHOW DATABASES;
```

可以看到有4个数据库是属于MySQL自带的系统数据库

- `mysql`

  - mysql系统自带的核心数据库,存储mysql的用户账户和权限信息,一些存储过程,事件的定义信息,一些运行过程中产生的日志信息,一些帮助信息以及时区信息等

- `information_schema`

  - mysql自带的数据库,保存着mysql服务器`维护的所有其他数据库的信息`,比如有哪些表,哪些视图,哪些触发器,哪些列,哪些索引.这些信息并不是真实的用户数据,而是一些描述性信息,有时候也称之为`元数据`.在系统数据库`information_schema`中提供了一些以`innodb_sys`开头的表,用于表示内部系统表

    ```sql
    USE information_schema;
    
    SHOW TABLES LIKE 'innodb_sys%';
    ```

- `performance_schema`

  - mysql自带的数据库,这个数据库里主要保存mysql服务运行过程中的一些状态信息,可以用来`监控 MySQL 服务的各类性能指标`.包括统计最近执行了哪些语句,在执行过程的每个阶段都花费了多长时间,内存使用情况等等

- `sys`

  - mysql自带的数据库,这个数据库主要是通过`视图`的形式把`information_schema`和`performance_schema`结合起来,帮助系统管理员和开发人员监控MySQL的技术性能

### 数据库在文件系统中的表示

```sh
[root@CentOS7_002 mysql]# cd /var/lib/mysql
[root@CentOS7_002 mysql]# ll
总用量 188868
-rw-r-----. 1 mysql mysql       56 3月  22 22:12 auto.cnf
-rw-r-----. 1 mysql mysql     2120 3月  23 13:48 binlog.000001
-rw-r-----. 1 mysql mysql       16 3月  22 22:13 binlog.index
-rw-------. 1 mysql mysql     1680 3月  22 22:12 ca-key.pem
-rw-r--r--. 1 mysql mysql     1112 3月  22 22:12 ca.pem
-rw-r--r--. 1 mysql mysql     1112 3月  22 22:12 client-cert.pem
-rw-------. 1 mysql mysql     1676 3月  22 22:12 client-key.pem
drwxr-x---. 2 mysql mysql       38 3月  23 13:48 dbtest1
-rw-r-----. 1 mysql mysql   196608 3月  23 13:50 #ib_16384_0.dblwr
-rw-r-----. 1 mysql mysql  8585216 3月  22 22:12 #ib_16384_1.dblwr
-rw-r-----. 1 mysql mysql     5556 3月  22 22:12 ib_buffer_pool
-rw-r-----. 1 mysql mysql 12582912 3月  23 13:48 ibdata1
-rw-r-----. 1 mysql mysql 50331648 3月  23 13:50 ib_logfile0
-rw-r-----. 1 mysql mysql 50331648 3月  22 22:12 ib_logfile1
-rw-r-----. 1 mysql mysql 12582912 3月  22 22:13 ibtmp1
drwxr-x---. 2 mysql mysql      187 3月  22 22:13 #innodb_temp
drwxr-x---. 2 mysql mysql      143 3月  22 22:12 mysql
-rw-r-----. 1 mysql mysql 25165824 3月  23 13:48 mysql.ibd
srwxrwxrwx. 1 mysql mysql        0 3月  22 22:13 mysql.sock
-rw-------. 1 mysql mysql        5 3月  22 22:13 mysql.sock.lock
drwxr-x---. 2 mysql mysql     8192 3月  22 22:12 performance_schema
-rw-------. 1 mysql mysql     1676 3月  22 22:12 private_key.pem
-rw-r--r--. 1 mysql mysql      452 3月  22 22:12 public_key.pem
-rw-r--r--. 1 mysql mysql     1112 3月  22 22:12 server-cert.pem
-rw-------. 1 mysql mysql     1676 3月  22 22:12 server-key.pem
drwxr-x---. 2 mysql mysql       28 3月  22 22:12 sys
-rw-r-----. 1 mysql mysql 16777216 3月  23 13:50 undo_001
-rw-r-----. 1 mysql mysql 16777216 3月  23 13:48 undo_002

```

> 这个数据目录下的文件是子目录比较多,除了`information_schema`这个系统数据外,其他的数据库在`数据目录`下都有对应的子目录

以`dbtest1`数据库为例,进入到子目录下

- MySQL5.7

  ```sh
  [root@CentOS7_003 ~]# cd /var/lib/mysql/dbtest1
  [root@CentOS7_003 dbtest1]# ll
  总用量 112
  -rw-r-----. 1 mysql mysql    61 3月  22 16:33 db.opt
  -rw-r-----. 1 mysql mysql  8590 3月  22 16:37 tmp1.frm
  -rw-r-----. 1 mysql mysql 98304 3月  22 16:37 tmp1.ibd
  
  ```

- MySQL8.0

  ```sh
  [root@CentOS7_002 dbtest1]# cd /var/lib/mysql/dbtest1/
  [root@CentOS7_002 dbtest1]# ll
  总用量 160
  -rw-r-----. 1 mysql mysql 114688 3月  23 13:47 tmp1.ibd
  
  ```

### 表在文件系统中的表示

#### InnoDB存储引擎模式

##### 表结构

> 为了保存表结构,`InnoDB`在`数据目录`下对应的数据库子目录下创建了一个专门用户`描述表结构的文件`文件名叫 `表名.frm`

```txt
我们创建一个`dbtest1`数据库,在数据库下创建一个名为`tmp1`的表;
那在数据库`dbtest1`对应的子目录下就会创建一个名为`tmp1.frm`的用于描述表结构的文件
`.frm`文件的格式在不同的平台上都是相同的,
这个后缀名为`.frm`是以`二进制格式`存储的,如果直接打开是乱码的
```

##### 表中数据和索引

> ##### 系统表空间(system tablespace)

```txt
默认情况下,InnoDB会在数据目录下创建一个名为`ibdata1`,大小为`12M`的文件,这个文件就是对应的`系统表空间`在文件系统上的表示
ps: 为什么才12M
注意这个文件是`自扩展文件`,当不够用的时候会自己增加文件大小

当然,如果你想让系统表空间对应问价系统上多个世纪文件,或者仅仅觉得原来的`ibdata1`这个文件名难听,那可以在MySQL启动时配置对应的文件路径以及它们的大小,比如修改一下`my.cnf`的配置文件

[server]
innodb_data_file_path = data1:512M;data2:512M:autoextend
```

> ##### 独立表空间(file-per-table tablespace)

```txt
在MySQL5.6.6以及之后的版本中,InnoDB并不会默认的把各个表的数据存储到系统表空间中,而是为`每一个表建立一个独立表空间`,也就是说我们创建了多少个表,就有多少个独立表空间.使用`独立表空间`来存储表数据的话,会在该表所属数据对应的子目录下创建一个表示该独立表空间的问价,文件名和表名相同,只不过添加了一个`.ibd`的扩展名而已,所以完整的文件名称长这样 `.ibd`

比如: 在使用了`独立表空间`去存储`dbtest1`数据库下的`test`表的话,那么在该表所在数据库对应的`dbtest1`目录下会为`test`表创建两个文件
"test.frm"
"test.ibd"
其中`test.ibd`文件就用来存储`test`表中的数据和索引
```

> ##### 系统表空间与独立表空间的设置

```txt
我们可以自己指定使用`系统表空间`还是`独立表空间`来存储数据,这个空能由启动参数`innodb_fire_per_table`控制,比如说我们想刻意将表数据都存储到`系统表空间`时,可以再启动MySQL服务器的时候这样配置
[server]
innodb_fire_per_table =0 # 0:代表使用系统表空间; 1:代表使用独立表空间
默认情况:
SHOW VARIABLES LIKE 'innodb_file_per_table';

+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_file_per_table | ON    |
+-----------------------+-------+
```

> ##### 其他类型的表空间

```txt
随着MySQL的发展,除了上述两种老牌表空间之外,还新提出了一些不同类型的表空间,比如通用表空间(general tablespace), 临时表空间(temporary tablespace) 等
```

#### MyISAM存储引擎模式

##### 表结构

> 在存储表结构方面 `MyISAM`和`InnoDB`一样,也是在`数据目录`下对应的数据库子目录下创建了一个专门用于描述表结构的文件 `表名.frm`

##### 表中数据和索引

> 在MyISAM中的索引全部都是`二级索引`,该存储引擎的`数据和索引是分开存放的`,所以在文件系统中也是使用不同的文件来存储数据文件和索引文件,同时表数据都存放在对应的数据库子目录下.假如`test`表使用了MyISAM存储引擎的话,那么在它所在的数据库对应的`dbtest1`目录下会为`test`表创建三个文件
>
> `test.frm`(存储表结构); `test.MYD`(存储数据 MYData); `test.MYI`(存储索引 MYIndex)

> #### 举例
>
> 创建一个`MyISAM`表,使用`ENGINE`选项显示指定存储引擎. 因为`InnoDB`是默认存储引擎

```sql
CREATE TABLE `student_myisam` (
	`id` int NOT NULL AUTO_INCREMENT,
    `name` varchar(255) DEFAULT NULL,
    `age` int DEFAULT NULL,
    `sex` varvhar(2) DEFAULT NULL,
    PRIMARY KEY (`id`)
)ENGINT = MYISAM AUTO_INCREMENT=0 DEFAULT CHARSET = uft8mb3;
```

#### 总结

> #### 举例: `数据库 a`,`表 b`

- 如果表b采用`InnoDB`,data\a中会产生1个或者2个文件

  - `b.frm`: 描述表结构文件,字段长度等
  - 如果采用`系统表空间`存储模式,数据信息和索引信息会存储在`ibdata1`中
  - 如果采用`独立表空间`存储模式, data\a 中会产生`b.ibd`文件( 存储数据信息和索引信息)

  > ### 此外:

  - MySQL5.7中会在 data\a 的目录下生成 `db.opt`文件用于保存数据库的相关配置, 比如:字符集,比较规则等,而 MySQL8.0不提供 db.opt文件
  - MySQL8.0中不再单独提供 b.frm , 而是合并在 b.ibd 文件中

- 如果表b采用`MyISAM`, data\a 中会产生3个文件

  - MySQL5.7中: `b.frm` 描述表结构文件,字段长度等

    MySQL8.0中: `b.xxx.sdi` 描述表结构文件,字段长度等

  - `b.MYD`(MYData): 数据信息文件,存储数据信息(如果采用独立表存储模式)

  - `b.MYI`(MYIndex): 存放索引信息文件





























































