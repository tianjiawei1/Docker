# Docker

## 一、Docker的基本组成

Docker是基于Go语言实现的云开源项目。理念”Build, Ship and Run Any app,Anywhere“,也就是通过对应用组件的封装，分发，部署，运行等生命周期的管理，使用户的APP（可以使一个Web应用或数据酷应用等等）及其运行环境能够做到”一次封装，到处运行“。

解决了运行环境和配置问题，方便做持续集成并有助于整体发布的容器虚拟化技术。

- 更快速的应用交互和部署
- 更便捷的升级和扩缩容
- 更简单的系统运维
- 更高效的计算资源利用

### Why Docker?

- 更轻量：基于容器的虚拟化，仅包含业务运行所需的runtime环境，CentOS/Ubuntu基础镜像仅170M；宿主机可部署100~1000个容器

- 更高效：无操作系统虚拟化开销
  - 计算：轻量，无额外开销
  - 存储：系统盘 aufs/dm/overlayfs；数据盘volume
  - 网络：宿主机网络，NS隔离
- 更敏捷，更灵活：
  - 分层的存储和包管理，devops理念
  - 支持多种网络配置

### 官网

1，docker官网：http://www.docker.com

2,   docker中文网站：https://www.docker-cn.com

### Docker三要素

> 镜像（image）

就是一个只读的模板。镜像可以用来穿件Docker容器，**一个镜像可以穿件很多容器**。

容器与镜像的关系类似于面向对象编程中的对象与类。

> 容器（container）

独立运行的一个或一组应用，容易是用镜像创建的运行实例。

它可以被启动，开始，停止，删除。每个容器都是相互隔离的，保证安全的平台。

*可以把容器看做是一个简易版的linux*（包括root用户权限，进程空间，用户空间和网格空间等）和运行在其中的运行程序。

容器的定义和镜像几乎一模一样，也是一堆层的统一视角，唯一区别在于容器的最上面那一层是可读可写的。

> 仓库(Repository)

是集中存放镜像文件的场所。

仓库(Repository)和仓库注册服务器（Registry）是有区别的。仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像又不同的标签（tag）。

仓库分为公开仓库（Public）和私有仓库（Private）两种形式，存放了数量庞大的镜像供用户下载。国内的公开仓库包括阿里云、网易云等。

**总结**

Docker本身是一个容器运行载体或称之为管理引擎。我们把应用程序和配置依赖打包好的运行环境就类似image镜像文件。只有通过这个镜像文件才能生成多个Docker容器。image文件可以看作是容器的模板。Docker根据image文件生成容器的实例。同一个image文件，可以生成多个同时运行的容器实例。

- image文件生成的容器实例，本身也是一个文件，称之为镜像文件。
- 一个容器运行一种服务，当我们需要的时候，就可以通过docker客户端创建一个对应的运行实例，也就是我们的容器
- 至于仓库，就是放了一堆镜像的地方，我们可以把整个镜像发步到仓库中，需要的时候从仓库中拉下来就可以了。

## 二、Docker安装

**前提条件**

目前，CentOS仅发现版本中的内核支撑Docker。

Docker运行在CentOS上，要求系统为64位，系统内核版本位3.10以上。

#### 1.CentOS 6.8安装Docker

1. yum install -y epel-release
2. yum install -y docker-io
3. 安装后的配置文件：/ect/sysconfig/docker
4. 启动Docker后台服务：service docker start
5. docker version验证

#### 2.CentOS 7安装Docker

https://docs.docker.com/engine/install/centos/

### 镜像加速

1.阿里云镜像加速

https:/dev.aliyun.com/search.html

获得加速器地址连接：登录阿里云开发者平台；获取加速器地址

配置本机Docker运行镜像加速器

重新启动Docker后台服务：service docker restart

Linux系统下配置完加速器需要检查是否生效

> ps -ef|grep docker(查看是否配有加速地址)

### Docker是怎么工作的

Docker是一个Client-service结构的系统，Docker守护进程运行在主机上，然后通过Socket连接从客户端访问，守护进程从客户端接收命令并管理运行在主机上的容器。**容器，是一个运行时环境，就是我们前面说的集装箱。**

### 为什么Docker比VM快

1. docker有着比虚拟机更少的抽象层。由于docker不需要Hypervisor实现硬件字眼虚拟化，运行在docker容器上的程序直接使用的实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有明显优势。
2. docker利用的是宿主机的内核，而不需要Guest OS。因此，当新建一个容器是，docker不需要和虚拟机一样重新加载一个操作系统内核。因而避免引寻、加载操作系统内核返回比较费时非自愿的过程，当新建一个虚拟机时，虚拟机软件需要加载Guest OS，返回新建过程是分钟级别的。而docker由于直接利用宿主机的操作系统，则省略了返回过程，因此新建一个docker容器只需要几秒钟。 

![](D:\软件\Typora\笔记\image\图片35.png)

## 三、Docker常用命令

### 1，帮组命令

> docker version  查看版本信息
>
> docker info      详细信息
>
> docker --help

