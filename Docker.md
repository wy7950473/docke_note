```
docker： 解决了运行环境和配置问题软件容器，方便做持续集成并有利于整体发布的容器虚拟化技术。
虚拟机的缺点: 资源占用多、冗余步骤多、启动慢
DveOps的好处：
	更快速的应用交付和部署
	更便捷的升级和扩容
	更简单的系统运维
	更高效的计算资源利用
```

docker三要素:

```
Docker本身是一个容器运行载体或称之为管理引擎。我们把应用程序和配置依赖打包好形成一个可交付的运行环境，这个打包好的运行环境似乎image镜像文件，只有通过这个镜像文件才能生成Docker容器。image文件可以看作是容器的模板。Docker根据image文件生成容器实例。同一个image文件,可以生成多个同时运行的容器实例。

镜像: 
	就是一个只读的模板。镜像可以用来创建Docker容器，一个镜像可以创建很多容器(相当于Java中定义的一个类)
容器：
	docker利用容器独立运行的一个或一组应用，容器是用镜像创建的运行实例(相当于Java中一个类的实例化对象)，可以把容器看做是一个简易版的Linux环境和运行在其中的应用程序。容器的定义和镜像几乎一模一样，也是一堆层的统一视角，唯一区别在于容器的最上面那一层是可读可写的。
仓库:
	是存放镜像文件的场所，分为公开库(public)和私有库(private),最大的公开仓库是Docker Hub(https://hub.docker.com/)
```

安装

``` 
启动docker：systemctl start docker
停止docker: systemctl stop docker

vim /etc/docker/daemon.json
systemctl daemon-reload
systemctl restart docker
```

底层原理

```
怎么工作:
	Docker是一个Client-Server结构的系统，Docker守护进程运行在主机上，然后通过Socket连接从客户端访问，
	守护进程从客户端受命令并管理运行在主机上的容器，容器，是一个运行时环境，就是前面我们说到的集装箱。

docker为什么比VM快:
	1、docker有着比虚拟机更少的抽象层，由于docker不需要Hypervisor实现硬件资源虚拟化，运行docker容器上的程序直接使用的都是实际物理机的硬件资源。因此CPU、内存利用率上docker将会在效率上有明显优势。
	2、docker使用的宿主机的内核，而不需要Guest OS。因此当新建一个容器时，docker不需要和虚拟机一样重新加载一个操作系统内核。仍而避免引寻、加载操作系统内核这个比较浪费资源的过程，当新建一个虚拟机时，虚拟机软件需要加载Guest OS，返个新建进程是分钟级别的。而docker由于直接利用宿主机的操作系统，则省略了这个过程，因此新建一个docker容器只需要几秒钟。
```

命令

