# 用户与权限管理

## 用户管理

### 登录MySQL服务器

> 启动MySQL服务后,可以通过mysql命令来登录mysql服务器

```sh
mysql -u username -h hostname|hostIP -P port -p databaseName -e "sql语句"
```

- 参数解析
  - `-h 参数` 后面接主机名或者主机ip,hostname为主机, hostIP为主机IP
  - `-P 参数` 后面接MySQL服务的端口,通过该参数连接到指定的端口. MySQL服务的默认端口是3306,不使用该参数时自动连接到3306端口, port为连接的端口号
  - `-u 参数` 后面接用户名, username为用户名
  - `-p 参数` 用户密码
  - `DatabaseName 参数`指明登录到哪一个数据库中,如果没有该参数,就会直接登录到MySQL数据库中,然后使用use命令来选择数据库
  - `-e 参数` 后面可以直接加sql语句,登录到MySQL服务器以后即可执行这个sql语句,然后退出MySQL服务器

### 创建用户

```sql
-- CREATE USER 语句
CREATE USER 用户名 [IDENTIFIED BY '密码'][, 用户名 [IDENTIFIED BY '密码']];
```

- 用户名参数表示新建用户的账号,由`用户 (user) 和 主机名 (host)`构成;
- "[]"表示可选,也就是说,可以指定用户登录时需要密码验证,也可以不指定密码验证,这样用户可以直接登录;不过不推荐,不安全;如果指定密码,需要使用`IDENTIFIED BY`指定明文密码值,MySQL服务会对其进行加密保存
- `CREATE USER`语句可以同时创建多个用户

> ### 举例

```sql
-- 方式一
CREATE USER zhang3 IDENTIFIED BY 'abc123'; -- 如果没有指定host,默认是 %

-- 方式二
CREATE USER 'li4'@'localhost' IDENTIFIED BY 'abc123'; -- 指定host 
```

### 修改用户

```sql
-- 修改用户名
UPDATE mysql.user SET user='li4' WHERE user='wang5';
FLUSH PRIVILEGES; -- 修改后刷新
```

### 删除用户

- 方式一: 使用`DROP`方式删除(推荐)

  ```sql
  -- 使用 DROP USER 语句删除用户时,必须用于 DROP USER 权限
  DROP USER user[, user]...;
  
  -- 举例
  DROP USER li4; -- 默认删除host为%的用户
  DROP USER 'wang5'@'localhost';
  ```

- 方式二: 使用`DELETE`方式删除

  ```sql
  -- DELETE 语句格式
  DELETE FROM mysql.user WHERE host='hostname' AND user='username';
  
  -- 执行完删除后,需要使用 FLUSH 命令使用户生效
  FLUSH PRIVILEGES;
  
  -- 举例
  DELETE FROM mysql.user WHERE host='localhost' AND user='zhang3';
  FLUSH PRIVILEGES;
  ```

  > ### 注意: 不推荐使用`DELETE FROM USER u WHERE = 'li4'`进行删除,系统会有残留信息保留;而`DROP USER`命令会删除用户以及对应的权限,执行命令后会发现`mysql.user`表和`mysql.db`表的相应记录都消失了

### 设置当前用户密码

- `旧版本写法`

  ```sql
  -- 修改当前用户的密码: (mysql5.7版本测试有效)
  SET PASSWORD = PASSWORD('123456');
  ```

- #### `推荐写法`

  - 使用`ALTER USER`命令来修改当前用户密码

    ```sql
    ALTER USER USER() IDENTIFIED BY 'new_password';
    ```

  - 使用`SET`语句来修改当前用户密码

    ```sql
    SET PASSWORD = 'new_password'; -- 该语句自动将密码加密后赋给当前用户
    ```

### 修改其他用户密码

- 使用`ALTER`语句来修改普通用户的密码

  ```sql
  ALTER USER user [IDENTIFIED BY '新密码'][, user [IDENTIFIED BY '新密码']]...;
  ```

