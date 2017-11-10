<h1>docker 的使用命令</h1>

参考：　https://yeasy.gitbooks.io/docker_practice/content/basic_concept/


<h3>docker images</h3>
列出已经下载下来的镜像（显示的＜ｎｏｎｅ＞，是虚悬镜像　） <br>
<h3>docker images -a　</h3>
<br>　可以保持中间镜像一起显示出　（中间镜像一般是＜ｎｏｎｅ＞，但与　虚悬镜像不同，他们是上层镜像的依赖镜像，删除所依赖的镜像中间镜像也会被删除掉）

<h3>docker images　创库名称　＋　标签　：</h3>
<br>查找相应的镜像　　docker images ubuntu　或　docker images ubuntu:16.04


<h3>docker rmi $(docker images -q -f dangling=true)</h3>
<br>删除虚悬镜像　（一般来说，虚悬镜像已经失去了存在的价值　＜ｎｏｎｅ＞）

<h3>启动与修改镜像</h3>
镜像启动命令　nginx命令<br>
    docker run --name webserver -d -p 80:80 nginx：版本　＃　这条命令会用 nginx 镜像启动一个容器，命名为 webserver，并且映射了 80 端口（没有版本号，默认启动最新的版本　latest　）<br>
        $ docker exec -it webserver bash　＃　修改ｎｇｎｉｘ的默认显示页面　<br>
        root@3729b97e8226:/# echo '《h1》Hello, Docker!《/h1》' > /usr/share/nginx/html/index.html<br>
        root@3729b97e8226:/# exit<br>
        exit　<br>
    我们修改了容器的文件，也就是改动了容器的存储层。我们可以通过 docker diff 命令看到具体的改动<br>
    
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]　   ：保存修改的镜像<br>
　　　　$ docker commit \　　<br>
        --author "Tao Wang <twang2218@gmail.com>" \　＃　修改者　<br>
        --message "修改了默认网页" \　＃　修改信息<br>
        webserver \　＃　容器名称　　<br>
        nginx:v2　＃　仓库名称：变更的后的标签　<br>
    
docker history　＃　查看　镜像的修改记录　<br>

<h3>使用 Dockerfile 定制镜像</h3>
<h4>编写Dockerfile文件</h4> 
在一个空白目录中，建立一个文本文件，并命名为 Dockerfile<br>
　　其内容为：<br>
      FROM nginx<br>
      RUN echo 'h1>Hello, Docker!</h1' > /usr/share/nginx/html/index.html<br>
FROM 指定基础镜像<br>
RUN 执行命令<br>
　　shell 格式：RUN <命令>　如上<br>
  　exec 格式：RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式<br>
   
   ＃　注意１，每一个ｒｕｎ都为建了一层新的镜像，果然是连贯的ｓｈｅｌｌ命令应如下，支持&& 将各个所需命令串联起来， # 进行注释的格式，\ 的命令换行方式<br>
   ＃　注意２，每一层的东西并不会在下一层被删除，会一直跟随着镜像，因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该<b>清理掉</b><br>
   
   FROM debian:jessie
   RUN buildDeps='gcc libc6-dev make' \
       && apt-get update \
       && apt-get install -y $buildDeps \
       && wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
       && mkdir -p /usr/src/redis \
       && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
       && make -C /usr/src/redis \
       && make -C /usr/src/redis install \
       && rm -rf /var/lib/apt/lists/* \
       && rm redis.tar.gz \
       && rm -r /usr/src/redis \
       && apt-get purge -y --auto-remove $buildDeps
       
<h4>构建镜像</h4>

在 Dockerfile 文件所在目录执行：
   docker build -t nginx:v3 .
   docker build [选项] <上下文路径/URL/->   ，指定了最终镜像的名称与标签 -t nginx:v3
   注意1，docker build 命令最后有一个 “.”  ，表示当前目录， 这是在指定<b>上下文路径</b>。 
    
    docker build 还支持从 URL 构建，比如可以直接从 Git repo 中构建, 如：docker build https://github.com/twang2218/gitlab-ce-zh.git#:8.14
    <br>
    用给定的 tar 压缩包构建 如：  docker build http://server/context.tar.gz
    <br>
    从标准输入中读取 Dockerfile 进行构建  如： docker build - < Dockerfile 或 cat Dockerfile | docker build -
    <br>
    从标准输入中读取上下文压缩包进行构建 如：docker build - < context.tar.gz
    <br>
    　

<h3>删除镜像</h3>
docker rmi [选项] <镜像1> [<镜像2> ...] 如: docker rmi 501 镜像id，镜像名称 = <仓库名>:<标签> 
<br>
更精确的删除 是使用 “镜像摘要” 删除镜像
docker images --digests
$ docker images --digests
REPOSITORY                  TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
node                        slim                sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228   6e0c4c8e3913        3 weeks ago         214 MB

<b>$ docker rmi node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228</b><br>
Untagged: node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228<br>
  docker rmi $(docker images -q -f dangling=true)  批量删除虚悬镜像<br>
  docker rmi $(docker images -q redis)  删除所有仓库名为 redis 的镜像<br>
  docker rmi $(docker images -q -f before=mongo:3.2)  删除所有在 mongo:3.2 之前的镜像<br>
  

<h3>容器控制</h3>
   <h4>启动容器</h4>
    docker run 镜像名称  # 如：docker run ubuntu:14.04 /bin/echo 'Hello world' <br>
    docker run -t -i ubuntu:14.04 /bin/bash  # 其中，-t 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， -i 则让容器的标准输入保持打开。 <br>
    
   可以利用 docker start 命令，直接将一个已经终止的容器启动运行。
       可以在伪终端中利用 ps 或 top 来查看进程信息
       
       
 <h4>后台(background)运行</h4>
   更多的时候，需要让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加 -d 参数来实现。 <br> 
        如：在寄存机上控制需要 “-d”参数  docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done" <br>
   可以通过 docker ps 命令来查看容器信息。<br>
   要获取容器的输出信息，可以通过 docker logs 命令。 <br>
   
   <h4>终止容器</h4>
       docker stop 来终止一个运行中的容器<br>
          只启动了一个终端的容器，用户通过 exit 命令或 Ctrl+d 来退出终端时，所创建的容器立刻终止。<br>
       docker restart 命令会将一个运行态的容器终止，然后再重新启动它。<br>
       
   <h4>进入容器</h4>
       某些时候需要进入容器进行操作，有很多种方法，包括使用 docker attach 命令或 nsenter 工具等。
       
       docker attach 是 Docker 自带的命令。下面示例如何使用该命令。<br>
       $ sudo docker run -idt ubuntu
       243c32535da7d142fb0e6df616a3c3ada0b8ab417937c853a9e1c251f499f550
       $ sudo docker ps
       CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
       243c32535da7        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                   nostalgic_hypatia
       $sudo docker attach nostalgic_hypatia
       root@243c32535da7:/#
       
    <h4>导入导出容器</h4>
       cat ubuntu.tar | sudo docker import - test/ubuntu:v1.0 #  导入
       sudo docker export 7691a814370e > ubuntu.tar  # 导出
       
    <h4>删除容器</h4>
        docker ps -a 命令可以查看所有已经创建的包括终止状态的容器 <br>
        docker rm  trusting_newton  #  trusting_newton 是ps 查询出的names <br>
        如果要删除一个运行中的容器，可以添加 -f 参数 <br>
        可以组合docker rm $(docker ps -a -q) 可以全部清理掉  <br>