```
帮助命令:
	查看版本: docker version
	查看docker详细信息: docker info
	帮助命令: docker --help

镜像命令:
	列出本机上的镜像：docker images [option]
		-a:列出本地所有镜像(含中间映像层)
		-q:只显示镜像信息
		--digests:显示镜像完整的摘要信息
		--no-trunc:显示完整的镜信息
	查看指定名称的镜像: docker search [option] <name>
		-s:列出收藏数小于指定值的镜像
		--no-trunc:显示完整的镜像信息
		--automated:只列出automated build类型的镜像
	下载镜像:docker pull 镜像名称:[Tags]
	删除镜像: docker rmi [option] <name>
		-f:强制删除
		
容器命令:
	创建并启动容器: docker run [option] image [command] [tag....]
		--name "容器的名字": 为容器指定一个名称
		-d:后台运行容器，并返回容器ID,也即启动守护式容器
		-i:以交互模式运行容器，通常与-t同时使用
		-t:为容器重新分配一个伪输入终端，通常与-i同时使用
		-P:随机端口映射
		-p:指定端口映射，有以下四种格式
			ip:hostPort:containerPort
			ip::containerPort
			hostPort(主机端口):containrPort(容器端 口)
			containerPort
	列出当前正在运行的容器:docker ps [option]
		-a:列出当前所有正在运行的容器+历史上运行过的
		-l:显示最近创建的容器
		-n:显示最近n个创建的容器
		-q:静默模式，只显示容器的编号
		--no-trunc:不截断输出
	退出容器: 
		exit:容器停止退出
		ctrl+P+Q:容器不停止退出
	启动容器:docker start 容器名或容器ID
	重启容器:docker restsrt 容器名或容器ID
	停止容器:docker stop 容器名或容器ID
	强制停止:docker kill 容器名或容器ID
	删除已停止的容器:docker rm 容器名或容器ID
		-f:强制删除
	一次删除多个容器：
		docker rm -f $(docker ps -a -q)
		docker ps -a -q | xargs docker rm

---------------------------------------------------------------------------------------
使用镜像centos:lalest以后台模式启动一个容器
docker run -d centos

问题：然后docker ps -a进行查看，会发现容器已经退出了
很重要的要说明的一点：Docker容器后台运行，就必须有一个前台进程
容器运行的命令如果不是那些一直挂起的命令(比如运行top,tail),就是会自动退出的。

这是docker的机制问题，比如你的wen容器，以我们的nginx为例，正常情况下，我们配置启动服务只需要启动响应的service即可。例如 service nginx start
但是，这样做，nginx为后台进程模式运行，就导致docker前台没有运行的应用，这样的容器后台启动，就会自杀因为他觉得他没有事可做了。所以，最佳的解决方案是，将你要运行的程序以前台进程的形式运行。
---------------------------------------------------------------------------------------

	查看容器日志: docker logs -f -t --tail 容器ID
		-t:加入时间戳
		-f:跟随最新的日志打印
		--tail 数字：显示最后多少条
	查看容器内运行的进程:docker top 容器ID
	查看容器内部细:docker inspect 容器ID
	进入正在运行的容器:
		docker exec -it 容器ID bashShell
		重新进入:docker attach 容器ID
			attach:直接进入容器启动命令的终端，不会启动新的进程
			exec:是在容器中打开新的终端，并且可以启动新的进程
	从容器内拷贝文件到主机上:docker cp 容器ID:/tmp/yum.log /root
```

docker镜像

```
镜像:是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需要的所有内容，包括代码、运行时、库、环境变量和配置文件。

UnionFS(联合文件系统)：Union文件系统(UnionFS)是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下。Union文件系统是Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像(没有父镜像)，可以制作各种具体的应用镜像。
特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

Docker镜像加载原理：docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS.
	bootfs(boot file system)主要包含bootloader和kernel,bootloader主要是引导加载kernel,Linux刚启动会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的，加载boot加载器和内核。当boot加载完文件之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。
	rootfs(root file system),在bootfs之上。包含的就是典型的Linux系统中的/dev,/proc,/bin,/etc等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu,CentOS等等。
	对于一个精简的OS,rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接使用Host的Kernel,只需要提供rootfs就行了。由此可以见对于不同的linux发行版，bootfs基本上是一致的，rootfs会有差别，因此不同的发行版可以共用bootfs。
	
分层的镜像：

docker为什么采用分层结构:最大的好处-共享资源
	比如：有多个镜像都从相同的base镜像构建而来，那么宿主机只需要在磁盘上保存一份base镜像，同时内存中只需要加载一份base镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。
	
特点：Docker镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作为“容器层”，“容器层”之下的都叫“镜像层”。

docker commit：提交容器副本使之成为一个新的镜像
docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名：[标签名]
```

Docker容器数据卷

