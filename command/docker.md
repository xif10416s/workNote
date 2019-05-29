# docker 
## win10 安装docker
*   toolbox 安装
*   修改镜像
    *   在Windows命令行执行docker-machine ssh [machine-name]进入VM bash
    *   sudo vi /var/lib/boot2docker/profile
    *   在--label provider=virtualbox的下一行添加--registry-mirror https://registry.docker-cn.com
    *   重启docker服务
*   toolbox win10 修改 proxy <== 不一定有效
    *    C:\Program Files\Docker Toolbox, find start.sh file. add following two proxy settings:
        *   export http_proxy="http://127.0.0.1:1080/"
        *   export https_proxy="http://127.0.0.1:1080/"    




##  centos  初始化
*   ssh 安装

##  常用操作
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



## docker 命令
*   容器启动 centos
    *   docker run -it --privileged=true  1e1148e4cc2c  /usr/sbin/init -p 18001:22
    *   IP是可以指定的 - -https://www.jianshu.com/p/57532f6fe7b0
        *    docker network create -o "com.docker.network.bridge.name"="test01" --subnet 172.20.0.0/16 test01
        *   docker run -it  --rm --ip 172.20.100.100 --net test01 --privileged=true   centos_base:ssh-jdk  /usr/sbin/init -p 18001:22
*   进入容器
    *   docker exec -it 8fb956ee65ec  /bin/bash
*    创建一个卷
    *   docker volume ls
    *   $ docker volume create my-vol
    *   docker volume inspect my-vol
    *   docker run -it --privileged=true  -v  my-vol:/data centos_base:ssh-jdk  /usr/sbin/init -p 18001:22 
    *   https://www.cnblogs.com/sammyliu/p/5932996.html
    *   docker volume rm my-vol
*   
Docker数据卷的备份和还原迁移：
    *   docker run --volumes-from [container name] -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar [container data volume]
    *  docker run --rm -v my-vol:/volume -v $pwd/backup:/backup centos_base:ssh-jdk sh -c 'cp -r /volume /backup'
    *     docker run --volumes-from [container name] -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar [container data volume]
    *   http://note.qidong.name/2018/11/docker-migration/
        *   docker run --rm -d --name test -v test-vol:/data test-img tail -f /dev/null
f4ff81f4c31025ff476fbebc2c779a915b43ba5940b5bcc42e3ef9b1379eaeab
        *   docker run --rm -v test-vol:/volume -v $PWD:/backup alpine tar cvf /backup/backup.tar volumes-from
    *   docker run --rm -v my-vol:/volume -v $pwd/backup:/backup centos_base:ssh-jdk tar cvf /backup/backup.tar volume
    *   docker run --rm -v my-vol:/volume -v $pwd/backup:/backup centos_base:ssh-jdk tar xf /backup/backup.tar



## 端口修改
*   /var/lib/docker/containers/[hash_of_the_container]/hostconfig.json
    *   iptables -t nat -A DOCKER -p tcp --dport 28100 -j DNAT --to-destination 172.20.100.120:10000
    *   iptables -t nat -A DOCKER -p tcp --dport 18888 -j DNAT --to-destination 172.20.100.120:7180


## 问题：
*   D:\Docker Toolbox\docker.exe: Error response from daemon: cgroups: cannot find cgroup mount destination: unknown
    *   docker-machine ssh default
    *   sudo mkdir /sys/fs/cgroup/systemd
        sudo mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd

*  No space left on device Docker toolbox
```
https://forums.docker.com/t/no-space-left-on-device-docker-toolbox/37681
 WARNING: This will delete all existing containers or images used by the docker VM. Please be aware that you might lose data from non-Divio Cloud related containers as well!

You can set disk size using the “–virtualbox-disk-size” flag.

To delete your old VM and create a new one from scratch with 100GB, you’d run:

$ docker-machine rm default

$ docker-machine create -d virtualbox --virtualbox-disk-size "400000" default

-m 100m --memory-swap=100m
```

共享名称
\\?\c:\Users
\\?\E:\vm_share

# docker compose
*   https://blog.csdn.net/pushiqiang/article/details/78682323
*   compose 文件是一个定义服务、 网络和卷的 YAML 文件 。Compose 文件的默认路径是 ./docker-compose.yml
*   https://blog.csdn.net/qq_36148847/article/details/79427878


# Dockerfile
