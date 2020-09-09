---
title: docker
tags:
  - docker
  - 容器
categories:
  - docker
date: 2020-09-02 09:59:13
---



# 镜像

## 镜像加速器

linux下编辑`/etc/docker/daemon.json`, 写入如下内容(文件不存在时先创建). 其中, **阿里云镜像需要登录后在[镜像加速器](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)页面获取专属地址**

```json
{
    "registry-mirrors":[
        "https://xxxx.mirror.aliyuncs.com",
	    "https://mirror.baidubce.com",
    	"https://hub-mirror.c.163.com/",
	    "https://reg-mirror.qiniu.com/",
    ]
}
```

之后重启应用

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

验证加速器是否生效

```bash
docker info
```

如果看到如下内容则说明配置成功

```
...
 Registry Mirrors:
  https://xxxx.mirror.aliyuncs.com/
  https://mirror.baidubce.com/
  https://hub-mirror.c.163.com/
  https://reg-mirror.qiniu.com/
  https://registry.docker-cn.com/
....
```



## 获取镜像

```bash
docker pull centos:7
```

```
$ docker pull centos:7
7: Pulling from library/centos
75f829a71a1c: Pull complete
Digest: sha256:19a79828ca2e505eaee0ff38c2f3fd9901f4826737295157cc5212b7a372cd2b
Status: Downloaded newer image for centos:7
docker.io/library/centos:7
```



## 查看镜像列表

```
$ docker images
REPOSITORY          TAG           IMAGE ID            CREATED             SIZE
centos              7             7e6257c9f8d8        3 weeks ago         203MB
alpine              3.12.0        a24bb4013296        3 months ago        5.57MB
```

注意输出的镜像体积比Docker Hub上看到的要大, 这是因为Docker Hub上显示的压缩后的体积.

另外, 不同镜像可能会使用相同的基础镜像, 从而拥有共同的层, 所以**实际镜像的磁盘占用空间很可能比这个列表里的镜像大小总和要小的多.**

查看镜像, 容器, 数据卷所占空间

```bash
$ docker system df
TYPE             TOTAL      ACTIVE      SIZE        RECLAIMABLE
Images           2          1           208.9MB     203.3MB (97%)
Containers       1          0           36.03MB     36.03MB (100%)
Local Volumes    0          0           0B          0B
Build Cache      0          0           0B          0B
```



## 镜像列表过滤

```bash
# 列出java仓库中的所有镜像
docker images centos

# 指定仓库名和标签
docker images centos:7

# 使用过滤器参数, centos:7之后建立的镜像, since -> before则显示之前的
docker images -f since=centos:7
```



## 虚悬镜像

在镜像列表中, 有时会看到一个特殊的镜像, 这个镜像就没有仓库名, 也没有标签, 均为`<none>`

```
<none>        <none>        00285df0df87        5 days ago          342 MB
```

这个镜像原本是有镜像名和标签的(比如: `mongo:3.2`), 但随着官方镜像维护, 发布了新版本后, 重新 `docker pull mongo:3.2` 时，`mongo:3.2` 这个镜像名被转移到了新下载的镜像身上，而旧的镜像上的这个名称则被取消，从而成为了 `<none>`。除了 `docker pull` 可能导致这种情况，`docker build` 也同样可以导致这种现象。由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为 `<none>` 的镜像。这类无标签镜像也被称为 **虚悬镜像(dangling image)** ，可以用下面的命令专门显示这类镜像：

```bash
$ docker image ls -f dangling=true
REPOSITORY          TAG            IMAGE ID            CREATED             SIZE
<none>              <none>         00285df0df87        5 days ago          342 MB
```

一般来说，虚悬镜像已经失去了存在的价值，是可以随意删除的，可以用下面的命令删除

```bash
$ docker image prune
```



## 中间镜像