- 使用`SET`命令来修改普通用户的密码; 使用root用户登录到MySQL服务器后,使用SET语句修改普通用户的密码

  ```sql
  SET PASSWORD FOR 'username'@'hostname'='new_password';
  ```

- 使用`UPDATE`语句修改普通用户的密码 `(不推荐)`

  ```sql
  UPDATE mysql.user SET authentication_string=PASSWORD('123456')
  WHERE user='username' AND host='hostname';
  ```

### MySQL8密码管理(了解)

#### 密码过期策略

- 在MySQL中,数据库管理员可以`手动设置`账号密码过期,也可以建立一个`自动`密码过期策略
- 过期策略可以是`全局的`,也可以为`每个账号`设置单独的过期策略

```sql
ALTER USER user PASSWORD EXPIRE;

-- 示例
ALTER USER 'wang5'@'%' PASSWORD EXPIRE;

-- 方式一: 使用sql语句更改变量的值,并持久化
SET PERSIST default_password_lifetime = 180; -- 建立全局策略,设置密码每隔180天过期
-- 方式二: 配置文件my.cnf中进行维护
[mysqld]
default_password_lifetime=180 --建立全局策略,设置密码每隔180天过期

-- 手动设置指定时间过期: 单独设置
--- 每个账号既可使用全局过期策略,也可以单独设置策略; 在 CREATE USER 和 ALTER USER 语句上加入 PASSWORD EXPIRE 选项可实现单独设置策略

---- 设置账密90过期
CREATE USER 'zhang3'@'localhost' PASSWORD EXPIRE INTERVAL 90 DAY;
ALTER USER 'zhang3'@'localhost' PASSWORD EXPIRE INTERVAL 90 DAY;

---- 设置密码永不过期
CREATE USER 'wang5'@'localhost' PASSWORD EXPIRE NEVER;
ALTER USER 'wang5'@'localhost' PASSWORD EXPIRE NEVER;

---- 使用全局密码过期策略
CREATE USER 'li4'@'localhost' PASSWORD EXPIRE DEFAULT;
ALTER USER 'li4'@'localhost' PASSWORD EXPIRE DEFAULT;
```

#### 密码重用策略

- 手动设置密码重用方式1: 全局

  - 方式一

    ```sql
    SET PERSIST password_history = 6; 设置不能选择最近使用过的6个密码
    ```

  - 方式二: 配置文件

    ```cnf
    [mysqld]
    password_history=6  # 不能使用最近使用的6个密码
    password_reuse_interval=365 # 不能使用最近365天内的密码
    ```

- 手动设置密码重用方式2: 单独设置

  ```sql
  -- 不能使用最近5个密码
  CREATE USER 'zhao7'@'localhost' PASSWORD HISTORY 5;
  ALTER USER 'zhao7'@'localhsot' PASSWORD HISTORY 5;
  
  -- 不能使用最近365天内的密码：
  CREATE USER 'zhang3'@'localhost' PASSWORD REUSE INTERVAL 365 DAY;
  ALTER USER 'zhang3'@'localhost' PASSWORD REUSE INTERVAL 365 DAY;
  
  -- 既不能使用最近5个密码，也不能使用365天内的密码
  CREATE USER 'zhang3'@'localhost'
  PASSWORD HISTORY 5
  PASSWORD REUSE INTERVAL 365 DAY;
  
  ALTER USER 'zhang3'@'localhost'
  PASSWORD HISTORY 5
  PASSWORD REUSE INTERVAL 365 DAY;
  ```



## 权限管理

### 权限列表

```sql
-- 查看权限列表
SHOW PRIVILEGES;
```

