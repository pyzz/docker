# docker
docker 的使用命令


#　docker images ：列出已经下载下来的镜像（显示的＜ｎｏｎｅ＞，是虚悬镜像　）
docker images -a　：可以保持中间镜像一起显示出　（中间镜像一般是＜ｎｏｎｅ＞，但与　虚悬镜像不同，他们是上层镜像的依赖镜像，删除所依赖的镜像中间镜像也会被删除掉）

查找相应的镜像　　docker images ubuntu　或　docker images ubuntu:16.04
docker images　创库名称　＋　标签　：　

删除虚悬镜像　（一般来说，虚悬镜像已经失去了存在的价值　＜ｎｏｎｅ＞）
　　docker rmi $(docker images -q -f dangling=true)　：　

镜像启动命令　nginx命令
    docker run --name webserver -d -p 80:80 nginx
