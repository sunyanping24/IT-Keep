<!-- TOC -->

- [Mybatis中的占位符有几种，有什么区别？](#mybatis中的占位符有几种有什么区别)
- [需要做分页查询时，分页查询的sql如何写？](#需要做分页查询时分页查询的sql如何写)
- [常用的标签都有哪些？](#常用的标签都有哪些)
- [Mybatis的接口绑定有哪些实现方式？](#mybatis的接口绑定有哪些实现方式)
- [Mybatis的DAO接口文件中的方法是否可以重载?](#mybatis的dao接口文件中的方法是否可以重载)
- [Mybatis的一级缓存和二级缓存](#mybatis的一级缓存和二级缓存)

<!-- /TOC -->


MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

Mybatis也是目前市场上技术选型中最常选用的持久层框架之一。在面试过程中Mybatis也常常会是技术面试中容易被提问的一块知识。

## Mybatis中的占位符有几种，有什么区别？
占位符有两种：`${}`和`#{}`

- `${}`: 位置占位符，`${}`会被参数直接替换，相当于字符串的拼接。比如`select * from uase where name=${}`，参数`小明 or 1=1`，会被拼接成`select * from user where name='小明' or 1=1`。明显看到参数值实际上就不是参数值了。而是变成sql语句（非值）的一部分，所以会导致SQL侵入问题。
- `#{}`: 参数占位符，`#{}`占用的位置只会替换成参数值。比如`select * from uase where name=${}`，参数`小明 or 1=1`，会被拼接成`select * from user where name='小明 or 1=1'`。这样就可以避免sql入侵问题。因为`#{}`在编译时会被`?`替换，相当于使用了`PreparedStatement`的原理。

## 需要做分页查询时，分页查询的sql如何写？
使用`limit`。`limit a, b`，a-代表从a+1条数据开始查询，b-代表查询几条数据，a从0开始，0代表查询的是第一页。

比如参数分页参数`pageNumber=2,pageSize=10`。可以通过计算得出：`a=(pageNumber-1) * pageSize, b=pageSize`。  
注意：在项目中通常要注意，接口调用时pageNumber是从0开始传还是从1开始传。如果从0开始传，则不需要`pageNumber-1`直接`pageNumber`就是正确的。总之就是按照`limit`后面数字的意思，可以自行调整适用的公式。

## 常用的标签都有哪些？
最常见的增删改查：`<select>` `<update>` `<delete>` `<insert>`，动态拼接sql的：`<where>` `<if>` `<when>` `<set>` `<choose>` `<otherwise>` `<trim>` `<foreach>` `<bind>`，映射字段时常用的：`<resultMap>` `<result>`，复用sql时经常用的：`<sql>` `<include>`，多对一、一对多、多对多结果映射时用的：`<association>` `<collection>`

- `<bind>`标签：可以用来代替`concat()`函数。在面对不同类型数据库时，可能`concat()`函数不一定适用，而`<bind>`可以解决这种问题。

## Mybatis的接口绑定有哪些实现方式？
接口绑定，就是在MyBatis中任意定义接口，然后把接口里面的方法和SQL语句绑定，我们直接调用接口方法就可以，这样比起原来了SqlSession提供的方法，可以有更加灵活的选择和设置。

接口绑定有两种实现方式，一种是通过注解绑定，就是在接口的方法上面加上 @Select、@Update等注解，里面包含Sql语句来绑定；另外一种就是通过xml里面写SQL来绑定，在这种情况下，要指定xml映射文件里面的namespace必须为接口的全路径名。

当Sql语句比较简单时候，用注解绑定，当SQL语句比较复杂时候，用xml绑定。一般情况下，用xml绑定的比较多。

使用MyBatis的mapper接口调用时要注意的事项有：
- Mapper接口方法名和mapper.xml中定义的每个sql的id相同；
- Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql 的parameterType的类型相同；
- Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同；
- Mapper.xml文件中的namespace即是mapper接口的类路径。

## Mybatis的DAO接口文件中的方法是否可以重载?
**不能重载!!!**    
因为DAO接口文件中的接口和xml文件中的sql对应关系是：全DAO类路径+方法名=标签的id名，如果方法可以重载，那id则就相当于不唯一，那就无法正常对应。

## Mybatis的一级缓存和二级缓存

1. 一级缓存: 基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空，默认打开一级缓存。**（一级缓存只在数据库会话内部共享。）**

2. 二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同在于其存储作用域为 Mapper(Namespace)，并且可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现Serializable序列化接口(可用来保存对象的状态)，可在它的映射文件中配置 <cache /> ；

3. 对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存Namespaces)的进行了C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。

参考文章：[https://tech.meituan.com/2018/01/19/mybatis-cache.html](https://tech.meituan.com/2018/01/19/mybatis-cache.html)