为了加快镜像构建, 重复利用资源, Docker会利用**中间镜像**, 默认的`docker images`只会显示顶层镜像, 如果想要显示包括中间镜像在内的所有镜像, 需要加`-a`参数:

```bash
 docker images -a
```

这样会看到很多无标签镜像, 与虚悬镜像不同, 这些镜像很多是中间镜像, 是其他镜像所依赖的镜像, 不应该被删除, 否则会导致上层镜像出错. 而上层镜像删除以后, 这些被依赖的中间镜像也会被连带着删除.



## 查看中间层

```bash
$ docker history centos:7
IMAGE        CREATED       CREATED BY                                      SIZE                COMMENT
7e6257c9f8   3 weeks ago   /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>    3 weeks ago   /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B
<missing>    3 weeks ago   /bin/sh -c #(nop) ADD file:61908381d3142ffba…   203MB
```

可以看到镜像的分层状况, 由镜像的Dockerfile中每个指令生成每个层



## 删除镜像

```bash
docker image rm 标识
# 更简单的命令
dokcer rmi 标识
```

其中 **标识** 可以是 **完整的镜像id**, **镜像id的前几位**(能唯一区分就行), **镜像名**, **镜像摘要**

```bash
$ docker images --digests
REPOSITORY   TAG      DIGEST                                                                    IMAGE ID            CREATED             SIZE
centos       7        sha256:19a79828ca2e505eaee0ff38c2f3fd9901f4826737295157cc5212b7a372cd2b   7e6257c9f8d8        3 weeks ago         203MB
alpine       3.12.0   sha256:185518070891758909c9f839cf4ca393ee977ac378609f700f60a771a2dfe321   a24bb4013296        3 months ago        5.57MB

# 使用长id
$ docker rmi 7e6257c9f8d8
# 使用短id
$ dokcer rmi a24
# 使用镜像名, 也就是 仓库名:标签
$ docker rmi centos:7
# 更精确的是使用镜像摘要删除  仓库@摘要
$ docker rmi centos@sha256:19a79828ca2e505eaee0ff38c2f3fd9901f4826737295157cc5212b7a372cd2b

Untagged: centos:7
Untagged: centos@sha256:19a79828ca2e505eaee0ff38c2f3fd9901f4826737295157cc5212b7a372cd2b
Deleted: sha256:7e6257c9f8d8d4cdff5e155f196d67150b871bbe8c02761026f803a704acb3e9
Deleted: sha256:613be09ab3c0860a5216936f412f09927947012f86bfa89b263dfa087a725f81
```

**镜像的唯一标识是其id和摘要,** 一个镜像可以有多个标签, 因此当我们删除了所指定的标签后, 可能还有别的标签指向了这个镜像, 此时会不会产生删除镜像的行为, 有可能仅仅是取消某个标签而已. 但是当该镜像的所有标签都被取消了, 就会触发删除行为. 并且是从上层到底层依次判断删除. 直到遇到被别的镜像依赖的某一层. 



## 批量删除

```bash
$ docker rmi $(docker images -q centos)
```



# Dockerfile

Dockerfile中每条指令都会新建一层, 而Docker 使用的 Union FS是有最大层数限制的,  比如 AUFS不得超过 127 层, 因此最好将多条指令合并成1条, 如:

