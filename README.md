# mysql
mysql数据库的基本使用以及增删改查
# 一、数据库相关术语和概念

- **数据库**: 数据库是一些关联表的集合。

- **数据表**: 表是数据的矩阵。在一个数据库中的表看起来像一个简单的电子表格。

- **列**: 一列(数据元素) 包含了相同类型的数据, 例如邮政编码的数据。

- **行**: 一行（=元组，或记录）是一组相关的数据，例如一条用户订阅的数据。

- **冗余**: 存储两倍数据，冗余降低了性能，但提高了数据的安全性。

- **主键**: 主键是唯一的。一个数据表中只能包含一个主键。你可以使用主键来查询数据。

- **外键**: 外键用于关联两个表。

- **复合键**: 复合键（组合键）将多个列作为一个索引键，一般用于复合索引。

- **索引**: 使用索引可快速访问数据库表中的特定信息。索引是对数据库表中一列或多列的值进行排序

的一种结构。类似于书籍的目录。

- **参照完整性**: 参照的完整性要求关系中不允许引用不存在的实体。与实体完整性是关系模型必须满

足的完整性约束条件，目的是保证数据的一致性。

# 二、Linux数据库的开启和连接

## 1.安装数据库

基于Debian平台的Linux系统，可以直接使用apt命令安装mysql

```bash
sudo apt install -y mysql-server mysql-client 
```

## 2. 开启数据库服务

Ubuntu : 

```bash
service mysql start|stop|restart|status 
```

## 3. 连接数据库

各个 Linux 系统连接数据库的语法都一样

语法:  `mysql -hloaclhost -uroot -p123456 -P3306`

1. -h : host(ip地址) localhost = 127.0.0.1

2. -u : username(用户账户) 

3. -p : password(密码) 

4. -P : port(端口, 默认端口3306)

除了在centoOS7里面会生成一个临时密码，在其他的linux系统的root用户的默认密码都是为空，在centoOS&总我们可以通过以下命令查看临时密码

```bash
cat /var/log/mysqld.log |grep password
```

## 4. 设置root用户密码

数据库连接成功以后，因为此时使用的是root用户的临时密码，此时无法进行任何的操作，需要先修改

root用户的密码。

```mysql
alter user root@localhost identified with mysql_native_password by '你的密码';
```

**备注** 第一次使用 root 连接后最好添加一个新的用户来操作。出于安全考虑，日常开发中最好不要使用

root

```mysql
-- 创建新用户，并设置密码
-- *.* 代表该用户可以操作任何库、任何表
-- 主机名可以使用 '%', 代表允许该用户从任何机器登陆 
GRANT ALL PRIVILEGES on *.* to '用户名'@'localhost' IDENTIFIED BY "密码" WITH GRANT OPTION; -- 刷新使权限生效 flush privileges;
```

## 5. 退出数据库

四种方式效果一样: 

1. exit

2. quit

3. \q

4. 快捷键: ctrl + d

## 6. 密码忘记怎么办? 

1. 打开配置: vim /etc/my.cnf 

2. 添加这么一段:

如果文件中已存在 [mysqld] , 则直接将 skip-grant-tables 写到其下方即可。

3. 修改完成后，保存退出，重启服务: sudo systemctl restart mysql.service 

4. 使用命令 sudo mysql -uroot 重新连接MySql服务器，此时可以不使用密码直接登录用户。

5. 执行 update mysql.user set authentication_string=password('你的密码') where 

user="root"; 修改root用户的密码。

6. 执行 flush privileges 刷新策略，使策略立刻生效，并退出mysql客户端。

7. 修改 /etc/msyql/mysql.cnf 文件，注释掉第二步添加的两段内容。

8. 运行 sudo systemctl restart mysql.service 重启mysql服务器。

9. 此时可以使用新密码登录mysql服务器。

# 三、权限管理

## 1. MySQL 权限的两个阶段

1. 第一阶段为连接验证，主要限制用户连接 mysql-server 时使用的 ip及密码。
2. 第二阶段为操作检查，主要检查用户执行的指令是否被允许，一般非管理员账户不被允许执行
   drop、delete 等危险操作。

## 2.权限控制安全准则

1. 只授予能满足需要的最小权限，防止用户执行危险操作。
   -- 创建新用户，并设置密码 -- *.* 代表该用户可以操作任何库、任何表 -- 主机名可以使用 '%', 代表允许该用户从任何机器登陆 GRANT ALL PRIVILEGES on *.* to '用户名'@'localhost' IDENTIFIED BY "密码" WITH GRANT OPTION; -- 刷新使权限生效 flush privileges; [mysqld] skip-grant-tables
2. 限制用户的登录主机，防止不速之客登录数据库。
3. 禁止或删除没有密码的用户。
4. 禁止用户使用弱密码。
5. 定期清理无效的用户，回收权限或者删除用户。

