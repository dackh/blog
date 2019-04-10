# docker入门
> 主要是docker使用命令
## docker守护进程
### 修改守护进程的网络 
	sudo docker daemon -H tcp://0.0.0.0:2375  
	这条命令将docker守护进程
	//使用docker_host环境变量
	export DOCKER_HOST = "tcp:0.0.0:2375`
### 检查docker是否运行
	sudo status docker		//sudo service docker status
### 启动和关闭
	sudo stop docker		//sudo service docker stop
	sudo start docker		//sudo service docker  start
	
## docker容器操作
###查看docker是否正常工作
	sudo docker info
###运行容器
	sudo docker run -i -t ubuntu /bin/bash
	//-i标记保证容器是STDIN开启的
	//-t标记告诉docker要创建的容器分配一个伪tty终端
	//ubuntu 指定镜像
	// /bin/bash告诉docker容器要运行什么命令

###容器命名
docker默认为创建的容器生成一个随机的名称，可通过--name标记来给容器命名

	sudo docker run --name myName -i -t ubuntu /bin/bash

###启动已经停止的容器

	sudo docker start myName  //也可通过id指定
	//也可使用docker restart命令

###附着到容器上
docker容器重新启动的时候，会沿用`docker run`命令时指定的参数来运行，因此上例中重新启动会运行一个交互式的shell，此外也可以用`docker attach`重新附着到该容器的会话上

	sudo docker attach myName	//也可通过id指定

###创建守护式容器
除了交互式运行的容器，也可以创建长期运行的容器，守护式容器没有交互式会话，非常适合运行应用程序和服务。

	sudo docker run --name daemon_name -d ubuntu /bin/sh -c "while true; do echo hello world;sleep 1;done"
	//-d 标记docker将容器放在后台运行
	//while循环一直打印hello world

###查看docker容器

	docker ps 		//查看正在运行的容器
	docker ps -a 	//查看所有的容器

###查看容器内部
	
	docker logs daemon_name	//获取容器的日志，可通过-f来追踪日志

###日志驱动
docker1.6开始，可通过`--log-driver`选项来控制docker守护线程和容器所有的日志驱动，可以在执行docker守护线程或者执行`docker run`命令时使用这个选项。

	sudo docker run --log-driver="syslog" --name daemon_name -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
	//将daemon_name容器的日志输出到syslog，导致docker logs命令不会输出任何东西

###查看容器内的进程
	
	sudo docker top daemon_name

###在容器内部运行进程

	sudo docker exec -d daemon_name touch /etc/new_config_file
	//-d 标记运行一个后台进程 后面明智执行名气的名字以及执行的命令
	sudo docker exec -t -i daemon_name /bin/bash
	//创建TTY并捕捉STDIN

###关闭容器

	sudo docker stop daemon_name

### 深入容器
	sudo docker inspect daemon_name 
	//查看更多容器的信息
	//该命令也可用来查看镜像信息

###删除容器

	sudo docker rm 80430f8d0921		//通过指定容器id
	//删除全部容器
	sudo docker rm `sudo docker ps -a -q`	//-q标记表示只返回容器的id


## docker镜像

###列出docker镜像

	sudo docker images
	//本地镜像都保存在docker宿主机的/var/lib/docker目录i西安，每个镜像都保存在docker所采用的存储驱动目录下面，如aufs或者devicemapper，也可以在/var/bin/docker/containers目录下看到所有容器

###拉取镜像
	
	sudo docker pull ubuntu:12.04

### 运行一个带标签的docker镜像
	
	sudo docker run -t -i --name new_container ubuntu:12:04 /bin/bash

### 查找镜像
可以通过`docker search`命令来查找所有`docker hub`上公共的可用镜像
	
	sudo docker search puppet
	NAME                        DESCRIPTION                                     STARS   OFFICIAL       AUTOMATED
	puppet/puppetserver         A Docker Image for running Puppet Server. Wi…   75                               
	alekzonder/puppeteer        GoogleChrome/puppeteer image and screenshots…   51                     [OK]

- 仓库名
- 镜像描述
- stars，用户评价
- official，是否官方，由上游开发者管理的镜像（如fedora镜像由fedora团队管理）。
- automated，自动构建，表示这个镜像由docker hub的自动构建流程构建的。	


### 构建镜像
#### 构建镜像有两种方式

- docker commit命令
- docker build命令和dockerfile文件

