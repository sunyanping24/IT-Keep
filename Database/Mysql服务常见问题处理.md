<!-- TOC -->

- [Too many connections](#too-many-connections)
- [Error related to only_full_group_by when executing a query in MySql](#error-related-to-only_full_group_by-when-executing-a-query-in-mysql)
- [mysql多端口实现多实例，使用mysqld_multi管理](#mysql多端口实现多实例使用mysqld_multi管理)
- [清理掉information_schema.processlist表中的连接信息](#清理掉information_schemaprocesslist表中的连接信息)
- [mysql数据库中尝试使用枚举类](#mysql数据库中尝试使用枚举类)
- [问题`max_allowed_packet`](#问题max_allowed_packet)
- [Mysql服务忘记密码](#mysql服务忘记密码)
- [数据库大小写敏感问题](#数据库大小写敏感问题)

<!-- /TOC -->

# Too many connections

**方法一**：修改`my.cnf`文件

`[mysql]`下增加一行配置: `max_connections=102400000`

默认配置此值为100，一般情况下，都需要修改。

**方法二**：使用命令修改

查询该值：`show variables like 'max_connections'`

修改该值：`set global max_connections=102400000`

# Error related to only_full_group_by when executing a query in MySql

> 由于从v5.7.2版本开始，group by的数据列必须涵盖所有select的数据列。使用高版本时在不修改sql的前提下，达到和v5.6版本相同的效果可以在my.cnf中配置以下配置。

```
[mysqld]
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```
# mysql多端口实现多实例，使用mysqld_multi管理

[mysql多端口实现多实例，使用mysqld_multi管理](http://www.webyang.net/Html/web/article_311.html)

# 清理掉information_schema.processlist表中的连接信息

show full processlist;  //列出当前的操作process，一般会看到很多waiting的process，说明已经有卡住的proces了，我们要杀死这些process！！

再执行：

kill processid;  //processid表示process的id，比如kill 3301，就会将id为3301的process杀死。

一般会有很多processid,所以一个一个清理是会死人的

select concat('KILL ',id,';') from information_schema.processlist where user='root';

将查询到的kill id复制出来，一次执行即可


# mysql数据库中尝试使用枚举类
注意点：1、插入值时需要插入的在枚举中的值，不然报错，这点可以有效的防止村的数据不是设计中需要的数据  2、插入值的时候可以不使用枚举中的相应值而是使用枚举的相应的元素的位置（第一个元素的值是1不是0）

# 问题`max_allowed_packet`
查询出现类似问题`Packet for query is too large (12238 > 1024). You can change this value`

mysql max_allowed_packet 设置过小导致记录写入失败

mysql根据配置文件会限制server接受的数据包大小。

有时候大的插入和更新会受max_allowed_packet 参数限制，导致写入或者更新失败。

查看目前配置

show VARIABLES like '%max_allowed_packet%';

修改方法

1、修改配置文件

可以编辑my.cnf来修改（windows下my.ini）,在[mysqld]段或者mysql的server配置段进行修改。

max_allowed_packet = 20M
如果找不到my.cnf可以通过

mysql --help | grep my.cnf
去寻找my.cnf文件。

linux下该文件在/etc/下。

2、在mysql命令行中修改

在mysql 命令行中运行

set global max_allowed_packet = 2*1024*1024*10
然后退出命令行，重启mysql服务，再进入。

show VARIABLES like '%max_allowed_packet%';
查看下max_allowed_packet是否编辑成功

 注意：该值设置过小将导致单个记录超过限制后写入数据库失败，且后续记录写入也将失败。

# Mysql服务忘记密码

`Mysql5.7`

1. 在`my.cnf`中添加一行，可以跳过权限验证。这种情况相当于任何账户都可以对任何表进行操作，这是相当的危险。
所以在修改完成之后要立即关闭。
```
[mysqld]
skip-grant-tables=true
```

2. 重启Mysql实例，然后免密进入mysql命令行，修改root密码。
```
update mysql.user set authentication_string=password('123456') where user='root';
```

# 数据库大小写敏感问题

在Linux下数据库名、表名、列名、别名大小写规则是这样的： 

1、数据库名与表名是严格区分大小写的；

2、表的别名是严格区分大小写的；

3、列名与列的别名在所有的情况下均是忽略大小写的； 

4、变量名也是严格区分大小写的； 

MySQL在Windows下都不区分大小写。

在`[mysqld]` 下配置`lower_case_table_names `可以进行设置。

```
[mysqld]

lower_case_table_names =0  // 0-区分大小写 1-不区分大小写
```
但是在windows环境下进行配置时，该属性取值不能为0，对应的值是1，2