## 3.常用操作

1. 创建账户、权限授予

   - 8.0 之前版本

     ```mysql
     GRANT ALL PRIVILEGES on *.* to '用户名'@'主机' IDENTIFIED BY "密码" WITH GRANT OPTION; flush privileges; -- 刷新使权限生效
     ```

     

     - `ALL PRIVILEGES` : 授予全部权限, 也可以指定
       `select` , `insert` , `update` , `delete` , `create` , `drop` , `index` , `alter` , `grant` , `references` , `reload` , `shutdown` , `process` , `file` 十四个权限。
     - `*.*` : 允许操作的数据库和表。
     - 主机名:表示允许用户从哪个主机登录， % 表示允许从任意主机登录。

   - WITH GRANT OPTION : 带有该子句说明允许用户将自己拥有的权限授予别人。

   - 8.0 之后的版本

     ```mysql
     CREATE USER `用户名`@`主机` IDENTIFIED BY '密码'; -- 创建账户 GRANT ALL ON *.* TO `用户名`@`主机` WITH GRANT OPTION; -- 授权
     ```

2. 修改密码

   ```mysql
   update user set authentication_string=password('你的密码') where user="root" 或者alter user '用户名'@'主机' identified with mysql_native_password by '你的密码';
   ```

3. 查看权限

   ```mysql
   update user set authentication_string=password('你的密码') where user="root" 或者alter user '用户名'@'主机' identified with mysql_native_password by '你的密码';
   ```

4. 回收权限

   ```mysql
   revoke all privileges on *.* from 'abc'@'localhost'; -- 回收用户 abc 的所有 权限
   revoke grant option on *.* from 'abc'@'localhost'; -- 回收权限的传递
   ```

5. 删除用户

   ```mysql
   use mysql; select host, user from user; drop user 用户名@'%';
   ```

> 了解:创建一个用户，允许该用户通过主机远程登录。

- 创建一个允许任意主机登录的账号。

  ```mysql
  GRANT ALL PRIVILEGES on *.* to 'zhangsan'@'%' IDENTIFIED BY "密码" WITH GRANT OPTION; flush privileges;
  ```

- 修改mysql配置文件（不同操作系统，不同MySql版本，配置文件存储不同，ubuntu18.04配置文
  件保存在/etc/mysql/mysql.conf.d/mysqld.cnf; Centos7.7 配合文件保存在 /etc/my.cnf） 

  ```bash
  sudo vim /etc/my.cnf
  ```

将 bind-address=127.0.0.1 代码注释掉（如果这段代码存在的话），让计算机允许mysql远程登录。

- 打开服务器的 3306 端口。

- 使用客户端远程登录到mysql数据库。

  ```bash
  sudo mysql -uzhangsan -h 远端服务器地址 -P 3306 -p
  ```

# 四、数据库的操作

## 1.创建数据库

```mysql
create database [if not exists] `数据库名` charset=字符编码(utf8mb4);
```

1. 如果多次创建会报错

2. 如果不指定字符编码，默认为 utf8mb4 (一个汉字占用 4 个字节) 

3. 给数据库命名一定要习惯性加上反引号, 防止和关键字冲突

## 2.查看数据库

```mysql
show databases;
```

## 3.选择数据库

```mysql
use `数据库的名字`;
```

## 4.创建数据库

```mysql
create database `数据库名`;
```

## 5.修改数据库

```mysql
-- 只能修改字符集 
alter database `数据库名` charset=字符集;
```

## 6.删除数据库

```mysql
drop database [if exists] `数据库的名字`;
```

# 五、表的操作

表是建立在数据库中的数据结构，是一类数据的存储集。

## 1. 表的创建

```mysql
create table [if not exists] `表的名字`( 
    id int not null auto_increment primary key comment '主键', 
    account char(255) comment '用户名' default 'admin', 
    pwd text(16383) comment '密码' not null 
)charset=utf8mb4;
```

备注：

- 字符集如果不指定，默认继承库的字符集.

## 2.查看所有的表

选择数据库后，才能查看表

```mysql
show tables;
```

## 3.删除表

删除表必须在数据库中进行删除

```mysql
drop table [if exists] `表名`
```

## 4.显示建表结构

```mysql
desc `表名`;
describe `表名`;
```

## 5.修改表

```mysql
-- 修改表的名称 
alter table `old_name` rename `new_name`; 
-- 移动表 到指定的数据库 
alter table `表名` rename to 数据库名.表名;
```

## 6.修改字段

