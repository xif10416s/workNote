# docker  -- 精通Docker第三版 https://alanhou.org/portainer-gui-docker/
## win10 安装docker
*   toolbox 安装
*   mklink /J "C:\Users\Administrator\.docker" "G:\vm\machines"
*   C:\Users\Administrator\.docker\machine\machines\default\disk.vmdk
*   镜像加速 == http://guide.daocloud.io/dcs/docker-9153151.html
    *   docker pull registry.docker-cn.com/myname/myrepo:mytag
    *   加速列表 --https://juejin.im/post/5cd2cf01f265da0374189441
*   修改镜像
    *   在Windows命令行执行docker-machine ssh [machine-name]进入VM bash
    *   sudo vi /var/lib/boot2docker/profile
    *   在--label provider=virtualbox的下一行添加--registry-mirror https://registry.docker-cn.com
    *   eval $(docker-machine env default)
    *   重启docker服务
*   toolbox win10 修改 proxy <== 不一定有效
    *    C:\Program Files\Docker Toolbox, find start.sh file. add following two proxy settings:
        *   export http_proxy="http://127.0.0.1:1080/"
        *   export https_proxy="http://127.0.0.1:1080/"    


###   创建网卡 - 固定ip
    *    docker network create -o "com.docker.network.bridge.name"="test01" --subnet 172.20.0.0/16 test01   
    *   --ip 172.20.100.120 --net test01

###  win10 本机ip 与docker 容器ip互通 -- docker for windows
*   https://blog.csdn.net/u014104286/article/details/82961203
*   route add -p 172.20.100.0 mask 255.255.0.0 192.168.99.100

### win10 本机ip 与docker 容器ip互通 -- toolbox
*   https://my.oschina.net/gudaoxuri/blog/513923
*   ifconfig eth1:0 192.168.99.10 netmask 255.255.255.0 up

### 动态添加ip映射
*   /var/lib/docker/containers/XXXXXX/hostconfig.json 
*   /var/lib/docker/containers/XXXXXX/config.v2.json 
*   ,"8081/tcp":[{"HostIp":"","HostPort":"28081"}]
*   docker-machine restart

### 测试镜像
*   kylin
    *   docker pull 1nj0zren.mirror.aliyuncs.com/apachekylin/apache-kylin-standalone:3.0.1
*   cdh6.3 kudu-impla
    *   https://hub.docker.com/r/fedormalyshkin/cdh6-impala-kudu
    *   docker pull  1nj0zren.mirror.aliyuncs.com/fedormalyshkin/cdh6-impala-kudu
    *    docker run -d -p 8050:8050 -p 8051:8051 -p 21000:21000 -p 21050:21050 -p 25000:25000 -p 25010:25010 -p 25020:25020 -p 50070:50070  -p 50075:50075 --name cdh6-impala-kudu 1nj0zren.mirror.aliyuncs.com/fedormalyshkin/cdh6-impala-kudu:latest
        *    docker exec -it cdh6-impala-kudu  /bin/bash
*   cdh --TODO
    *   https://hub.docker.com/r/bluedata/cdh632multi/tags
    *   docker pull  1nj0zren.mirror.aliyuncs.com/bluedata/cdh632multi:1.0
        *   docker run -it  --ip 172.20.100.110 --net test01 -h="cdh632" --name cdh632  -d --privileged=true  -v cdh_name_var:/var -v cdh_name_etc:/etc -v cdh_name_opt:/opt   -p 28001:22 -p 28002:7180  -p 28080:8080 -p 28070:7070 -p 28081:8081 -p 28040:4040 -p 28057:50070 -p 28088:8088 -p 28010:16010 -p 28006:60010   -p 28100:10000 -p 28888:8888    1nj0zren.mirror.aliyuncs.com/bluedata/cdh632multi:1.0  
*   centos7.6
    *    docker pull daocloud.io/centos:7.7.1908
        *   docker run -d -it -p  28022:22 -v /vm_share/centos7_base:/home --privileged=true -h="base_centos7" --ip 172.20.100.110 --net test01 --name base_centos7 daocloud.io/centos:7.7.1908 /usr/sbin/init sh -c '/etc/rc.local;'
        *   docker exec -it 26b4b8af698f /bin/bash