- `CREATE和DROP权限`,可以创建新的数据库和表,或删除(移掉)已有的数据库和表,如果将MySQL数据库中的DROP权限授予某用户,用户就可以删除MySQL访问权限保存的数据库
- `SELECT, INSERT, UPDATE 和 DELETE 权限`允许在一个数据库现有的表上实施操作
- `SELECT权限`只有在它们真正从一个表中检索行时才被用到
- `INDEX权限`允许创建或删除索引;index适用于已有的表,如果具有某个表的CREATE权限,就可以在CREATE TABLE 语句中包括索引定义
- `ALTER权限`可以使用ALTER TABLE来更改表的结构和重命名表
- `CREATE ROUTINE权限`用来创建保存的程序(函数和程序),ALTER ROUTINE 权限用来更改和删除保存的程序
- `EXECUTE权限`用来执行保存的程序
- `GRANT 权限`允许授权给其他用户,可用于数据库,表和保存的程序
- `FILE权限`使用户可以使用 LOAD DATA INFILE 和 select ... into outfile 语句读或写服务器上的文件,任何被授予file权限的用户都能读或写MySQL服务器上的任何文件(说明用户可以读任何数据库目录下的文件,因为服务器可以访问这些文件)

### 授予权限的原则

- 能`满足需要的最小权限`;防止用户干坏事,比如用户只需要查询,只需要授予`select`权限即可,不要再给用户授予`update, insert或者delete`权限
- 创建用户的时候,`限制用户的登录主机`一般是限制成`指定的 ip 或者内网 ip段`
- 为每个用户`设置满足密码复杂度的密码`
- `定期清理不需要的用户`回收权限或者删除用户

### 授予权限

> 给用户授权的方式有2种,分别是通过`角色赋予用户给用户授权`和`直接给用户授权`.用户是数据库的使用者,我们可以通过给用户授予访问数据库中资源的权限,来控制使用者对数据库的访问,消除安全隐患啊

```sql
-- 授权命令
GRANT 权限1, 权限2, ...权限n  ON 数据库名称.表名称 TO 用户名@用户地址 [IDENTIFIED BY '密码'];
-- 该命令如果发现没有该用户,则会直接新建一个用户

-- 举例: 给li4用户本地命令行方式,授予dbtest1库下所有表的增删改查权限
GRANT SELECT,UPDATE,DELETE,INSERT ON dbtest1.* TO li4@localhost;

-- 举例: 授予通过网络方式登录的joe用户,对所有库所有表的全部权限,密码设为123;注意这里唯独不包括 grant的权限
GRANT ALL PRIVILEGES ON *.* TO joe@'%' IDENTIFIED BY '123';
```

> 在开发中,要根据用户的不同,对数据进行横向和纵向的分组
>
> - 所谓横向的分组,就是指用户可以接触到的数据的范围,比如可以查看哪些表的数据
> - 所谓纵向的分组,就是指用户对接触到的数据能访问到什么程度,比如能看,能改甚至是可以删除

### 查看权限

```sql
-- 查看当前用户权限
SHOW GRANTS;
或者
SHOW GRANTS FOR CURRENT_USER;
或者
SHOW GRANTS FOR CURRENT_USER();
```

```sql
-- 查看某用户的全局权限
SHOW GRANTS FOR 'user'@'主机地址';
```

### 回收权限

> 回收权限就是取消已经赋予用户的某些权限,`收回用户不必要的权限可以再一定程度上保证系统的安全性` MySQL中使用`revoke语句`取消用户的某些权限, 使用revoke回收权限之后,用户账户的记录将从`db, host, tables_priv和columns_priv表中删除`但是用户账户记录仍然在user表中保存(删除user表中的账户记录使用 `drop user`语句)
>
> 注意: 再将用户账户从user表删除之前,应该回收相应用户的所有权限

