# 存储过程

## 创建存储过程
```
create [or replace] procedure sp_name([proc_parameter[,...]])
    [characteristic ...] routine_body
```
**`or replace`是`mariadb10.1.3`版本中才支持的语法，在`mysql`中是不支持的**  
当然也可以先使用`drop procedure if exist sp_name;`语句删除再创建。  

在存储过程创建的过程中，若使用`;`当作结束符会就会出现语法错误。所以在创建存储过程时需要使用结束符时使用`delimiter`命令来暂时修改语句的结束符为其他的的符号，待结束后再将符号修改回`;`即可。
```
delimiter $$
create procedure customer_select()
begin
    select * from customer;
end $$
delimiter ;
```

## 存储过程的调用

存储过程的调用使用语法：`call sp_name([proc_parameter[,...]])`  
在输入参数的时候，注意`IN`模式的参数可以直接传值，但是`OUT`、`INOUT`模式的参数必须以变量的形式传值，并且变量需要以`@`开头来命名。

`OUT`、`INOUT`模式的参数，可以使用`SELECT @VAR;`的形式来获取。

## 存储过程输入参数说明

1. `IN`： 表示将调用者给定的值传递给存储过程。存储过程可能会修改这个值，但是对于调用者来说，在存储过程返回结果时，所做的修改是不可见的。

2. `OUT`：表示将存储过程的返回值传递给调用者。其初始值为NULL，当存储过程返回时，这个值对调用者来说是可见的。

3. `INOUT`：表示由调用者传递值给存储过程，存储过程可能会修改这个值，当存储过程返回的时候，所做的修改对调用者来说是可见的。

**默认每个参数都是IN。要指定其他类型的参数，可以在参数名前面使用关键字OUT或INOUT。**

## 查看存储过程的信息

```
show {procedure} status like 'sp_name';     // 查看sp_name的基本信息
show create {procedure} 'sp_name';       // 查看创建语句
select information_schema.routines where routine_name 'sp_name';    // 查看更详细的过程信息
```