```dockerfile
FROM debian:stretch

RUN set -x; buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

同时要注意, 在每一个指令的最后, 做好清理工作



## 构建

从标准输入中读取 Dockerfile 进行

```bash
docker build - < Dockerfile
cat Dockerfile | docker build -
```

用给定的 tar 压缩包构建

```bash
$ docker build http://server/context.tar.gz
$ docker build - < context.tar.gz
```



## 指令

### FROM 基础镜像

`FROM`必须是第一条指令, 用以指定**基础镜像**. 除了Docker Hub上的服务类和操作系统镜像外, 还有一个特殊的镜像, `scratch`, 这个镜像是虚拟的概念, 并不实际存在, 它表示一个空白的镜像.



### RUN 执行命令

`RUN`用来执行命令行命令的, 是最常用的指令之一, 有两种格式: 

- **shell格式** : `RUN <命令>`, `RUN yum install -y vim`
- **exec格式** : `RUN ["可执行文件", "参数1", "参数2"]`,  `RUN ["./test.php", "dev", "offline"]` 



### CMD 启动命令

`CMD`用于指定默认的容器主进程的启动命令的, **只有最后一条`CMD`命令会生效**, 比如我们可以配置

```bash
CMD java -jar xxx.jar
```

这样, 当容器启动后, 我们的java应用也就自动启动了

`CMD`和`RUN`一样, 有两种格式:

- **shell格式** : `RUN <命令>`
- **exec格式** : `RUN ["可执行文件", "参数1", "参数2"]`

在运行时可以指定新的命令来替代镜像设置中的这个默认命令, 比如, `ubuntu` 镜像默认的 `CMD` 是 `/bin/bash`, 如果我们直接 `docker run -it ubuntu` 的话, 会直接进入 `bash`. 我们也可以在运行时指定运行别的命令, 如 `docker run -it ubuntu cat /etc/os-release`, 这就是用 `cat /etc/os-release` 命令替换了默认的 `/bin/bash` 命令了, 输出了系统版本信息. 

一般**推荐`exec`格式**, 这类格式在解析时会被解析为 JSON 数组, 因此一定要使用双引号 `"`,而不要使用单引号.

如果使用 `shell` 格式的话, 实际的命令会被包装为 `sh -c` 的参数的形式进行执行, 比如:

```bash
CMD echo $HOME
```

在实际执行中, 会将其变更为:

```bash
CMD [ "sh", "-c", "echo $HOME" ]
```



**Docker 不是虚拟机，容器就是进程。** 容器中的应用都应该以前台执行, 而不是像虚拟机, 物理机里面那样, 用 `systemd` 去启动后台服务, 容器内没有后台服务的概念, 比如

```bash
CMD service nginx start
```

然后发现容器执行后就立即退出了, 甚至在容器内去使用 `systemctl` 命令结果却发现根本执行不了, 这就是因为没有搞明白前台/后台的概念, 没有区分容器和虚拟机的差异, 依旧在以传统虚拟机的角度去理解容器.

对于容器而言, 其启动程序就是容器应用进程, 容器就是为了主进程而存在的, 主进程退出, 容器就失去了存在的意义, 从而退出, 其它辅助进程不是它需要关心的东西.

而使用 `service nginx start` 命令, 则是希望 upstart 来以后台守护进程形式启动 `nginx` 服务. 而刚才说了 `CMD service nginx start` 会被理解为 `CMD [ "sh", "-c", "service nginx start"]`, 因此主进程实际上是 `sh`. 那么当 `service nginx start` 命令结束后, `sh` 也就结束了, `sh` 作为主进程退出了, 自然就会令容器退出. 

正确的做法是直接执行 `nginx` 可执行文件, 并且要求以前台形式运行. 比如:

```bash
CMD ["nginx", "-g", "daemon off;"]
```



### ENTRYPOINT 启动入口

`ENTRYPOINT` 的目的和 `CMD` 一样, 都是在指定容器启动程序及参数, 也分为 `exec` 格式和 `shell` 格式. `ENTRYPOINT` 在运行时也可以替代, 不过比 `CMD` 要略显繁琐, 需要通过 `docker run` 的参数 `--entrypoint` 来指定.

当指定了 `ENTRYPOINT` 后, `CMD` 的含义就发生了改变, 不再是直接的运行其命令, 而是将 `CMD` 的内容作为参数传给 `ENTRYPOINT` 指令, 换句话说实际执行时, 将变为:

```bash
<ENTRYPOINT> "<CMD>"
```

