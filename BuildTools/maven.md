<!-- TOC -->

- [`<scope>`-依赖范围](#scope-依赖范围)
- [maven插件](#maven插件)
- [maven常用的命令](#maven常用的命令)

<!-- /TOC -->

# `<scope>`-依赖范围

1. `compile`（编译范围）  
compile是默认的范围；如果没有提供一个范围，那该依赖的范围就是编译范围。编译范围依赖在所有的classpath 中可用，同时它们也会被打包。

2. `provided`（已提供范围）  
provided 依赖只有在当JDK 或者一个容器已提供该依赖之后才使用。它们不是传递性的，也不会被打包。

3. `runtime`（运行时范围）
runtime 依赖在运行和测试系统的时候需要，但在编译的时候不需要。比如，你可能在编译的时候只需要JDBC API JAR，而只有在运行的时候才需要JDBC驱动实现。

4. `test`（测试范围）  
test范围依赖 在一般的编译和运行时都不需要，它们只有在测试编译和测试运行阶段可用。

5. `system`（系统范围）  
system范围依赖与provided 类似，但是你必须显式的提供一个对于本地系统中JAR 文件的路径。这么做是为了允许基于本地对象编译，而这些对象是系统类库的一部分。这样的构件应该是一直可用的，Maven 也不会在仓库中去寻找它。如果你将一个依赖范围设置成系统范围，你必须同时提供一个 systemPath 元素。注意该范围是不推荐使用的（你应该一直尽量去从公共或定制的 Maven 仓库中引用依赖）。


# maven插件

1. `maven-jar-plugin`: 将工程打成jar, 包含的是`src/main/java`、`src/test/java`的java文件
2. `maven-resource-plugin`: 可以将`src/main/java`、`src/main/resource`包含的资源文件打入jar包，也可以指定到其他目录输出
3. `maven-dependency-plugin`：可以将外部依赖打包，可以将依赖中的部分抽取打包等。maven-dependency-plugin最大的用途是帮助分析项目依赖，dependency:list能够列出项目最终解析到的依赖列表，dependency:tree能进一步的描绘项目依赖树。
4. `maven-assembly-plugin`: 可以将文件指定输出到指定目录，也可以给文件赋权限等
5. `maven-antrun-plugin`: 能让用户在Maven项目中运行Ant任务。用户可以直接在该插件的配置以Ant的方式编写Target， 然后交给该插件的run目标去执行。在一些由Ant往Maven迁移的项目中，该插件尤其有用。
6. `maven-war-plugin`: 打war包时可以使用该插件，过滤/添加打包时不需要的资源文件、依赖包等。

# maven常用的命令
平时经常用的就不列举了，列举一些有时会用到的，但是又记不住的。
```
## 将依赖包安装到本地仓库
mvn install:install-file -Dfile=<依赖包的位置> -DgroupId=<groupId> -DartifactId=<artifactId> -Dversion=<版本号> -Dpackaging=jar -Dclassifier=<classifier>
```