### 2，镜像命令

#### docker images 

列出本地主机上的镜像

> REPOSITORY：表示镜像的仓库源
>
> TAG：镜像的标签
>
> IMAGE ID：镜像ID
>
> CREATED：镜像创建时间
>
> SIZE：镜像大小

同一个仓库源可以有多少个TAG，代表这个仓库源的不同的版本，我们使用REPOSITORY：TAG 来定义不同的镜像。

如果不指定一个镜像的版本标签，例如你只使用ubuntu，docker将默认使用ubuntu:latest镜像。

OPTIONS说明：

docker image -a

> -a ：列出本地所有镜像（含中间映像层）
>
> -q：只显示镜像ID  (docker images -qa )
>
> --digests：显示镜像的摘要信息
>
> --no-trunc：显示完整的镜像信息

#### docker seacher 某个xxx镜像名字

网站： https://hub.docker.com

> docker search tomcat
>
> docker search -s 30 tomcat  ：fork超过30的版本镜像

命令

> docker search [OPTIONS] 镜像名字

OPTIONS说明

> --no-trunc：显示完整的镜像描述
>
> -s：列出收藏数不小于指定值的镜像
>
> --automated：只列出automated build类型的镜像

#### docker pull  某个xxx镜像名字

下载镜像

docker pull 镜像名字[:TAG]

> docker pull tomcat:latest

#### docker rimi 某个xxx镜像名字ID

删除镜像

删除单个： docker rmi -f  镜像ID

> docker  rmi  hello-world:latest
>
> docker rim  -f  hello-world ：强制删除（干掉容器运行）

删除多个：docker rmi -f 镜像名:TAG 镜像名2:TAG

>  docker rmi -f hello-world nginx

删除全部：docker rmi -f ${docker images -qa}

>  docker rmi -f  $(docker images -q)

### 3，容器常用命令

- 有镜像才能创建容器，这是根本前提（下载一个CentOS镜像演示）

docker上跑CentOS  (docker pull centos)

#### 3.1新建并启动容器

>   docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

##### OPTIONS说明（常用）

有些是一个减号，有些是两个减号

> --name="容器新名字" ：为容器指定一个名称
>
> -d：后台运行容器，并返回容器ID，也即启动守护式容器；
>
> **-i：以交互模式运行容器，通常与-t同时使用；**
>
> **-t：为容器重新分配一个伪输入终端，通常与-i同时使用；**
>
> -P：随机端口映射；
>
> -p：指定端口映射，有以下四种格式
>
> ​      ip:hostport:containerPort
>
> ​      ip::containerPort
>
> ​       hostport:containerPort
>
> ​       containerPort
>
> docker   run  -it  -- name mycentos   centos

##### 启动交互式容器

使用镜像**centos：latest**以交互模式启动一个容器，在容器内执行**/bin/bash**命令。

> docker run -it centos /bin/bash

#### 3.2列出当前所有正在运行的容器

> docker  ps [OPTIONS]

##### OPTIONS说明（常用）

> -a：列出当前所有**正在运行**的容器+**历史上运行过**的
>
> -l：显示最近创建的容器
>
> -n：显示最近n个创建的容器
>
> **-q：静默模式，只显示容器编号**
>
> --no-trunc：不截断输出。

#### 3.3退出容器

- exit  ：容器停止退出
- ctrl+P+Q ：容器不停止退出

#### 3.4启动容器

> docker start  容器ID或者容器名

#### 3.5重启容器

> docker restart 容器ID或者容器名

#### 3.6停止容器

> docker stop 容器ID或者容器名

#### 3.7强制停止容器

> docker  kill 容器ID或者容器名

#### 3.8删除已停止的容器

> docker  rm 容器ID
>
> 一次性删除多个容器:
>
> - docker rm -f  ${docker ps  -a -q}
> - docker ps -a -q|xargs docker rm

#### 3.9**重要**

##### 3.9.1启动守护式容器

> docker  run  -d 容器名
>
> 如：docker run -d centos

问题：然后*docker ps -a* 进行查看，会发现容器已经退出，很重要的要说明一点：**Docker容器后台运行，就必须有一个前台进程**

容器运行的命令如果不是那些**一直挂起的命令**（比如运行top,tail），就是会自动退出的。

这个是dock二人的机智问题，比如你的web容器，我们以nginx为例，正常情况下，我们配置启动服务只需要启动相应的service即可。

例如：service nginx start

但是，这样做，nginx为后台进程模式运行，就导致docker前台没有运行的应用，这样的容器后台启动后，会立即自杀因为它觉得它没事可做了。所以，最佳的解决方案是，将你要运行的程序以前台进程的形式运行。

##### 3.9.2查看容器日志

> docker log -f  -t --tail 容器ID
>
> - -t：是加入时间戳
> - -f：跟随最新的日志打印
> - --tail  数字：显示最后多少条

例：

> docker run  -d  centos /bin/sh -c "while true;do echo hello zzyy;sleep 2;done"
>
> docker ps
>
> docker logs 容器ID