那么有了 `CMD` 后, 为什么还要有 `ENTRYPOINT` 呢? 启动容器就是启动主进程, 但有些时候, 启动主进程前, 需要一些准备工作, 比如`mysql`数据库, 可能需要做一些数据库配置, 初始化的工作, 而且使用root身份去执行, 但最后切换到服务用户是否去启动服务. 这些准备工作和容器的`CMD`无关, 这时, 可以写一个脚本, 然后放入`ENTRYPOINT`中去执行, 而这个脚本会将接受到的参数(也就是`<CMD>`)作为命令, 在脚本最后执行. 比如官方镜像`redis`中就是这么做的:

```dockerfile
FROM debian:buster-slim
...
RUN groupadd -r -g 999 redis && useradd -r -g redis -u 999 redis
...
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 6379
CMD ["redis-server"]
```

可以看到其中为了 redis 服务创建了 redis 用户, 并在最后指定了 `ENTRYPOINT` 为 `docker-entrypoint.sh` 脚本

```sh
#!/bin/sh
set -e
...
# allow the container to be started with `--user`
if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
	find . \! -user redis -exec chown redis '{}' +
	exec gosu redis "$0" "$@"
fi

exec "$@"
```

该脚本内容就是根据`CMD`的内容来判断, 如果是`redis-server`的话, 则切换到`redis`用户是否启动服务, 否则使用`root`身份执行, 比如:

```bash
$ docker run -it redis id
uid=0(root) gid=0(root) groups=0(root)
```



### ENV 环境变量

该指令用于设置环境变量, 无论是后面的其他指令(如`RUN`)或者是运行中的应用, 都可以使用这里定义的环境变量. 该指令有两种格式:

- ENV key value
- ENV key1=value1 key2=value2 ...

```dockerfile
ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"
    
RUN xxx.sh -v $VERSION -n $NAME    
```



### ARG 构建参数

和`ENV`的效果一样, `ARG`也是用来设置环境变量的. 不同的是, `ARG`设置的是**构建环境**的环境变量, 在将来容器运行时时不会存在这些环境变量的. 但不要因此就使用`ARG`保存敏感信息, 因为`docker history`还是可以看到所有值的.

`Dockerfile`中的`ARG`指令定义参数名称以及默认值, 但在构建目录`docker build`中可以用`--build-arg key1=value`来覆盖.



### COPY 文件复制

该指令用于将构建上下文目录中`源路径`的文件/目录复制到新的一层镜像内的`目的路径`位置, 比如:

```dockerfile
COPY package.json /usr/src/app

COPY application-*.yml /mydir/
COPY hom?.txt /mydir/

COPY --chown=nginx:nggroup files* /mydir/
COPY --chown=redis files* /mydir/
```

`<目标路径>` 可以是容器内的绝对路径, 也可以是相对于工作目录的相对路径(工作目录可以用 `WORKDIR` 指令来指定), **目标路径不需要事先创建**, 如果目录不存在会在复制文件前先行创建缺失目录.

如果源路径为文件夹, 复制的时候不是直接复制该文件夹, 而是将文件夹中的内容复制到目标路径.



### ADD 高级文件复制

`ADD`指令格式和`COPY`的格式和性质基本一致, 但在`COPY`基础上多了一些高级功能.

比如`源路径`是个tar文件, 压缩格式为 `gzip`, `bzip2` 以及 `xz` 的情况下, `ADD`指令会自动解压这个文件到`目的路径`.

在Docker官方的 [Dockerfile最佳实践文档](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)中, 推荐**尽量使用`COPY`**, 因为其语义很明确. **在需要文件复制时均使用`COPY`, 仅在需要自动解压缩是使用`ADD`.**



### VOLUME 匿名卷

该指令用于将目录挂载为匿名卷, 比如`redis`容器运行时的数据文件(如:`/data`)需要保持到容器外的宿主机上, 这样容器运行时, 任何向`/data`写入的信息都不会记录进容器存储层, 而是保存到了宿主机上, 从而保证了容器存储层的无状态化.  当然, 运行时可以覆盖这个挂载设置.

