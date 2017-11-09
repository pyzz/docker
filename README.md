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
    用给定的 tar 压缩包构建 如：  docker build http://server/context.tar.gz
    从标准输入中读取 Dockerfile 进行构建  如： docker build - < Dockerfile 或 cat Dockerfile | docker build -
    从标准输入中读取上下文压缩包进行构建 如：docker build - < context.tar.gz
    　