```sql
-- 回收权限
REVOKE 权限1, 权限2, ...权限n ON 数据库名称.表名称 FROM 用户名@用户地址;

-- 举例: 收回全库全表的所有权限
REVOKE ALL PRIVILEGES ON *.* FROM joe@'%';
--- 可能会报错:" you need (at least one of) the SYSTEM_USER privilege(s) for this operation" 需要给root授予 'system_uer'权限
GRANT system_user ON *.* TO 'root';

-- 举例: 收回mysql库下所有表的增删改查权限
REVOKE SELECT,INSERT,UPDATE,DELETE ON mysql.* FROM li4@localhost;
```

> ### 注意: 收回权限需用户重新登录后才能生效

## 权限表

### user表

> user表是mysql中最重要的一个权限表,`记录用户账户和权限信息`有50个字段

![](E:\igsshan_learn\mysql\images\mysql8-user-table.png)

> 这些字段可以分成4类,分别是范围列(或者用户列), 权限列, 安全列和资源控制列

#### 1.范围列

- host: 表示连接类型
  - `%`表示所有远程通过tcp方式的连接
  - `IP 地址`如(192.168.33.33,127.0.0.1)通过指定ip地址进行的tcp方式的连接
  - `机器名`通过指定网络中的机器名进行的tcp方式的连接
  - `::1`IPV6的本地ip地址,等同于IPV4的127.0.0.1
  - `localhost`本地方式通过命令行方式的连接,比如mysql -uxxx -p xxx方式的连接
- user: 表示用户名,同一用户通过不同方式连接的权限是不一样的
- password: 密码
  - 所有密码串通过password(明文字符串)生成的密文字符串.MySQL8.0在用户管理方面增加了角色管理,默认的密码加密方式也做了调整,由之前的`SHA1`改成了`SHA2`,不可逆,同时加上MySQL5.7的禁用用户和用户过期的功能,MySQL在用户管理方面的功能和安全性都较之前大大的增强了
  - mysql5.7及之后版本的密码保存到`authentication_string`字段中不再使用`password`字段

#### 2.权限列

- Grant_priv
  - 表示是否拥有GRANT权限
- Shutdown_priv
  - 表示是否拥有停止MySQL服务的权限
- Super_priv
  - 表示是否拥有超级权限
- Execute_priv
  - 表示是否拥有execute权限,拥有execute权限,可以执行存储过程和函数
- Select_priv,Insert_priv 等
  - 为该用户所拥有的权限

#### 3.安全列

> 安全列有6个字段,其中两个是ssl相关的(ssl_type,ssl_cipher)用于`加密`;两个是x509相关的(x509_issuer,x509_subject)用户`标识用户`,另外两个plugin字段用于`验证用户身份`的插件,该字段不能为空,如果该字段为空,服务器就是用内建授权验证机制验证用户身份

#### 4.资源控制列

>资源控制列的字段用来`限制用户使用的资源`分别为

- max_questions: 用户每小时允许执行查询操作次数
- max_updates: 用户每小时允许执行的更新操作次数
- max_connections: 用户每小时允许执行的连接操作次数
- max_user_connections: 用户允许同时建立的连接次数

### db表

```sql
-- 查看db表的基本结构
DESCRIBE mysl.db;
```

![](C:\Users\21926\Nutstore\1\igsshan_learn\mysql\images\mysql8-db-table.png)

#### 1.用户列

> db表用户列有3个字段,分别是Host,User,Db;这个三个字段分别表示主机名,用户名和数据库名;这三个字段组合构成了db表的主键

#### 2.权限列

> Create_routine_priv和Alter_routine_priv这两个字段决定用户是否具有创建和修改存储过程的权限

### tables_priv表和columns_priv表

> tables_priv表用来`对表设置操作权限`
>
> columns_priv表用来对表的`某一列设置权限`

```sql
-- 查看tables_priv表和columns_priv表的构造
DESCRIBE mysql.tables_priv
DESCREBE mysql.columns_priv

-- tables_priv 表有8个字段,分别是: Host, Db, User, Table_name, Grantor, Timestamp, Table_priv 和 Column_priv

-- columns_priv 表有7个字段,分别是: Host, Db, User, Table_name, Column_name, Timestamp, Column_priv
```