正常情况下, 启动容器时会将宿主机某个目录挂载到容器的`/data`, 但是为了保证在运行时用户不指定挂载, 应用也可以正常运行, 在`Dockerfile`里可以实现指定将目录挂在为匿名卷. 此后每次运行容器, 如果不指定挂载, 宿主机则会创建类似`/var/lib/docker/volumes/af45a2190a85ee4919cde3cfa010314afe9ac73a53e64d6841d8ecdaa1de866b`目录, 然后挂载到容器里的`/data/mysql`目录. 

如果在运行时指定挂载, 如:

```bash
# 挂载数据卷
$ docker run --name redis6 -v my-volume:/data -P -d redis:6 redis-server
$ docker run --name redis6 --mount source=my-volume,target=/data -P -d redis:6 redis-server

# 挂载主机目录
$ docker run --name redis6 -v /iflytek/data/redis-docker/:/data -P -d redis:6 redis-server
$ docker run --name redis6 \
	--mount type=bind,source=/iflytek/data/redis-docker/,target=/data \
    -P -d redis:6 redis-server

# 主机目录为只读 readonly
$ docker run -d -P \
    --name web \
    # -v /src/webapp:/usr/share/nginx/html:ro \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html,readonly \
    nginx:alpine

```

则会在宿主机上创建`/var/lib/docker/volumes/my-volume`目录, 挂载到容器的`/data`目录上

```bash
# 查看容器新
$ docker inspect redis6
...
"Mounts": [
    {
        "Type": "bind",
        "Source": "/iflytek/data/redis-docker",
        "Destination": "/data",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    }
]
...

# 删除容器时也删除卷
$ docker rm -v <container_id>
# 清理无主卷
$ docker volume prune
```



### EXPOSE 暴露端口

`EXPOSE`指令是声明运行时容器提供服务端口, 这只是一个声明, 在运行时并不会因为这个声明应用就会开启这个端口的服务. 在 Dockerfile 中写入这样的声明有两个好处, 一个是帮助镜像使用者理解这个镜像服务的守护端口, 以方便配置映射; 另一个用处则是在运行时使用随机端口映射时, 也就是每次 `docker run -P` 时, 会自动随机将宿主机的某个端口映射到 `EXPOSE` 的端口上, 但是每次的主机端口都不一样.

要将 `EXPOSE` 和在运行时使用 `-p <宿主端口>:<容器端口>` 区分开来. `-p`是映射宿主端口和容器端口, 换句话说, 就是将容器的对应端口服务公开给外界访问, 而 `EXPOSE` 仅仅是声明容器打算使用什么端口而已, 并不会自动在宿主进行端口映射.

```bash
# 查看容器端口映射信息
[root@jzcpx-no ~]# docker port redis6
6379/tcp -> 0.0.0.0:32778
```



### WORKDIR 指定工作目录

`WORKDIR`指令用来指定工作目录(或称为当前目录), 以后各层的当前目录就被改为指定的目录, 如果该目录不存在, `WORKDIR`会帮你创建. 

不要把`Dockerfile`当shell脚本来写

```dockerfile
RUN cd /app
RUN echo "hello" > world.txt
```

如果使用上面的`Dockerfile`来构建镜像, 会发现找不到`/app/world.txt`文件. 原因是, 在shell中, 连续的命令是在同一个进程执行环境, 因此前一个命令会直接影响下一个. 而在`Dockerfile`中, 这两行命令的执行环境是两个完全不同的容器. 这就是对`Dockerfile`构造分层存储的概念不清导致的.

因此如果需要改变以后各层的工作目录位置, 应该使用`WORKDIR`指令.



### USER 指定当前用户

