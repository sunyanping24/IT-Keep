<!-- TOC -->

- [存储过程](#存储过程)
  - [创建存储过程](#创建存储过程)
  - [存储过程的调用](#存储过程的调用)
  - [存储过程输入参数说明](#存储过程输入参数说明)
  - [查看存储过程的信息](#查看存储过程的信息)
  - [`DECLARE`和`SET`两种定义变量的方式](#declare和set两种定义变量的方式)
  - [预处理语句（`PREPARE...FROM...`、`EXECUTE...`、`DEALLOCATE PREPARE ...`）](#预处理语句preparefromexecutedeallocate-prepare-)

<!-- /TOC -->

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

## `DECLARE`和`SET`两种定义变量的方式

这两种都是可以在复合语句中定义变量的指令，`DECLARE`相比于`SET`指令可以类似于java中的局部变量和全局变量。SET定义的变量也称作为会话变量。所以DECLARE声明的变量相当于只是在BEGIN/END块中定义了而已，而SET定义的变量就可以在任何地方使用。

1. `DECLARE`
```
CREATE PROCEDURE p11 () 
BEGIN 
    DECLARE x1 CHAR(5) DEFAULT 'outer'; 
    BEGIN 
        DECLARE x1 CHAR(5) DEFAULT 'inner'; 
        SELECT x1; 
    END; 
    SELECT x1; 
END 
```
上面的这段过程是合法的。嵌套的BEGIN/END中的*x1*变量在执行到END语句时,就会消失。那么就不会作用到外部的作用域。如果需要保留内部*x1*的值的时候可以将其值保存到会话变量。   
在定义变量的时候可以使用`DEFAULT`子句给变量初始化值，若不初始化值则默认为null；也可以使用`SET`子句来进行赋值。

2. `SET`
```
CREATE PROCEDURE temp()
BEGIN
  DECLARE a INT DEFAULT 1;
 
  SET a=a+1;
  SET @b=@b+1;
  SELECT a,@b;
 
END
```
执行`SET @b=1; `初始化，再调用存储过程`call temp();`当多次调用存储过程时，我们会发现变量a的值不会改变，而变量@b的值会一直增加，这正好验证了SET定义的是全局变量，DECLARE定义的是局部变量。

## 预处理语句（`PREPARE...FROM...`、`EXECUTE...`、`DEALLOCATE PREPARE ...`）
```
CREATE DEFINER=`mysql`@`%` PROCEDURE `customer_test1`(in v_name LONGTEXT)
BEGIN
    ## 定义变量
	SET @TEMP = "SELECT * FROM customer WHERE 1=1";
	## 拼接参数
	IF v_name is not null THEN 
		SET @TEMP = CONCAT(@TEMP, " AND `name` = '", v_name, "'");
	END IF;
	## 预处理
	PREPARE SIMT FROM @TEMP;
    ## 执行
	EXECUTE SIMT;
	## 清理预处理的声明
	DEALLOCATE PREPARE SIMT;
END
```
上面的这段例子就可以很直观的说明这几个指令的用法。其中值得注意的是，`prepare ... from...`的from后面跟的变量必须使用`set`来进行声明，而不能使用`declare`来进行声明。因为`prepare`、`execute`语句相当于在另外的栈帧中执行，不是在同一栈帧中执行，所以必须得使用全局变量来进行声明。