```
Docker理念:
	将运用与运行的环境打包成形成容器运行，运行可以伴随着容器，但是我们对数据的要求希望是持久化的
	容器之间希望可以共享数据
Docker容器产生的数据，如果不通过docker commit生成新的镜像，使得数据做为镜像的一部分保存下来，当容器删除后，数据自然就没有了。
为了能保存数据在docker中使用数据卷。类似Redis里面的rdb和aof文件

卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些持续存储或共享数据的特性。
卷的设计目的就是数据的持久化，完全独立于容器的生命周期，因此Docker不会在容器删除时删除其挂载的数据卷

特点：
	数据卷可以在容器之间共享或重用数据
	卷中的更改可以直接生效
	数据卷中的更改不会包含在镜像的更新中
	数据卷的生命周期一直持续到没有容器使用它为止
	
添加数据卷的方式：
	直接命令添加：
		命令:docker run -it -v /宿主机绝对路径目录:/容器内目录 镜像名
		查看数据卷是否挂载成功:docker inspect 容器ID
		容器和宿主机之间数据共享
		容器停止退出后，主机修改后数据是否同步：完全同步
		命令(带权限):docker run -it -v /宿主机绝对路劲目录:/容器内目录:ro(容器只读) 镜像名
	DockerFile添加：
		1、根目录下创建mydocker文件夹并进入
		2、可在Dockerfile中使用VOLUME指令给镜像添加一个或多个数据卷
			VOLUME["/dataVolumeContaine","/dataVolumeContaine2","/dataVolumeContaine3"]
			说明：出于可移植和分享的考虑，用-v主机目录：容器目录这种方式不能够直接在Dockerfile中实现。由于宿主机目录是依赖于特定宿主机的，并不能够保证在所有的宿主机上都存在这样特定目录。
	File构建：vim Dockerfile
	build后生成镜像：docker build -t <name> .
	run容器：docker run -it  镜像名
```

数据卷容器

```
命名的容器挂载数据卷，其它容器通过挂载这个(父容器)实现数据共享，挂载数据卷的容器，称之为数据卷容器

先启动一个父容器dc01,在容器目录dataVotlumesContainer2新增内容
dc02/dc03继承自dc01
	docker run -it --name dc02/dc03 --volumes-from dc01 镜像名
	dc02/dc03分别在dataVolumes2各自新增内容
回到dc01可以看到02/03各自添加的都能共享
删除dc01,dc02修改后dc03可否访问：可以
删除dc02后dc03可否访问:可以
新建dc04继承dc03后再删除dc03,dc04可否访问：可以

结论：容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用它为止
```

DockerFile解析

```
DockerFile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本

构建三步骤：
	编写Dockerfile文件
	docker build
	docker run
	
基本语法:
	1、每条保留字指令都必须为大写字母且后面要跟随至少一个参数
	2、指令按照从上到下，顺序执行
	3、#表示注解
	4、每条指令都会创建一个新的镜像层，并对镜像进行提交

docker执行Dockerfile的大致流程：
	1、docker从基础镜像运行一个容器
	2、执行一条指令并对容器做出修改
	3、执行类似docker commit的操作提交一个新的镜像层
	4、docker在基于刚提交的镜像运行一个新的容器
	5、执行Dockerfile中的下一条指令直到所有指令都执行完成
```

Dockerfile体系结构(保留字指令)

```
FROM 		基础镜像，当前新镜像是基于哪个镜像的
MAINTAINER 	镜像维护者的姓名和邮箱
RUN 		容器构建时需要运行的命令
EXPOSE 		当前镜像对外暴露出的端口
WORKDIR   	指定在创建容器后，终端默认登录的进来工作目录，一个落脚点
ENV			用来在构建镜像过程中设置环境变量
ADD			将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
COPY		拷贝文件或目录到镜像中 COPY src dest/COPY["src","dest"]
VOLUME		容器数据卷，用于数据保存和持久化工作
CMD			指定一个容器启动时要运行的命令，Dockerfile中可以有多个CMD指令，但只有最后一个生效，CMD会被
			docker run之后方参数替换
ENTRYPOINT	指定一个容器启动时要运行的命令，ENTRYPOINT的目的CMD一样，都是在指定容器启动程序及参数，会对
			docker run之后方参数进行追加
ONBUILD		当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发
```

案例

```
列出镜像的变更历史:docker history 镜像名
```