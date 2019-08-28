# DDL语句（数据库操作语句）
```
create database <dbname>;   // 创建数据库
use <dbname>;   // 指定使用的数据库
drop database <dbname>;  // 删除指定的数据库 
create table <tablename> ([column...]);     // 创建表
drop table <tablename>;     // 删除表
show create table <tablename> ;     // 查看表的定义语句
alter table <tablename> rename <new_tablename>;     // 修改表名
```
# DML语句（数据操作语句）

# DCL语句（用户、角色、权限相关）

# 其他常用的语句

## 数据复制  
```
insert into table1(c1, c2) select a1,a2 from table2;        // 将table2的数据赋值到table1中，前提是table1需要存在
select count(1) into @a from tablename;     // 将表的数据数复制到变量@a上
create table <new_table> select * from <table>;       // 创建new_table并且将table的数据结构和数据一并复制到new_table
create table <new_table> select * from <table> where 1=2;   // 创建新表new_table,并且保持和table的数据结构一致
```

## 存在则更新，不存在则新增  
通常我们在做这个需求的时候会考虑先查询，根据唯一字段或者其他唯一的标识来查询表中的数据数目，当数量大于0是，则进行更新，当数量等于0时则进行新增。所以一般包含2步。现在总结几种只需要一步就可以实现的方法。  

以下几种方式都需要遵循要求：
-  **使用此种方式的前提必须有主键或者有唯一索引。原理就是根据主键/唯一索引来判断进行新增/更新操作**
- **作为唯一索引的字段（单个或者多个）不允许为空**

1.  使用`on duplicate key update`语句
```
insert into customer(name, id_card, phone) values('张三', '1234', '110')
on duplicate key 
update phone = '120';
```
（1）当表中没有数据执行新增时，影响的数据行数是1；当表中有主键/唯一索引相同的数据时，并且更新的字段值和当前数据表中的数据不同时，影响的数据行数是；当表中有主键/唯一索引相同的数据时，并且更新的字段值和当前数据表中的数据相同时，影响的数据行数是0；

2. 使用`replace`  
`replace`相当于包含了2部分的操作：`delete`和`insert`。但是`replace`是一个原子操作。
```
replcae into customer(name, id_card, phone) values('lisi', '23332'，'23233' );
```
（1）当表中没有数据执行新增时，影响的数据行数是1；当表中有主键/唯一索引相同的数据时，并且更新的字段值和当前数据表中的数据不同时，影响的数据行数是；当表中有主键/唯一索引相同的数据时，并且更新的字段值和当前数据表中的数据相同时，影响的数据行数是1；

3. 使用`insert ignore`  
这个和上面2个不同，这个是：不存在则新增，存在则忽略插入，而不是更新。
```
inert ignore into customer(name, id_card, phone) values('huan', '23334', '23424');
```