```mysql
-- 增加一个新的字段 
alter table `表名` add `字段名` 数据类型 [属性]; 
-- 增加一个新的字段, 并放在首位 
alter table `表名` add `字段名` 数据类型 [属性] first; 
-- 增加一个新的字段, 并放在某一个字段之后 
alter table `表名` add `字段名` 数据类型 [属性] after 指定字段; 
-- 修改字段的属性 
alter table `表名` modify `字段名` 数据类型 [属性]; 
-- 修改字段的名称 
alter table `表名` change `原字段名` `新的字段名` 数据类型 [属性];
-- 修改字段的位置 
alter table `表名` change `原字段名` `新的字段名` 数据类型 after `指定字段`; 
-- 删除字段 
alter table `表名` drop `字段名`;
```

## 8.复制表

1. 先在创建一个表，并在表中插入一些数据

   ```mysql
   /* 创建 abc 表*/ 
   create table abc( 
       id int primary key auto_increment comment '主键', 
       username char(32) not null comment '账户', 
       password char(32) not null comment '密码' 
   );
   
   /* 插入两条数据 */ 
   insert into abc values(null, 'tom', md5(123456)), (null, 'bob', md5(123456));
   ```

   

2. 复制表，并且复制数据

   - 执行：

     ```mysql
     create table `复制表的名称` select * from `原表名`
     ```

   - 特点：
     - 完整的复制一个表，既有原表的结构，又有原表的数据
     - 表内字段的属性会丢失，主键的自增等特性不复存在，新插入数据时会有问题
     - **最好不要使用这种方式复制**

- 仅复制表结构，不复制数据

  - 执行：

    ```mysql
    create table `复制表的名称` like `原表名`;
    ```

  - 特点：复制后的表结构与原表完全相同，字段的属性与原表也完全一致。但是里面没有数据，是一张空表

  - 如果需要，数据可以单独复制

    ```mysql
    insert into `复制表的名称` select * from `原表名`;
    ```

  

# 六、CURD语句的基本使用

对表中数据的操作一般分为四类, 常记做 "CURD":

- C: 创建（Create）

- U: 更新（Update）

- R: 读取（Retrieve）

- D: 删除（Delete） 

## 1. INSERT插入

完整的insert语句为:

```mysql
INSERT INTO `表名` (`字段1`, `字段2`, ...) VALUES (`值1`, `值2`, ...);
```

其中的 INTO 在 MySQL 数据库中可以省略, 但在某些数据库中必须要有。

```mysql
-- 一次插入一行 
insert into `表名` set `字段`=值, `字段`=值; 
-- 按照指定字段, 一次插入多行 
insert into `表名` (字段1, 字段2 ...) values (值1, 值2, ...), (值1, 值2, ...); 
-- 指定全部字段, 一次插入多行 
insert into `表名` values (null, 值1, 值2, ...), (null, 值1, 值2, ...);
```

## 2. SELECT(查询)

```mysql
-- 通过 * 获取全部字段的数据 
select * from `表名`; 
-- 获取指定字段的数据 
select `字段1`, `字段2` from `表名`;
```

## 3. UPDATE(更新)

```mysql
-- 修改全表数据 
update `表名` set `字段1`=值, `字段2`=值; 
-- 使用 where 修改满足条件的行 
-- where 类似于 if 条件, 只执行返回结果为 True 的语句 
update `表名` set `字段1`=值, `字段2`=值 where `字段`=值; 
update `表名` set `字段1`=值, `字段2`=值 where `字段`=值 and `字段`=值;
```

## 4. DELETE()删除

```mysql
-- 删除表中的所有数据 (逐行删除) 
delete from `表名`; 
-- 清空全表 (一次性整表删除) truncate `表名` 
-- 使用 where 修改满足条件的行 
delete from `表名` where `字段` = 值; 
delete from `表名` where `字段` in (1, 2, 3, 4);
```

# 七、小总结

- 增

  - **数据库**:

    ```mysql
    create database `库名`;
    ```

  - **表**:

    ```mysql
    create table `表名`;
    ```

  - **字段**:

    ```mysql
    alter table `表名` add `字段名` 类型 [属性];
    ```

  - **数据**:

    ```mysql
    insert into `表名`;
    ```

- 删

  - 数据库:

    ```mysql
    drop database `库名`;
    ```

  - 表:

    ```mysql
    drop table `表名`;
    ```

  - 字段:

    ```mysql
    alter table `表名` drop `字段名`;
    ```

  - 数据:

    ```mysql
    delete from `表名` where ...;
    ```

- 改

  - 数据库:

    ```mysql
    alter database `库名` ...;
    ```

  - 表:

    ```mysql
    alter table `表名` ...;
    ```

  - 字段:

    ```mysql
    alter table `表名` modify | change ...;
    ```

  - 数据:

    ```mysql
    update `表名` set ...;
    ```

- 查

  - 数据库:

    ```mysql
    show databases [like ...];
    ```

  - 表:

    ```mysql
    show tables [like ...];
    ```

  - 字段:

    ```mysql
    desc `表名`;
    ```

  - 数据:

    ```mysql
    select * from `表名` where ...;
    ```

    

