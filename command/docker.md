*   拉取ubuntu镜像
    -   sudo docker pull daocloud.io/library/ubuntu:xenial-20160317
    -   sudo docker run -it daocloud.io/library/ubuntu
*   做自己的镜像
    -   docker commit c312ca2a51b5 ubuntu:fxiUbuntu
*   导出
    -   docker save -o myubuntu.tar.gz ubuntu:fxiUbuntu
    -   sudo docker run -it ubuntu:fxiUbuntu sh -c '/etc/rc.local;/bin/bash'  
    -   docker load -i myubuntu.tar.gz
*   登录
    -   ssh xifei@192.168.70.176 -p 21001
*   启动内存
    -   -m --memory
*   压缩image,http://jasonwilder.com/blog/2014/08/19/squashing-docker-images/
    -   docker save 49b5a7a88d5 | sudo docker-squash -t jwilder/whoami:squash | docker load
    -   https://github.com/jwilder/docker-squash
    -   sudo ./docker-squash -i spark2.2.tar.gz - o spark.tar.gz
*   docker machin 清理
    -   http://www.cnblogs.com/junneyang/p/6133157.html
    -   docker system prune -a