- `Host`,`Db`,`User`和`Table_name`四个字段表示主机名,数据库名,用户名和表名
- `Grantor`表示修改记录的用户
- `Timestamp`表示修改该记录的时间
- `Table_priv`表示对象的操作权限,包括 select, insert, update, delete, create, drop, grant, references, index 和 alter
- `Columns_priv`字段表示对表中的列的操作权限,包括 select, insert, update 和 references

### procs_priv表

> procs_priv 表可以对`存储过程和存储函数设置操作权限`

```sql
-- 查看procs_priv结构
DESC mysql.procs_priv;
```

![](C:\Users\21926\Nutstore\1\igsshan_learn\mysql\images\mysql8-procs_priv-table.png)

## 访问控制

### 连接核实阶段

> 当用户视图连接MySQL服务器时,服务器基于用户的身份以及用户是否能提供正确的密码验证身份来确定接收或者拒绝连接.即客户端用户会在连接请求中提供用户名,主机地址,用户密码,MySQL服务器接收到用户请求后,会`使用user表中的host,user和authentication_string这3个字段匹配客户端提供的信息`
>
> 服务器只有在user表记录的host和user字段匹配客户端主机名和用户名,并且提供正确的密码时才接受连接,`如果连接核实没有通过,服务器就完全拒绝访问;反之,服务器接收连接,然后进入阶段2等待用户请求`



#### 请求核实阶段

> 一旦建立了连接,服务器就进入了访问控制的阶段2,也就是请求核实阶段,对此连接上进来的每个请求,服务器检查该请求要执行什么操作,是否有足够的权限来执行它,这真是需要授权表中的权限列发挥作用的地方,这些权限可以来自于`user`,`db`,`tables_priv`,`column_priv`表

> 确认权限时,MySQL首先`检查user表`,如果指定的权限没有在user表中被授予,那么MySQL就会继续`检查db表`,db表是下一安全层级,其中的权限限定于数据库层级,在该层级的select权限允许用户查看指定数据库的所有表中的数据;如果在该层级没有找到限定的权限,则MySQL继续`检查tables_priv`以及`columns_priv`,如果所有权限表都检查完毕,但还是没有找到允许的权限操作,MySQL将`返回错误信息`,用户请求的操作不能执行,操作失败

```txt
用户向MySQL发出操作请求
	||
	||
	||
	\/
MySQL检查user权限表中的权限信息,匹配user,host字段值,查看请求的全局权限是否被允许,如果找到匹配结果,操作被允许执行,否则MySQL继续向下查找
	||
	||
	||
	\/
MySQL检查db权限表中的权限信息,匹配user,host字段值,查看请求的数据库级别的权限是否被允许,如果找到匹配结果,操作被允许执行,否则MySQL继续向下查找
	||
	||
	||
	\/
MySQL检查tables_priv权限表中的权限信息,匹配user,host字段值,查看请求的数据表级别的权限是否被允许,如果找到匹配结果,操作被允许执行,否则MySQL继续向下查找
	||
	||
	||
	\/
MySQL检查columns_priv权限表中的权限信息,匹配user,host字段值,查看请求的列级别的权限是否被允许,如果找到匹配结果,操作被允许执行,否则MySQL返回错误信息
```

> #### 注意: MySQL通过向下层级的顺序(从user表到columns_priv表)检查权限表,但并不是所有的权限都要执行该过程.例如;一个用户登录MySQL服务器之后执行对MySQL的管理操作,此时只涉及管理权限,因此MySQL只检查user表. 另外,如果请求的权限操作不被允许,MySQL也不会继续检查下一层级的表

## 角色管理

> 引入角色的目的`方便管理拥有相同权限的用户`,恰当的权限设定,可以确保数据的安全性,这是至关重要的

### 创建角色













































































































