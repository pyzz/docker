# docker
<h1>docker 的使用命令</h1>

参考：　https://yeasy.gitbooks.io/docker_practice/content/basic_concept/


<h3>docker images</h3>
列出已经下载下来的镜像（显示的＜ｎｏｎｅ＞，是虚悬镜像　）
docker images -a　：可以保持中间镜像一起显示出　（中间镜像一般是＜ｎｏｎｅ＞，但与　虚悬镜像不同，他们是上层镜像的依赖镜像，删除所依赖的镜像中间镜像也会被删除掉）

查找相应的镜像　　docker images ubuntu　或　docker images ubuntu:16.04
docker images　创库名称　＋　标签　：　

删除虚悬镜像　（一般来说，虚悬镜像已经失去了存在的价值　＜ｎｏｎｅ＞）
　　docker rmi $(docker images -q -f dangling=true)　：　

镜像启动命令　nginx命令
    docker run --name webserver -d -p 80:80 nginx：版本　＃　这条命令会用 nginx 镜像启动一个容器，命名为 webserver，并且映射了 80 端口（没有版本号，默认启动最新的版本　latest　）
        $ docker exec -it webserver bash　＃　修改ｎｇｎｉｘ的默认显示页面　
        root@3729b97e8226:/# echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
        root@3729b97e8226:/# exit
        exit
    我们修改了容器的文件，也就是改动了容器的存储层。我们可以通过 docker diff 命令看到具体的改动
    
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]　   ：保存修改的镜像
　　　　$ docker commit \
        --author "Tao Wang <twang2218@gmail.com>" \　＃　修改者
        --message "修改了默认网页" \　＃　修改信息
        webserver \　＃　容器名称　
        nginx:v2　＃　仓库名称：变更的后的标签
    
docker history　＃　查看　镜像的修改记录

    
    　
