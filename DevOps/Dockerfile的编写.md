<!-- TOC -->

- [FROM 指定基础镜像](#from-指定基础镜像)
- [RUN 执行命令](#run-执行命令)
- [WORKDIR 指定工作目录](#workdir-指定工作目录)
- [COPY 复制文件](#copy-复制文件)
- [ADD 更高级的复制文件](#add-更高级的复制文件)
- [CMD 容器启动命令](#cmd-容器启动命令)
- [ENTRYPOINT 入口点](#entrypoint-入口点)
- [ENV 设置环境变量](#env-设置环境变量)
- [ARG 构建参数](#arg-构建参数)
- [EXPOSE 声明端口](#expose-声明端口)
- [VOLUME 定义匿名卷](#volume-定义匿名卷)
- [USER 指定当前用户](#user-指定当前用户)
- [HEALTHCHECK 健康检查](#healthcheck-健康检查)
- [ONBUILD 为他人做嫁衣裳](#onbuild-为他人做嫁衣裳)

<!-- /TOC -->


Dockerfile 是一个文本文件，其内包含了一条条的 指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

本篇的内容基本就是这篇文章的**Dockfile执行详解**一节的内容：[Docker从入门到实践](https://yeasy.gitbooks.io/docker_practice/image/dockerfile/)

# FROM 指定基础镜像
FROM 就是指定 基础镜像，因此一个 Dockerfile 中 FROM **是必备的指令，并且必须是第一条指令**。

在Docker Hub上有非常多的高质量的官方镜像，可以从其中选择最符合我们终极目标的镜像为基础镜像进行定制。其中还提供了一些操作系统的镜像。除了选择现有的镜像外，Docker还存在一个特殊的镜像，**scratch**，这个镜像是虚拟的概念，并不实际存在，它表示一个空白镜像。

如果你以 scratch 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

不以任何系统为基础，直接将可执行文件复制进镜像的做法并不罕见，比如 swarm、etcd。对于 Linux 下静态编译的程序来说，并不需要有操作系统提供运行时支持，所需的一切库都已经在可执行文件里了，因此直接 FROM scratch 会让镜像体积更加小巧。使用 Go 语言 开发的应用很多会使用这种方式来制作镜像，这也是为什么有人认为 Go 是特别适合容器微服务架构的语言的原因之一。

# RUN 执行命令
`RUN` 指令是用来执行命令行命令的。由于命令行的强大能力，`RUN` 指令在定制镜像时是最常用的指令之一。其格式有两种：
- shell 格式： `RUN <命令>`
- exec 格式：`RUN ["可执行文件", "参数1", "参数2"]`

**Dockerfile 中每一个指令都会建立一层，RUN 也不例外**。每一个 `RUN` 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，commit 这一层的修改，构成新的镜像。

如果多个shell命令，分行`RUN <命令>` 来执行，这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。 

比如下面的`RUN`命令：
```
RUN cd /usr/src/nginx-1.17.9 \
        && mkdir /usr/local/nginx \
        && ./configure --prefix=/usr/local/nginx && make && make install \
        && ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/
```

**制作的镜像很臃肿的原因之一就是，忘记了在每一层构建完成后清理掉无用的文件。**

# WORKDIR 指定工作目录
格式：`WORKDIR <工作目录路径>`

使用 `WORKDIR` 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，`WORKDIR` 会帮你建立目录。

之前提到一些初学者常犯的错误是把 Dockerfile 等同于 Shell 脚本来书写，这种错误的理解还可能会导致出现下面这样的错误：
```
RUN cd /app
RUN echo "hello" > world.txt
```

如果将这个 `Dockerfile` 进行构建镜像运行后，会发现找不到 `/app/world.txt` 文件，或者其内容不是 hello。原因其实很简单，在 Shell 中，连续两行是同一个进程执行环境，因此前一个命令修改的内存状态，会直接影响后一个命令；**而在 Dockerfile 中，这两行 RUN 命令的执行环境根本不同，是两个完全不同的容器。** 这就是对 Dockerfile 构建分层存储的概念不了解所导致的错误。

之前说过**每一个 `RUN` 都是启动一个容器、执行命令、然后提交存储层文件变更**。第一层` RUN cd /app `的执行仅仅是当前进程的工作目录变更，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的容器，跟第一层的容器更完全没关系，自然不可能继承前一层构建过程中的内存变化。

因此如果需要改变以后各层的工作目录的位置，那么应该使用 `WORKDIR` 指令。

# COPY 复制文件
格式：   
- `COPY [--chown=<user>:<group>] <源路径>... <目标路径>`
- `COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]`

和 `RUN` 指令一样，也有两种格式，一种类似于命令行，一种类似于函数调用。   
`COPY` 指令将从构建上下文目录中 `<源路径>` 的文件/目录复制到新的一层的镜像内的 `<目标路径>` 位置。比如：
```
COPY ./nginx-1.17.9.tar.gz /home/
```
`<源路径>` 可以是多个，甚至可以是通配符，其通配符规则要满足 Go 的 filepath.Match 规则，如：
```
COPY hom* /mydir/
COPY hom?.txt /mydir/
```
`<目标路径>` 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 WORKDIR 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。

此外，还需要注意一点，使用 `COPY` 指令，**源文件的各种元数据都会保留**。比如读、写、执行权限、文件变更时间等。这个特性对于镜像定制很有用。特别是构建相关文件都在使用 Git 进行管理的时候。

# ADD 更高级的复制文件
`ADD` 指令和 `COPY` 的格式和性质基本一致。但是在 `COPY` 基础上增加了一些功能。

比如 **`<源路径>` 可以是一个 `URL`，这种情况下，Docker 引擎会试图去下载这个链接的文件放到 `<目标路径>` 去**。下载后的文件权限自动设置为 600，如果这并不是想要的权限，那么还需要增加额外的一层 `RUN` 进行权限调整，另外，如果下载的是个压缩包，需要解压缩，也一样还需要额外的一层 `RUN` 指令进行解压缩。所以不如直接使用 `RUN` 指令，然后使用 wget 或者 curl 工具下载，处理权限、解压缩、然后清理无用文件更合理。因此，这个功能其实并不实用，而且不推荐使用。

如果 `<源路径>` 为一个 `tar` 压缩文件的话，压缩格式为 `gzip`, `bzip2` 以及 `xz` 的情况下，**ADD 指令将会自动解压缩这个压缩文件到 `<目标路径>` 去**。

在 `Docker` 官方的 Dockerfile 最佳实践文档 中要求，**尽可能的使用 `COPY`，因为 `COPY` 的语义很明确，就是复制文件而已**，而 `ADD` 则包含了更复杂的功能，其行为也不一定很清晰。**最适合使用 `ADD` 的场合，就是所提及的需要自动解压缩的场合。**

另外需要注意的是，ADD 指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。

因此在 `COPY` 和 ``ADD`` 指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用 `COPY` 指令，仅在需要自动解压缩的场合使用 ADD。

# CMD 容器启动命令
格式：   
- `shell` 格式：`CMD <命令>`
- `exec` 格式：`CMD ["可执行文件", "参数1", "参数2"...]`
- 参数列表格式：`CMD ["参数1", "参数2"...]`。在指定了 `ENTRYPOINT` 指令后，用 `CMD` 指定具体的参数。

Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。`CMD` 指令就是用于指定默认的容器主进程的启动命令的。

**在运行时可以指定新的命令来替代镜像设置中的这个默认命令**，比如，ubuntu 镜像默认的 `CMD` 是 `/bin/bash`，如果我们直接 `docker run -it ubuntu` 的话，会直接进入 bash。我们也可以在运行时指定运行别的命令，如 `docker run -it ubuntu cat /etc/os-release`。这就是用 `cat /etc/os-release` 命令替换了默认的 `/bin/bash` 命令了，输出了系统版本信息。

在指令格式上，一般推荐使用 `exec` 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 "，而不要使用单引号。

如果使用 shell 格式的话，实际的命令会被包装为 sh -c 的参数的形式进行执行。比如：`CMD echo $HOME` 在实际执行中，会将其变为 `CMD [ "sh", "-c", "echo $HOME" ]` 。这就是为什么我们可以使用环境变量的原因，因为这些环境变量会被 shell 进行解析处理。

提到 CMD 就不得不提容器中应用在前台执行和后台执行的问题。这是初学者常出现的一个混淆。Docker 不是虚拟机，**容器中的应用都应该以前台执行**，而不是像虚拟机、物理机里面那样，用 systemd 去启动后台服务，**容器内没有后台服务的概念**。

对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。
正确的做法是直接执行 nginx 可执行文件，并且要求以前台形式运行。比如：`CMD ["nginx", "-g", "daemon off;"]`

# ENTRYPOINT 入口点
`ENTRYPOINT` 的目的和 `CMD` 一样，都是在**指定容器启动程序及参数**。`ENTRYPOINT` 在运行时也可以替代，不过比 `CMD` 要略显繁琐，需要通过 `docker run` 的参数 `--entrypoint` 来指定。

**当指定了 `ENTRYPOINT` 后，`CMD` 的含义就发生了改变，不再是直接的运行其命令，而是将 `CMD` 的内容作为参数传给 `ENTRYPOINT` 指令**，换句话说实际执行时，将变为：`<ENTRYPOINT> "<CMD>"`

有了`CMD`后为什么还需要`ENTRYPOINT`呢？这是比较适用于几个场景：  
- 场景一：让镜像变成向命令一样使用   
假设需要查询容器目录下的文件，可以使用`CMD`实现：
```
FROM ubuntu
CMD ["ls", "-l"]
```
然后使用`docker build -t test .`来构建镜像，若要查询镜像工作目录的所有文件，执行`docker run test`即可。
```
# docker run test
drwxr-xr-x   2 root root 4096 Mar 11 21:05 bin
drwxr-xr-x   2 root root 4096 Apr 24  2018 boot
...
```
这样镜像就可以当作命令使用了，但是当我们需要查看目录下所有的文件，包含隐藏文件的时候，就需要增加参数了，这个时候如果直接在`docker run`增加参数`-a`，则`-a`会将`CMD`替换，那么`-a`就不是一个能够正常执行的命令了。当然使用`docker run test ls -l -a`也是能解决这个问题的，但是显然这不是一个很好的解决方案。而这种情况`ENTRYPOINT`就能很好的解决这个问题了。
```
FROM ubuntu
ENTRYPOINT ["ls", "-l"]
```
然后构建镜像，执行`docker run`，和上面`CMD`实现的是一样的。但是需要增加参数时`docker run tets -a`即可。相当于整个命令就是：`ls -l -a`。

- 场景二：应用运行前的准备工作   
*启动容器就是启动主进程，但有些时候，启动主进程前，需要一些准备工作。*   

这些准备工作是和容器 `CMD` 无关的，无论 `CMD` 为什么，都需要事先进行一个预处理的工作。这种情况下，可以写一个脚本，然后放入 `ENTRYPOINT` 中去执行，而这个脚本会将接到的参数（也就是 `<CMD>`）作为命令，在脚本最后执行。

**注意：ENTRYPOINT 指定的脚本需要放在容器内部，否则执行时找不到脚本会报错。**
```
COPY docker-entrypoint.sh /usr/local/src
ENTRYPOINT ["/usr/local/src/docker-entrypoint.sh"]
```

# ENV 设置环境变量
格式：  
- `ENV <key> <value>`  设置单个环境变量的可以使用这种
- `ENV <key>=<value> <key>=<value>`  设置多个环境变量的时候可以使用这种

定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。这样可以使得Dockerfile的维护变得更简单。

# ARG 构建参数
格式：`ARG <参数名>[=<默认值>]`

构建参数和 `ENV` 的效果一样，都是设置环境变量。所不同的是，`ARG` 所设置的构建环境的环境变量，**在将来容器运行时是不会存在这些环境变量的**。但是不要因此就使用 ARG 保存密码之类的信息，因为 docker history 还是可以看到所有值的。

`Dockerfile` 中的 `ARG` 指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令 `docker build` 中用 `--build-arg <参数名>=<值>` 来覆盖。

# EXPOSE 声明端口
格式：`EXPOSE <端口1> [<端口2>...]`   

`EXPOSE` 指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是 `docker run -P` 时，会自动随机映射 `EXPOSE` 的端口。

要将 `EXPOSE` 和在运行时使用 `-p <宿主端口>:<容器端口>` 区分开来。-p，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 `EXPOSE` 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。

# VOLUME 定义匿名卷
格式：   
- `VOLUME <路径>`
- `VOLUME ["<路径1>", "<路径12>"...]`

容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中，后面的章节我们会进一步介绍 Docker 卷的概念。为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 Dockerfile 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。

```
VOLUME /data
```
这里的 `/data` 目录就会在运行时自动挂载为匿名卷，任何向 `/data` 中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。当然，运行时可以覆盖这个挂载设置。比如：`docker run -d -v mydata:/data xxxx`。在这行命令中，就使用了 `mydata` 这个命名卷挂载到了 `/data` 这个位置，替代了 Dockerfile 中定义的匿名卷的挂载配置。

**注意：`VOLUME` 指定的匿名卷，和 `docker run -v` 绑定的宿主机的目录是不一样的。其实 `VOLUME` 指令只是起到了声明了容器中的目录作为匿名卷，但是并没有将匿名卷绑定到宿主机指定目录的功能。但是当我们生成镜像的Dockerfile中以 `VOLUME` 声明了匿名卷，并且我们以这个镜像 `run` 了一个容器的时候，docker会在安装目录下的指定目录下面生成一个目录来绑定容器的匿名卷（这个指定目录不同版本的docker会有所不同）。** 比如18.09.0版本的docker，生成的半丁容器的匿名卷的目录是`/var/lib/docker/volumes`。

# USER 指定当前用户
格式：`USER <用户名>[:<用户组>]`   
`USER` 指令和 `WORKDIR` 相似，都是改变环境状态并影响以后的层。`WORKDIR` 是改变工作目录，`USER` 则是改变之后层的执行 `RUN`, `CMD` 以及 `ENTRYPOINT` 这类命令的身份。当然，和 `WORKDIR` 一样，USER 只是帮助你切换到指定用户而已，**这个用户必须是事先建立好的，否则无法切换**。
```
RUN groupadd -r redis && useradd -r -g redis redis
USER redis
RUN [ "redis-server" ]
```

如果以 `root` 执行的脚本，在执行期间希望改变身份，比如希望以某个已经建立好的用户来运行某个服务进程，不要使用 `su` 或者 `sudo`，这些都需要比较麻烦的配置，而且在 TTY 缺失的环境下经常出错。建议使用 `gosu`。
```
# 建立 redis 用户，并使用 gosu 换另一个用户执行命令
RUN groupadd -r redis && useradd -r -g redis redis
# 下载 gosu
RUN wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/1.7/gosu-amd64" \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true
# 设置 CMD，并以另外的用户执行
CMD [ "exec", "gosu", "redis", "redis-server" ]
```

# HEALTHCHECK 健康检查
格式：  
- `HEALTHCHECK [选项] CMD <命令>`: 设置检查容器健康状况的命令
- `HEALTHCHECK NONE`: 如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

**`HEALTHCHECK` 指令是告诉 Docker 应该如何进行判断容器的状态是否正常**，这是 Docker 1.12 引入的新指令。

在没有 `HEALTHCHECK` 指令前，Docker 引擎只可以通过容器内主进程是否退出来判断容器是否状态异常。很多情况下这没问题，但是如果程序进入死锁状态，或者死循环状态，应用进程并不退出，但是该容器已经无法提供服务了。在 1.12 以前，Docker 不会检测到容器的这种状态，从而不会重新调度，导致可能会有部分容器已经无法提供服务了却还在接受用户请求。

而自 1.12 之后，Docker 提供了 `HEALTHCHECK` 指令，通过该指令指定一行命令，用这行命令来判断容器主进程的服务状态是否还正常，从而比较真实的反应容器实际状态。

当在一个镜像指定了 `HEALTHCHECK` 指令后，用其启动容器，初始状态会为 **starting**，在 `HEALTHCHECK` 指令检查成功后变为 **healthy**，如果连续一定次数失败，则会变为 **unhealthy**。

`HEALTHCHECK` 支持下列选项：

- `--interval=<间隔>`: 两次健康检查的间隔，默认为 30 秒；
- `--timeout=<时长>`: 健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
- `--retries=<次数>`: 当连续失败指定次数后，则将容器状态视为 unhealthy，默认 3 次。

和 `CMD`, `ENTRYPOINT` 一样，**`HEALTHCHECK` 只可以出现一次，如果写了多个，只有最后一个生效**。

在 `HEALTHCHECK [选项] CMD` 后面的命令，格式和 `ENTRYPOINT` 一样，分为 `shell` 格式，和 `exec` 格式。命令的返回值决定了该次健康检查的成功与否：0：成功；1：失败；2：保留，不要使用这个值。

假设我们有个镜像是个最简单的 Web 服务，我们希望增加健康检查来判断其 Web 服务是否在正常工作，我们可以用 curl 来帮助判断，其 Dockerfile 的 `HEALTHCHECK` 可以这么写：
```
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
  CMD curl -fs http://localhost/ || exit 1
```
这里我们设置了每 5 秒检查一次（这里为了试验所以间隔非常短，实际应该相对较长），如果健康检查命令超过 3 秒没响应就视为失败，并且使用 `curl -fs http://localhost/ || exit 1` 作为健康检查命令。

使用 docker build 来构建这个镜像：`$ docker build -t myweb:v1 .`    
构建好了后，我们启动一个容器：`$ docker run -d --name web -p 80:80 myweb:v1`   
当运行该镜像后，可以通过 `docker container ls` 看到最初的状态为 `(health: starting)`：
```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                            PORTS               NAMES
03e28eb00bd0        myweb:v1            "nginx -g 'daemon off"   3 seconds ago       Up 2 seconds (health: starting)   80/tcp, 443/tcp     web
```
在等待几秒钟后，再次 `docker container ls`，就会看到健康状态变化为了 `(healthy)`：
```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS               NAMES
03e28eb00bd0        myweb:v1            "nginx -g 'daemon off"   18 seconds ago      Up 16 seconds (healthy)   80/tcp, 443/tcp     web
```
如果健康检查连续失败超过了重试次数，状态就会变为 `(unhealthy)`。

为了帮助排障，健康检查命令的输出（包括 stdout 以及 stderr）都会被存储于健康状态里，可以用 `docker inspect` 来查看。
```
$ docker inspect --format '{{json .State.Health}}' web | python -m json.tool
{
    "FailingStreak": 0,
    "Log": [
        {
            "End": "2016-11-25T14:35:37.940957051Z",
            "ExitCode": 0,
            "Output": "<!DOCTYPE html>\n<html>\n<head>\n<title>Welcome to nginx!</title>\n<style>\n    body {\n        width: 35em;\n        margin: 0 auto;\n        font-family: Tahoma, Verdana, Arial, sans-serif;\n    }\n</style>\n</head>\n<body>\n<h1>Welcome to nginx!</h1>\n<p>If you see this page, the nginx web server is successfully installed and\nworking. Further configuration is required.</p>\n\n<p>For online documentation and support please refer to\n<a href=\"http://nginx.org/\">nginx.org</a>.<br/>\nCommercial support is available at\n<a href=\"http://nginx.com/\">nginx.com</a>.</p>\n\n<p><em>Thank you for using nginx.</em></p>\n</body>\n</html>\n",
            "Start": "2016-11-25T14:35:37.780192565Z"
        }
    ],
    "Status": "healthy"
}
```

# ONBUILD 为他人做嫁衣裳
格式：`ONBUILD <其他指令>`   
ONBUILD 是一个特殊的指令，它后面跟的是其它指令，比如 RUN, COPY 等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行。


**参考文章**    
- Docker从入门到实践：[https://yeasy.gitbooks.io/docker_practice/content/](https://yeasy.gitbooks.io/docker_practice/content/)
- Dockerfile官方文档：[https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)
