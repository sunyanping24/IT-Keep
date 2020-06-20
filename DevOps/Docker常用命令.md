<!-- TOC -->

- [创建Docker镜像](#创建docker镜像)
  - [.dockerignore文件](#dockerignore文件)
- [镜像操作](#镜像操作)
- [容器操作](#容器操作)
- [容器文件拷贝](#容器文件拷贝)

<!-- /TOC -->

# 创建Docker镜像
命令格式：`docker build [options] path|url`

参数可取：    
- `-build-arg=[]`：设置镜像创建时的变量；
- `--cpu-shares`：设置 cpu 使用权重；
- `-cpu-period`：限制 CPU CFS周期；
- `--cpu-quota`：限制 CPU CFS配额；
- `-cpuset-cpus`：指定使用的CPU id；
- `--cpuset-mems`：指定使用的内存 id；
- `--disable-content-trust`：忽略校验，默认开启；
- `-f`：指定要使用的Dockerfile路径；
- `-force-rm`：设置镜像过程中删除中间容器；
- `--isolation`：使用容器隔离技术；
- `--label=[]`：设置镜像使用的元数据；
- `-m`：设置内存最大值；
- `--memory-swap`：设置Swap的最大值为内存+swap，"-1"表示不限swap；
- `--no-cache`：创建镜像的过程不使用缓存；
- `--pull`：尝试去更新镜像的新版本；
- `--quiet, -q`：安静模式，成功后只输出镜像 ID；
- `--rm`：设置镜像成功后删除中间容器；
- `--shm-size`：设置/dev/shm的大小，默认值是64M；
- `--ulimit`：Ulimit配置。
- `--tag, -t`：镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签。
- `--network`：默认 default。在构建期间设置RUN指令的网络模式

```
docker build -t <image_name>:<tag_name> .    // 在Dockerfile同级目录下执行该命令，根据Dockerfile内容生成镜像
docker build -t <image_name>:<tag_name> -f <Dockerfile_path> . // 不在Dockerfile同级目录下时执行命令，可以使用-f 参数来指定Dockerfile的位置
```

**注意：在`docker build`命令的最后有个`.`，这个点很多人可能理解的是Dockerfile的文件位置，这是错误的。Dockerfile的文件的位置是使用`-f`参数来指定的。最后的`.`是指定的`docker build`这个命令执行的上下文环境。**
常常在Dockerfile中会存在类似于COPY这样的指令，`COPY ./home/test.txt /test`，复制的文件不是在本机的/home目录下的文件，而是上下文环境的/home/test.txt文件。

## .dockerignore文件
`.dockerignore`文件就比较类似于`.gitignore`文件，可以用来忽略某些文件。

当我们在 `docker build` 的过程中，首先会将指定的上下文目录打包传递给 **docker引擎**，而这个上下文目录中可能并不是所有的文件我们都会在 Dockerfile 中使用到，那么这个时候就可以在 **.dockerignore** 文件中指定在传递给 **docker引擎** 时需要忽略掉的文件或文件夹。

#  镜像操作
```
docker images    // 列出所有本地的镜像
docker images -a    // 列出所有本地的镜像，包含中间层镜像
docker images -q    // 列出本地所有的镜像，只显示镜像ID
docker images --no-trunc    // 显示镜像完整信息
docker search <image_name>    // 搜索镜像仓库的镜像
docker pull <image_name>    // 下载镜像到本地
docker rmi <image_name>   // 删除本地镜像
docker rmi -f $(docker images -q)   // 强制删除本地所有镜像
```

# 容器操作
```
docker run -i -t --name <container_name> -d  -p <container_port>:<host_port> <image_name>    // 新建并启动容器
docker start <container_id>   // 启动一个停止状态的容器
docker stop <container_id>    // 停止一个运行状态的容器
docker restart <container_id>   // 重启一个容器
docker ps // 列出所有正在运行的容器
docker ps -a  //   列出所有容器
docker ps -s    // 查看容器总文件大小
docker top <container_id>   // 查看一个容器的进程信息
docker logs <container_id>  // 查看容器的运行日志
docker logs -f -t --tail=2 <container_id>   // 查看容器日志，参数：-f  跟踪日志输出；-t   显示时间戳；--tail  仅列出最新N条容器日志；
docker logs --since="2019-05-21" --tail=10 <container_id>   // 查看容器从2019年05月21日后的最新10条日志。
docker exec -it <container_id> /bin/bash   // 在容器中打开新的交互模式终端，可以启动新进程，参数：-i  即使没有附加也保持STDIN 打开；-t  分配一个伪终端
docker exec -i -t <container_id> ls -l /tmp   // 以交互模式在容器中执行命令，结果返回到当前终端屏幕
docker exec -d <container_id>  touch cache.txt  // 以分离模式在容器中执行命令，程序后台运行，结果不会反馈到当前终端
docker inspect <container_id>   // 查看容器的信息
docker kill  <container_id>   // 杀死一个容器
docker commit -a="作者" -m="提交信息" <container_id> <image_name>:<tag_name>    // 基于当前容器创建一个新的镜像；参数：-a 提交的镜像作者；-c 使用Dockerfile指令来创建镜像；-m :提交时的说明文字；-p :在commit时，将容器暂停
```

# 容器文件拷贝
```
docker cp <container_id>:/<container_path> <local_path>   // 从容器向本地服务器拷贝文件
docker cp <local_path> <container_id>:/<container_path>/  // 从本地服务器向容器拷贝文件
docker cp <local_path> <container_id>:/<container_path>   // 从本地服务器向容器拷贝文件，目录重命名为<container_path>
```
