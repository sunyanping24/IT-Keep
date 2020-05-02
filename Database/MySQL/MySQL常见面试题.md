<!-- TOC -->

- [MySQL的体系结构](#mysql的体系结构)
- [一条Mysql语句的执行过程](#一条mysql语句的执行过程)
- [MySQL几种连接查询的区别](#mysql几种连接查询的区别)

<!-- /TOC -->

# MySQL的体系结构
MySQL的体系结构可以分成3个部分：    
- **mysql-client**： 比如一个命令行，或者Java中使用JDBC
- **mysql-server**：连接器（管理连接权限认证）、查询缓存（命中缓存的存起来）、分析器（语法分析）、优化器（执行计划选择、索引选择）、执行器（操作，返回结果）
- **mysql-存储引擎**：负责存储数据，提供读写接口（建表的时候指定MyISAM，InnoDB，Memory）

![MySQL逻辑架构](http://sunyanping.gitee.io/it-keep/ASSET/MySQL逻辑架构.jpeg)

# 一条Mysql语句的执行过程
`sql语句:select * from T where ID = 1`   

1. 连接器  
首先需要经过连接器,建立与MySQL的连接,在这里会做身份认证(验证账号密码)、权限读取(获取你的相关权限,用于做权限的逻辑判断),而且这会有个线程池用于管理线程。

2. 查询缓存   
验证身份通过后,会在查询缓存中查询找有没有缓存,命中的话就直接返回结果,否则进入分析器。查询缓存是以键值对的形式保存缓存的,key存储sql语句,value存储查询结果。  
**ps:建议关闭查询缓存。因为当表的更新时,相应表的查询缓存会被全部清空,这会导致缓存的命中率很低,维护查询缓存也会消耗一定的性能**

3. 分析器  
首先进行"词法分析",从你输入的SQL中识别出"select"则认为这是查询语句,还会识别出"T"为表名,"ID"为列名等等。然后进行"语法分析",判断整个sql语句是否错误,并判断是否存在"T"表,是否存在列"ID"

4. 优化器   
在这会对SQL语句进行优化,比如索引的选取,多表关联(join)时连接表的顺序等,然后选取最优的方案生成执行计划  
**ps:优化器有时也会有出错,比如选错索引**

5. 执行器    
首先判断该用户有无对该表查询的权限,无则直接返回,有则根据执行计划执行SQL语句。执行完成后，将结果缓存到查询缓存中,并返回结果给客户端。 

# MySQL几种连接查询的区别
1. 内连接 `inner join ... on ...`
**组合两个表中的记录，返回关联字段相符的记录，也就是返回两个表的交集（阴影）部分。**      
![MySQL连接查询-内连接图示](http://sunyanping.gitee.io/it-keep/ASSET/MySQL连接查询-内连接图示.png)

2. 左连接 `left join ... on ...`   
** 左(外)连接，左表(a_table)的记录将会全部表示出来，而右表(b_table)只会显示符合搜索条件的记录。右表记录不足的地方均为NULL。**     
![MySQL连接查询-左连接图示](http://sunyanping.gitee.io/it-keep/ASSET/MySQL连接查询-左连接图示.png)

3. 右连接 `right join ... on ...`
**右(外)连接，左表(a_table)只会显示符合搜索条件的记录，而右表(b_table)的记录将会全部表示出来。左表记录不足的地方均为NULL。**      
![MySQL连接查询-右连接图示](http://sunyanping.gitee.io/it-keep/ASSET/MySQL连接查询-右连接图示.png)

4. 全连接 `... union ...` 和 `... union all ...`
`union` 语句注意事项：
- 通过`union`连接的sql两边取出的结果集列数必须一致；
- ** 使用`union` 时，完全相等的行，将会被合并**，由于合并比较耗时，一般不直接使用 `union` 进行合并，而是通常采用`union all `进行合并；**`union all` 会保留那些重复的数据**；
- 被 `union` 连接的sql 子句，单个子句中不用写 `order by` ，因为不会有排序的效果。但可以对最终的结果集进行排序；   
**union连接会将两边数据相同的行进行合并**     
![MySQL连接查询-全连接UNION图示](http://sunyanping.gitee.io/it-keep/ASSET/MySQL连接查询-全连接UNION图示.jpg)    
**union all连接不会自动合并两边相同数据行，它包含两边所有的数据行**   
![MySQL连接查询-全连接UNION ALL图示](http://sunyanping.gitee.io/it-keep/ASSET/MySQL连接查询-全连接UNION%20ALL图示.jpg)