##### 3.9.3查看容器内运行的进程

> docker top 容器ID

##### 3.9.4查看容器内部细节

> docker inspect 容器ID

##### 3.9.5进入正在运行的容器并以命令行交互

> ：docker exec -it 容器ID bashShell
>
> 重新进入：docker attach 容器ID

上述两个区别：

> attach ：直接进入容器启动命令的终端，不会启动新的进程
>
> exec：是在容器中打开新的终端，并且可以启动新的进程

![](D:\软件\Typora\笔记\image\图片36.png)

##### 3.9.6从容器内拷贝文件到主机上

> docker cp 容器ID：容器内路径 目的主机路径

如：docker cp f5fa58c8c1c4:/usr/local/mycptest/container.txt /tmp/c.txt

#### 小总结

![](D:\软件\Typora\笔记\image\图片37.png)

## 四、Docker镜像

### 是什么？

镜像是一种轻量级、可执行的独立软件包，**用来打包软件运行环境和基于运行环境开发的软件**，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。

- UnionFS(联合文件系统)

  > UnionFS(联合文件系统)：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下（unite serveral directories into a single virtual filesystem）。Union文件系统是Docker镜像的基础。镜像可以通过分层来进行基层，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。
  >
  > 特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件

- Docker镜像加载原理

  > docker的镜像实际上由一层一层的文件组件，这种层级的文件系统UnionFS。
  >
  > bootfs（boot file system）主要包含BootLoader和kernel，BootLoader主要是引导加载kernel，Linux刚启动时会加载bootfs文件系统，**在Docker镜像的最底层是bootfs。**这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。
  >
  > rootfs（root file system），在bootfs之上，包含的就是典型的Linux系统中的/dev，/proc，/bin，/etc等标准目录和文件。rootfs就是各种不同额操作系统化发行版本，比如Ubuntu，Centos等等。

  *平时我们安装进虚拟机的CentOS都是好几个G，为什么docker这里才200M？*

  对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供rootfs就行了。由此可见对于不同的Linux发行版，bootfs基本是一致的，rootfs会有差别，因此不同的发行版可以共用bootfs。

- 分层的镜像

以我们pull为例，在下载的过程中我们可以看到docker的镜像好像是在一层一层的在下载。

![](D:\软件\Typora\笔记\image\图片38.png)

- 为什么Docker镜像要采用这种分层的结构呢

> 最大的一个好处就是-共享资源
>
> 比如：有多个镜像都从相同的base镜像构建而来，那么宿主机只需要在磁盘上保存一份base镜像，同时内存中也只需要加载一份base镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。

### 特点

> Docker镜像都是只读的。
>
> 当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作为“容器层“，”容器层“之下的都叫“镜像层”。

### Docker镜像commit操作补充

> docker commit提交容器副本使之成为一个新的镜像
>
> docker commit -m=“提交的描述信息” -a="作者" 容器ID 要创建的目标镜像名：[标签名]

案例演示

1.从Hub上下载Tomcat镜像到本地并成功运行

> docker run -it -p 8080:8080 tomcat
>
> -p 主机端口：docker容器端口
>
> -P 随机分配端口 | docker  run -it -P tomcat
>
> i：交互
>
> t：终端

2.故意删除上异步镜像生产Tomcat容器的文档

3.即当前的Tomcat运行实例是一个没有文档内容的容器，以它为模板commit一个没有doc的Tomcat新镜像 atguigu/tomcat02

> docker commit -a="zzyy" -m="tomcat without docs" 9fa4193e6e9a atguigu/mytomcat:1.2

4.启动新的镜像并和原来的对比

## 五、Docker容器数据卷

### 是什么

> 一句话：有点类似我们Redis里面的rdb和aof文件

- 将运用与运行的环境打包形成容器运行，运行可以伴随着容器，但是我们队数据的要求希望是持久化的
- 容器之间希望有可能 共享数据

Docker容器产生的数据，如果不通过 docker commit生成新的镜像，使得数据作为镜像的一部分保存下来，那么当容器删除后，数据自然也就没有了。

为了能保存数据在docker中我们使用卷。

### 能干嘛

> 容器的持久化
>
> 容器间继承+共享数据

卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性

卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷

特点：

1. 数据卷可以在容器之间共享或重用数据
2. 卷中的更改可以直接生效
3. 数据卷中的更改不会包含在镜像的更新中
4. 数据卷的生命周期一直持续到没有容器使用它为止

### 数据卷

### 容器内添加

#### 直接命令添加

1. 命令

> docker run -it -v /宿主机绝对路径目录:/容器内目录  镜像名
>
> 如：docker run -it  -v   /myDataVolume:/dataVolumeContainer centos

2. 查看数据卷是否挂载成功

3. 容器和宿主机之间数据共享

4. 容器停止退出后，主机修改后数据是否同步

5. 命令（带权限）

   > docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名

#### DockerFile添加

#### 备注

### 数据卷容器

## 六、DockerFile解析

## 七、Docker常用安装

## 八、本地镜像发布到阿里云