现在并不推荐使用docker commit命令，而应该使用更灵活、更强大的dockerfile来构建docker镜像

一般来说都是基于已有的基础镜像，而不是“创建”新镜像，从0开始可以参考：[https://docs.docker.com/develop/develop-images/baseimages/](https://docs.docker.com/develop/develop-images/baseimages/ "https://docs.docker.com/develop/develop-images/baseimages/")

#### 创建docker hub账号
创建完镜像之后，可以将镜像推送到docker hub或者私有的registryzhong，完成这个操作需要在docker hub上创建一个账号。
[https://hub.docker.com/signup](https://hub.docker.com/signup)

#### 登陆到docker hub
	
	sudo docker login
	Username:
	Password:
	Login Succeeded


#### 用docker commit来创建镜像

	//创建一个新容器
	sudo docker run -i -t ubuntu /bin/bash
	//在容器内部安装一些软件...
	
	//得到刚刚创建容器的id
	docker ps -l -q
	9649de16bffb
	
	//提交定制容器
	sudo docker commit 9649de16bffb	dack/apache2
		
	//检查新创建的镜像
	docker images dack/apache2
	REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
	dack/apache2        latest              2c5cd556c3f2        About a minute ago   209MB

也可以在提交镜像时指定更多的数据（包含标签）来详细描述所做的修改

	sudo docker commit -m "A new custom image" -a "dack huang" 9649de16bffb dack/apache2:webserver
	
	//-m 提交信息
	//-a 作息信息
	//dack/apache2:webserver 执行镜像的用户名和仓库名，并为该镜像增加一个webserver的标签	


#### 用Dockerfile构建镜像
并不推荐用`docker commit`构建镜像，相反推荐使用`Dockerfile`的定义文件和`docker build`来构建镜像

`Dockerfile`使用基本的基于`DSL(Domain Specific Language)`语法的指令来构建一个`docker`镜像，对比`docker commit`，其更具备可重复性、透明性和幂等性。

保存`Dockerfile`的目录称为上下文，`docker`会在构建镜像时将构建的上下文和该上下文的文件和目录上传到`docker`守护进程。这样`docker`守护进程就可以直接访问用户想在镜像中存储的任何代码、文件或者其他数据。

`dockerfile`中每条指令从上到下依次执行，大体的流程如下：

- docker从基础镜像中运行一个容器
- 执行一条指令，对容器做出修改
- 执行类似docker commit的操作，提交一个新的镜像层
- docker再基于刚提交的镜像运行一个新容器
- 执行下一条指令直至全部执行完毕

所以尽管某一步执行失败了，还是得到一个可以使用的镜像

	# Version : 0.01
	FROM ubuntu:14.04
	MAINTAINER dack huang "dack_huang@163.com"
	RUN apt-get update && apt-get install -y nginx
	RUN echo 'Hi, I am in your container' \
		>/usr/share/nginx/html/index.html
	EXPOSE 80


	FROM			//指定基础镜像
	MAINTAINER		//指定镜像作者
	RUN				//在当前镜像中运行的命令，默认在shell里面使用/bin/sh -c执行
	EXPOSE			//应用程序会使用容器指定的接口


通过`docker build`构建镜像

	docker build -t="dack/static_web"
	//-t设置镜像标签

	//从git仓库上面构建镜像
	//假设这个git仓库存在Dockerfile
	docker build -t="dack/static" git@github.com:dack/docker-static_web	


docker将每一步的构建过程都提交为镜像，所以docker会将之前的镜像层看成缓存，当修改某个步骤之后再次构建的，docker会直接从该步骤开始。

可用`docker build --no-cache`略过缓存

	docker build --no-cache -t="dack/static_web"

`docker history`查看镜像的每一层

	[root@dack static_web]# docker history e5f55354c141
	IMAGE               CREATED             CREATED BY                                      SIZE            COMMENT
	e5f55354c141        5 hours ago         /bin/sh -c #(nop)  EXPOSE 80                    0B                  
	66d44cca6536        5 hours ago         /bin/sh -c echo 'Hi, I am in your container'…   27B                 
	022c0b46f4f8        5 hours ago         /bin/sh -c apt-get update && apt-get install…   34.3MB              
	34d2d2b790f5        5 hours ago         /bin/sh -c #(nop)  MAINTAINER dack huang "da…   0B                  
	390582d83ead        3 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
	<missing>           3 weeks ago         /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B            



# 参考
- 第一本docke书