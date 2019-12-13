<!-- TOC -->

- [try...catch...finally](#trycatchfinally)

<!-- /TOC -->

# try...catch...finally
java中处理异常必然会用到的代码块，通常的几种出现方式：`try...catch`、`try...finally`、`try...catch...finally`。也就是说try是必不可少的块。
```
try {
  // 可能出现异常的代码块
} catch() {
  // 异常处理
} finally {
  // 无论是否出现异常都需要执行的代码块。
  // 通常用来：释放物理链接（数据库连接、网络连接、数据磁盘链接）
}
```
**如果在finally块中只是为了释放这些物理连接，那可以使用JAVA7中提供的一个语法糖（自动资源管理技术）来省略finally块的简写方式。如下：这样当代码块使用完创建的流时会自动关闭资源的占用。**
```
try (FileInputStream stream = new FileInputStream();) {

} catch {

}
```
**区分一个概念**： Java的GC清理的是内存区域的数据，并不会管物理连接，这是需要手动进行关闭的。  
```
try {
  return 0;
} catch  () {
  return 1;
} finally  {
  return 2;
}
```
//输出： 2  
**finally块中的输出是会将try、catch块中的输出覆盖掉的。所以不要在finally块中使用`return`、`throw`语句。**  
**通常在finally块都是会执行的，除非出现类似于`System.exit(-1)`这样的直接退出程序的代码。**

**总结：**
- 程序先执行try的代码，若遇到异常，则执行catch代码块，再执行finally块；若无异常，则直接执行finally代码块。
- finally中的return、throw会覆盖掉前面的return和throw。
- 除非遇到退出程序的代码，否则finally都是会执行的。