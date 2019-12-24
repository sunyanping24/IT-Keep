<!-- TOC -->

- [curl](#curl)
- [vi](#vi)
- [grep](#grep)
- [awk](#awk)
- [shell脚本第一行的`#!/bin/bash`和`#!/bin/sh`](#shell脚本第一行的binbash和binsh)
- [执行脚本的几种方式](#执行脚本的几种方式)
- [export](#export)
- [dirname](#dirname)
- [EOF](#eof)

<!-- /TOC -->

# curl
*用于传输来自服务器或者到服务器数据的工具*  
当指定的url没有协议类型时默认使用的是http协议。当没有指定请求方法时默认是get方式。

- `-#, --progress-bar`：curl进度显示为一个进度条，而不是标准的进度表
- `-a, --append`: (FTP/SFTP)当在上传中使用，告诉curl追加到目标文件而不是覆盖它，如果文件不存在将创建它。
- `-d, --data`: 使用该选项默认请求方式是post，指定向服务器发送的数据。这和提交表单的按钮是同样的，默认curl使用content-type application/x-www-form-urlencoded将数据传输给服务器。也可以使用`-F, form`指定。如果这些命令在同一个命令行使用多次，这些数据片段将使用指定的分隔符`&`合并。如果以`@`开始数据，则后面的应该跟的是一个文件名，一便读取文件中的数据。
- `-e, --referer`: 将从哪个页面跳转过来的信息发送到服务器。例如：`curl -e 'http://www.baidu.com' http://host:port/api`
- `-f, --fail`: 静默失败
- `-G, --get`: 指定get请求方式
- `-H, --header`: 定义请求头，例如：`curl -H 'Connection: keep-alive' -H 'Referer: https://sina.com.cn' -H 'User-Agent: Mozilla/1.0' http://www.zhangblog.com/2019/06/24/domainexpire/`
- `-o, --output`: 输出到一个文件，而不是标准输出
- `-s, --silent`: 静默
- `-S, --show-error`: 和`-s`一起使用时，如果出现失败，则失败信息会显示
- `-X, --request`: 指定请求方式
- FTP下载：`curl -O ftp://ftp1:123456@172.16.1.195:21/tmpdata/tmp.data`或者`curl -O -u ftp1:123456 ftp://172.16.1.195:21/tmpdata/tmp.data`
- `-T, --upload-file`: FTP文件上传，`curl -T tmp_client.data ftp://ftp1:123456@172.16.1.195:21/tmpdata/`或者`curl -T tmp_client.data -u ftp1:123456 ftp://172.16.1.195:21/tmpdata/`

*example*
```
# POST请求注意参数是json需要放在body传递时必须设置Header(Content-Type: application/json)
curl -X POST http://localhost/ -H POST -H 'Content-Type: application/json' -d '{"name": "张三", "age": 10}'  
curl -X POST http://localhost -d 'name=张三&age=10'
```

# vi
- 命令模式
```
yyp   // 复制当前行到下一行
dd    // 删除当前行
shift+g   // 跳到文本的最后一行
gg    // 跳到文本的第一行
dG    // 使用gg先跳到文本的第一行，再使用dG，可以清除文本的内容
```
- 输入模式（使用`i`进入）
```
```

- 底线命令模式/编辑模式（使用`:`进入）
```
:%s/<str>//gn   // 匹配整个文件中的str字符串，显示匹配到多少个，可以用来查看统计
```

# grep
```
grep -E "a|b"   // 匹配a或者b，注意一定是大写E参数
```

# awk
```
awk '{print $1}'    // 截取第一列，注意一定是单引号
```

# shell脚本第一行的`#!/bin/bash`和`#!/bin/sh`
在shell脚本中第一行往往有这样一行代码，这虽然是`#`开头但是它不是注释，这行代码是用来指定执行脚本文件时使用哪种解释执行器。
在linux系统中默认使用的是`bash`解释执行器，`bash`是兼容`sh`的。在`#!/bin/sh`脚本中不允许出现任何不在POSIX标准的命令，
比如脚本中包含一些自定义函数等，就需要使用`#!/bin/bash`。  

# 执行脚本的几种方式
- `bash xxx.sh`或者`sh xxx.sh`: 指定解释执行器执行脚本，脚本没有执行权限时，这种方式也可以执行。
- 指定路径执行`./xxx.sh`： 脚本没有执行权限时，这种方式不能执行。
- `source xxx.sh`或者`. xxx.sh`：会将shell中得命令加载在当前shell中进行执行，而不是在子shell进程中执行，所以当前shell中可以使用脚本中定义得变量等。

# export
- 可以设定环境变量，使得该变量不仅仅只是生效于当前得shell中
- 该变量仅仅是指生效于当前登陆
```
export INSTALL_HOME=$(
  cd $(dirname $0)
  pwd
)
```

# dirname
```
dirname $0    // $0代表输入的第一个参数，这个参数是一个文件或者目录的路径，dirname用来获取该文件或者目录的父目录
dirname /etc/hosts    // 结果是/etc
```

# EOF
EOF是END Of File的缩写,表示自定义终止符.既然自定义,那么EOF就不是固定的,可以随意设置别名。EOF一般会配合cat能够多行文本输出.

example: 将输入输出到文件
```
[root@iz2ze0bcvmlsi9ddobp0bpz home]# cat <<eof > test111.txt
> 112
> 122
> 122
> eof
[root@iz2ze0bcvmlsi9ddobp0bpz home]# cat test111.txt
112
122
122
[root@iz2ze0bcvmlsi9ddobp0bpz home]#
```

example: 将内容输出到文件
```
cat > /usr/local/mysql/my.cnf << EOF  //或者cat << EOF > /usr/local/mysql/my.cnf
[client]
port = 3306
socket = /usr/local/mysql/var/mysql.sock

[mysqld]
port = 3306
socket = /usr/local/mysql/var/mysql.sock
EOF
```