`USER`指令和`WORKDIR`指令类型, 它用来改变之后各层的执行`RUN`, `CMD` 以及 `ENTRYPOINT`这类命令的身份. **这个用户必须是实现建立好的, 否则无法切换**, `USER`只是帮助你切换到指定用户而已.

```dockerfile
RUN groupadd -r redis && useradd -r -g redis redis
USER redis
RUN [ "redis-server" ]
```



### HEALTHCHECK

`HEALTHCHECK`指令用来设置检查容器健康状况的命令, 有两种格式

- `HEALTHCHECK [选项] CMD <命令>` 设置容器健康状态检查的命令, 命令支持`shell`和`exec`格式, 返回值`0`:成功 ; `1`:失败; `2`:保留
- `HEALTHCHECK NONE` 如果基础镜像有健康检查指令, 用这行可以屏蔽掉

该指令是Docker 1.12引入的, 在此之前, Docker引擎只能通过容器内主进程是否退出来判断容器是否状态异常. 但如果程序进入死锁状态, 或者死循环, 应用进程并不退出, 但是容器已经无法提供服务了. 通过`HEALTHCHECK`指令则能比较真实的反应容器的实际状态.

容器初始状态为`starting`, 在`HEALTHCHECK`指令检查成功以后变为`healthy`, 连续一定次数失败后变为`unhealthy`

`HEALTHCHECK`支持以下选项:

- `--interval=<间隔>` 两次健康检查的间隔, 默认为30秒
- `--timeout=<时长>` 健康检查命令运行的超时时间, 超过该时间认为失败, 默认为30秒
- `--retries=<次数>` 当连续失败指定次数后, 容器被认为`unhealthy`, 默认3次

```dockerfile
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --intervals=5s --timeout=3s CMD curl -fs http://localhost:8080/ || exit 1
```



### ONBUILD

`ONBUILD` 是一个特殊的指令, 它后面跟的是其它指令, 比如 `RUN`, `COPY` 等, 而这些指令, 在当前镜像构建时并不会被执行. 只有当以当前镜像为基础镜像, 去构建下一级镜像的时候才会被执行.



```bash
$ docker run -it --rm centos:7 bash
$ docker run --name webserver -d -p 80:80 nginx
$ docker exec -it webserver bash

# 修改容器内文件后, 将容器保存为镜像, 该镜像称为"黑箱镜像", 很多其他文件被后台修改了, 慎用
$ docker commit --author "CoderTang <mails100@163.com>" \\
  --message "some files modified" \ 
  webserver \
  nginx:v2
  sha256:07e33465974800ce65751acc279adc6ed2dc5ed4e0838f8b86f0c87aa1795214
  
$ docker run --name web2 -d -p 81:80 nginx:v2  
```

- **-i** : 交互
- **-t** : 终端
- **--rm** : 容器退出后随之将其删除



# 容器管理

## 启动

```bash
# 启动容器, 输出Hello World, 容器终止
$ docker run ubuntu:18.04 /bin/echo 'Hello world'
Hello world

# 启动bash终端, 运行用户进行交互 -t:分配一个为终端并绑定到容器的标准输入上, -i:让容器标准输入保持打开
$ docker run -it ubuntu:18.04 /bin/bash
root@af8bae53bdd3:/#
```

当执行`docker run`时, Docker在后台执行了如下操作: 

- 检查本地是否存在指定的镜像，不存在就从公有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

 

## 启动已终止容器

```bash
docker container start ID/NAME
```



## 守护态运行

更多时候, 需要让容器在后台运行, 而不是直接把执行命令的结果输出在当前宿主机上, 此时可以使用`-d`参数来实现

```bash
# 如果不使用-d, 则容器会把输出结果打印到宿主机上
$ docker run ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
hello world
hello world
hello world
hello world

# 如果使用了-d参数则不会, 此时可以使用 docker logs 查看输出结果
$ docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
77b2dc01fe0f3f1265df143181e7b9af5e05279a884f4776ee75350ea9d8017a
```