*   portainer -- https://alanhou.org/portainer-gui-docker/
    *   docker pull 1nj0zren.mirror.aliyuncs.com/portainer/portainer
        *   docker container run -d -p 9000:9000 -v \\.\pipe\docker_engine:\\.\pipe\docker_engine 1nj0zren.mirror.aliyuncs.com/portainer/portainer
        *   docker run -d -p 9000:9000 -p 8000:8000 --name portainer --restart always -v \\.\pipe\docker_engine:\\.\pipe\docker_engine -v G:\ProgramData\Portainer:G:\data 1nj0zren.mirror.aliyuncs.com/portainer/portainer
        *   docker run -d -p 9000:9000 -p 8000:8000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock   1nj0zren.mirror.aliyuncs.com/portainer/portainer
*   redis集群
    *    docker pull 1nj0zren.mirror.aliyuncs.com/grokzen/redis-cluster
        *   docker run -d -it -p  192.168.99.10:27000:7000 -p  192.168.99.10:27001:7001  -p  192.168.99.10:27002:7002 -p  192.168.99.10:27003:7003 -p  192.168.99.10:27004:7004  -p  192.168.99.10:27005:7005   --privileged=true -h="redis_cluster" --name redis_cluster 1nj0zren.mirror.aliyuncs.com/grokzen/redis-cluster 
        *   docker exec -it XX
            *   redis-cli -c -p 7000
                *   redis-cli -c -p 7000 cluster nodes
*   kafka集群
    *   docker pull 1nj0zren.mirror.aliyuncs.com/wurstmeister/kafka
        *   export http_proxy=http://127.0.0.1:1081
        *   export https_proxy=http://127.0.0.1:1081
        *   docker-compose up -d --scale=kafka=3
*   mysql
    *   docker pull 1nj0zren.mirror.aliyuncs.com/mysql/mysql-server:5.7
    *   docker run --name first-mysql -p 3306:3306 -e MYSQL\_ROOT\_PASSWORD=123456 -d 1nj0zren.mirror.aliyuncs.com/mysql/mysql-server:5.7
    


##  centos  初始化
*   ssh 安装

##  常用操作
*   拉取ubuntu镜像
    -   sudo docker pull daocloud.io/library/ubuntu:xenial-20160317
        *    docker pull registry.docker-cn.com/library//ubuntu:16.04
        *    docker pull registry.docker-cn.com/library/apachekylin/apache-kylin-standalone:3.0.1
    -   sudo docker run -it daocloud.io/library/ubuntu
*   做自己的镜像
    -   docker commit c312ca2a51b5 ubuntu:fxiUbuntu
    -   docker login
        -   docker tag b94fb48155bc  448102337/centos7.7:v1
        -   docker push 448102337/centos7.7:v1
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
    *   启动参数： docker run  --help  [https://blog.csdn.net/Hello_World_QWP/article/details/84554031]
        *    -d, --detach                         Run container in background and print container ID --后台运行
        *   -i, --interactive   即使没有连接，也保持STDIN开放
        *   -t, --tty   为当前容器分配一个客户端
        *   --privileged=true 特权方式
*   进入容器
    *   docker exec -it 8fb956ee65ec  /bin/bash
*    创建一个卷
    *   docker volume ls
    *   $ docker volume create my-vol
    *   docker volume inspect my-vol
    *   docker run -it --privileged=true  -v  my-vol:/data centos_base:ssh-jdk  /usr/sbin/init -p 18001:22 
    *   https://www.cnblogs.com/sammyliu/p/5932996.html
    *   docker volume rm my-vol
*   Docker数据卷的备份和还原迁移：
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

$ docker-machine create -d virtualbox --virtualbox-disk-size "300000" default

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
*   *   *   *   *   *

# firfefox rongqi
```
docker run -d \
    --name=firefox \
    -p 5800:5800 \
    -v /docker/appdata/firefox:/config:rw \
    --ip 172.20.100.101 --net test01 \
    --shm-size 2g \
    jlesage/firefox

  http://your-host-ip:5800    
```