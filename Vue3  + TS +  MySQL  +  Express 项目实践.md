# Vue3  + TS +  MySQL  +  Express 项目实践

## MySQL基础

### MySQL安装与连接

#### 安装包下载

[链接](https://downloads.mysql.com/archives/community/)

#### 根据官网教程安装

[链接](https://dev.mysql.com/doc/refman/5.7/en/installing.html)

#### 数据库登录

1、终端运行命令:

```bash
mysql -u root -p
```

2、输入初始密码

3、首次使用初始密码登录需要修改密码

> Mysql密码需要符合**强密码**要求。强密码：必须至少包含一个大写字母、一个小写字母、一个特殊符号、一个数字，密码长度至少为8个字符。

#### 初始密码无法登录

1、停止mysql服务

````bash
sudo pkill mysqld
````

2、禁止mysql验证

````bash
# 终端切入到目标目录
cd /usr/local/mysql/bin
# 切换到root账户
sudo su 
# 禁止验证功能，回车后mysql会自动重启
./mysqld_safe --skip-grant-tables &
````

3、重新设备新密码

````bash
mysql 
# mysql 环境下运行下面的命令
flush privilleges；
set password for 'root'@'localhost' = 'Qing_Mao1234'
````

> 注意：mysql下的单个命令行以`;`作为结尾。

4、退出mysql，使用新密码登录

````bash 
# 退出mysql环境
quit 或者 exit
# 连接mysql，输入刚设置的密码
mysql -u root -p
````

#### CentOS下查看MySQL的初始密码

```
sudo grep "password" /var/log/mysqld.log
```

> 若是首次安装，需要使用命令`systemctl start mysqld.service` 启动数据库，才能生成密码，否则mysqld.log内容为空。

#### MySQL可视化管理工具

* phpadmin
* Php + apache + mysql

* [DataGrip][https://www.jetbrains.com/zh-cn/datagrip/]
* [Navicat Premium][https://www.navicat.com/en/products/navicat-premium]
* [MySQL Workbench][https://www.mysql.com/products/workbench/]
* [Navicat for MySQL][https://www.navicat.com/en/products/navicat-for-mysql]

#### MySQL客户端连接

客户端连接时需要的信息：

* host: 地址，连接本地为localhost或者127.0.0.1。连接远程数据库，地址为服务器的ip地址。
* port: 端口，默认情况下为3306
* user：用户名
* password: 用户密码

> 注意：如果MySQL数据是安装在远程的服务器上，如果是在目前比较主流的阿里云或者腾讯云服务器上，需要在对应的安全组中开始3306端口。
>
> 若服务器关闭了防火墙设置，还需要在防火墙中打开3306端口。

#### 在查询控制中查看数据的基本信息

````sql
# 显示所有的数据库
show databases;
# 查询数据的版本
select version();
````

#### 连接阿里云的RDS MySQL

1、将本地IP地址添加到RDS白名单；

2、获取外网地址后再客户端进行连接。

#### 无法连接数据库的几种可能

1、数据库未处于运行状态；

> 在Linux环境下，使用`systemctl status mysqld.service` 命令检查数据库的运行状态。

2、数据连接的3306端口未开启；

> 如果数据库是部署在远程服务器且服务器开启了防火墙机制，需要开启3306端口。如果是使用主流的云服务器，需要在服务器安全组中开启端口。

3、创建的用户不支持本地连接；

> 创建数据库账户时，如果账户的主机地址为`localhost`将不支持远程连接，可以添加用户的主机地址为`%`或者指定的端口好。

### 用户创建与权限分配

#### 创建数据库

````sql
create  database if not exists `qingmao` charset utf8mb4;
````

> 注意：创建数据库时如果使用数据库默认的编码格式，在表字段存储emoji可能会存在乱码。

#### 创建表

1、创建一个用户表

````sql
create  table  if not exists t_user (
    id  int auto_increment primary key comment '自增ID',
    user_name varchar(20) comment  '用户名',
    user_gender int default 0 comment  '性别，男：0，女：1',
    created_time datetime default now()
) comment '用户表';
````

2、插入一些测试数据

````sql
# 插入单行记录
insert into user_t (user_name)  values('Felix😀');

# 同时插入多行数据
insert into user_t (user_name)  values('Felix😀'), ('Felix'), ('Felix2');
````

#### 通过user表查询所有的用户

````sql
user mysql;
select * from  user;
````

用户权限参数说明：

| 字段名                 | 字段类型      | 是否为空 | 默认值 | 说明                                                         |
| ---------------------- | ------------- | -------- | ------ | ------------------------------------------------------------ |
| Select_priv            | enum('N','Y') | NO       | N      | 是否可以通过SELECT 命令查询数据                              |
| Insert_priv            | enum('N','Y') | NO       | N      | 是否可以通过 INSERT 命令插入数据                             |
| Update_priv            | enum('N','Y') | NO       | N      | 是否可以通过UPDATE 命令修改现有数据                          |
| Delete_priv            | enum('N','Y') | NO       | N      | 是否可以通过DELETE 命令删除现有数据                          |
| Create_priv            | enum('N','Y') | NO       | N      | 是否可以创建新的数据库和表                                   |
| Drop_priv              | enum('N','Y') | NO       | N      | 是否可以删除现有数据库和表                                   |
| Reload_priv            | enum('N','Y') | NO       | N      | 是否可以执行刷新和重新加载MySQL所用的各种内部缓存的特定命令，包括日志、权限、主机、查询和表 |
| Shutdown_priv          | enum('N','Y') | NO       | N      | 是否可以关闭MySQL服务器。将此权限提供给root账户之外的任何用户时，都应当非常谨慎 |
| Process_priv           | enum('N','Y') | NO       | N      | 是否可以通过SHOW PROCESSLIST命令查看其他用户的进程           |
| File_priv              | enum('N','Y') | NO       | N      | 是否可以执行SELECT INTO OUTFILE和LOAD DATA INFILE命令        |
| Grant_priv             | enum('N','Y') | NO       | N      | 是否可以将自己的权限再授予其他用户                           |
| References_priv        | enum('N','Y') | NO       | N      | 是否可以创建外键约束                                         |
| Index_priv             | enum('N','Y') | NO       | N      | 是否可以对索引进行增删查                                     |
| Alter_priv             | enum('N','Y') | NO       | N      | 是否可以重命名和修改表结构                                   |
| Show_db_priv           | enum('N','Y') | NO       | N      | 是否可以查看服务器上所有数据库的名字，包括用户拥有足够访问权限的数据库 |
| Super_priv             | enum('N','Y') | NO       | N      | 是否可以执行某些强大的管理功能，例如通过KILL命令删除用户进程；使用SET GLOBAL命令修改全局MySQL变量，执行关于复制和日志的各种命令。（超级权限） |
| Create_tmp_table_priv  | enum('N','Y') | NO       | N      | 是否可以创建临时表                                           |
| Lock_tables_priv       | enum('N','Y') | NO       | N      | 是否可以使用LOCK TABLES命令阻止对表的访问/修改               |
| Execute_priv           | enum('N','Y') | NO       | N      | 是否可以执行存储过程                                         |
| Repl_slave_priv        | enum('N','Y') | NO       | N      | 是否可以读取用于维护复制数据库环境的二进制日志文件           |
| Repl_client_priv       | enum('N','Y') | NO       | N      | 是否可以确定复制从服务器和主服务器的位置                     |
| Create_view_priv       | enum('N','Y') | NO       | N      | 是否可以创建视图                                             |
| Show_view_priv         | enum('N','Y') | NO       | N      | 是否可以查看视图                                             |
| Create_routine_priv    | enum('N','Y') | NO       | N      | 是否可以更改或放弃存储过程和函数                             |
| Alter_routine_priv     | enum('N','Y') | NO       | N      | 是否可以修改或删除存储函数及函数                             |
| Create_user_priv       | enum('N','Y') | NO       | N      | 是否可以执行CREATE USER命令，这个命令用于创建新的MySQL账户   |
| Event_priv             | enum('N','Y') | NO       | N      | 是否可以创建、修改和删除事件                                 |
| Trigger_priv           | enum('N','Y') | NO       | N      | 是否可以创建和删除触发器                                     |
| Create_tablespace_priv | enum('N','Y') | NO       | N      | 是否可以创建表空间                                           |

#### 新建用户和权限分配

1、 创建用户

````sql
create user 用户名@主机 identified by '密码'

-- 示例
create  user  'test2'@'%' identified  by 'test2_123';
````

2、给用户分配权限

````sql
grant 权限 on 数据库名.表名 to 用户名@主机 identified by '密码'

-- 创建一个test账号对online数据库下的所有表具有查询权限
grant  select on online.* to 'test2'@'%' identified  by 'test2_123';
````

3、查看用户权限

````sql
-- 使用对应账号登录后，可以直接使用下面的命令查询
show grants

-- 查看某个用户的权限
show grants for  prod_font;
````

4、用户权限撤销

````sql
-- 格式
revoke 权限 on 数据库名.表名 from 用户名@本机地址
-- 用户权限收回
revoke  SELECT, INSERT, UPDATE, DELETE ON `ningjin_prod`.* from 'prod_font'@'%';
````

5、用户重命名

```sql
rename user  'test2'@'%' to 'test1'@'%';
```

6、用户删除

````sql
drop  user  'test'@'%';
````

#### 创建角色和权限分配

1、创建角色

````sql
create role  if not exists  '角色名';
````

2、给角色分配权限

````sql
grant 权限列表 on 数据库名.表名 to '角色名'; 
````

3、给用户分配角色

```sql
grant 角色名  to 用户名@主机
```

4、显示用户的角色权限

````sql
show grants  for 用户名@主机 using 角色名
````

5、撤销角色权限

````sql
show grants  for 用户名@主机 using 角色名
````

6、删除角色

````sql
drop role '角色名'
````

### 