## 终止

```bash
docker stop ID/NAME
```



## 重启

```bash
docker restart ID/NAME
```



## 进入容器

```bash
docker exec -it ID/NAME bash
```



## 导出导入

将容器快照导出到本地

```bash
docker export 1eef > daocloud.tar
```

导入容器快照

```bash
cat daocloud.tar | docker import - daocloud.io/tonnyyi/cicd:master-f708ed1

# 也可以通过URL或某个目录导入
docker import http://example.com/exampleimage.tgz example/imagerepo
```

用户既可以使用 `docker load` 来导入镜像存储文件到本地镜像库, 也可以使用 `docker import` 来导入一个容器快照到本地镜像库. 这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息(即仅保存容器当时的快照状态), 而镜像存储文件将保存完整记录, 体积也要大. 此外, 从容器快照文件导入时可以重新指定标签等元数据信息



## 删除

```bash
# 删除一个已终止的容器
docker rm 1eef

# 删除一个运行中的容器 -f, Docker会发送 SIGKILL 信号给容器
docker rm -rf 1eef
```



## 容器网络

```bash
# 查看网络
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
90cb8b27210e        bridge              bridge              local
d90e2e43a75e        host                host                local
ca97d4ab0ef4        none                null                local
```



## 设置固定ip

Docke安装以后, 默认会创建下面三种网络

```bash
[root@jzcpx-no ~]# docker network  ls
NETWORK ID          NAME                DRIVER              SCOPE
90cb8b27210e        bridge              bridge              local
d90e2e43a75e        host                host                local
ca97d4ab0ef4        none                null                local
```

默认的网络是不支持指派固定ip的

```bash
[root@jzcpx-no ~]# docker run --name redis6 -v /iflytek/data/redis-docker/:/data --ip 172.17.0.10 -P -d redis:6 redis-server
f64716ac5277232f02a52679ad312159d878d215be00f03a4778e0065d147918
/usr/bin/docker-current: Error response from daemon: User specified IP address is supported on user defined networks only.
```

因此, 需要创建自定义网络

```bash
[root@jzcpx-no ~]# docker network create --subnet=172.18.0.0/16 mynetwork
c66eeabaa678b9c87a6a1222872a75b6f8072308c1f2d8c7a9d347e9e39d5cdb

[root@jzcpx-no ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
90cb8b27210e        bridge              bridge              local
d90e2e43a75e        host                host                local
c66eeabaa678        mynetwork           bridge              local
ca97d4ab0ef4        none                null                local

[root@jzcpx-no ~]# docker run --name redis6 -v /iflytek/data/redis-docker/:/data --network mynetwork --ip 172.18.0.10 -P -d redis:6 redis-server
8de37f0d12a4017a8a7f7e0c0649e8152690afb7525f383ca506be6f8324f844
```

查看网络信息

```bash
[root@jzcpx-no ~]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "90cb8b27210eb0092045eb6d0f8fbfc5dc8493855319c2287c47713a3b741360",
        "Created": "2020-06-16T16:21:27.080634077+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "1eef326e9e2c8b3b9fd9cd4a50f1275b1dc9eb4f2404d937f1b0af30865d4c40": {
                "Name": "dao_aaaa_1",
                "EndpointID": "e85c2aa0e4646ead82fa62fdd276f3a3224e4e3ec37c47bb5e391e15780e256d",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```





# alpine

```dockerfile
FROM alpine:3.12.0

MAINTAINER codertang.com 
#更新Alpine的软件源为国内（清华大学）的站点，因为从默认官源拉取实在太慢了。。。
RUN echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.4/main/" > /etc/apk/repositories

RUN apk update \
        && apk upgrade \
        && apk add --no-cache bash \
        bash-doc \
        bash-completion \
        vim \
        && rm -rf /var/cache/apk/* \
        && /bin/bash
```

