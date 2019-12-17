<!-- TOC -->

- [curl](#curl)
- [vi](#vi)
- [grep](#grep)
- [awk](#awk)

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