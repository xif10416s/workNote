#### 使用 Dockerfile 定制镜像

#####  基础命令

* FROM: 定制的镜像都是基于 FROM 的镜像，这里的 nginx 就是定制需要的基础镜像。后续的操作都是基于 nginx。 
  * 与 docker pull image一致
* MAINTAINER -- 维护信息
  * MAINTAINER Edison Zhou <edisonchou@hotmail.com>
  * 不过，MAINTAINER并不推荐使用，更推荐使用LABEL来指定镜像作者，例如
    * LABEL maintainer="edisonzhou.cn"
* RUN -- 构建是运行的shell命令
* CMD -- 启动容器是执行的shell命令
* EXPOSE-- 声明容器运行的服务端口
* ENV --  环境变量，如JAVA_HOME 
* ADD -- 拷贝文件到镜像中
  * ADD src  dest
  * 如果是URL或压缩包，会自动下载或自动解压。
* COPY -- 拷贝文件到目录镜像，不支持自动下载与解压
* ENTRYPOINT -- 　启动容器时执行的Shell命令，同CMD类似，只是由ENTRYPOINT启动的程序**不会被docker run命令行指定的参数所覆盖**，而且，**这些命令行参数会被当作参数传递给ENTRYPOINT指定指定的程序**
* VOLUME -- 指定容器挂载点到宿主机自动生成的目录或其他容器




```

FROM daocloud.io/centos:7.7.1908

ENV LANG=en_US.UTF-8

RUN yum install -y wget bzip2 ca-certificates \
    libglib2.0-0 libxext6 libsm6 libxrender1 \
    git mercurial subversion

RUN echo 'export PATH=/opt/conda/bin:$PATH' > /etc/profile.d/conda.sh && \
    wget --quiet https://repo.continuum.io/archive/Anaconda3-2019.07-Linux-x86_64.sh -O ~/anaconda.sh && \
    /bin/bash ~/anaconda.sh -b -p /opt/conda && \
    rm ~/anaconda.sh

ENV PATH /opt/conda/bin:$PATH



# 设置时区
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN yum -y update

RUN yum -y install sudo \
&& yum -y install net-tools \
&& yum -y install openssh-server \
&& yum -y install openssh-clients \
&& yum -y install vim \
&& yum -y install git \

# 修改root密码
RUN echo "root" | passwd --stdin root


# 开放22号端口
EXPOSE 22



CMD /bin/bash  ; /usr/sbin/sshd -D

```


####  参考
*	https://www.cnblogs.com/edisonchou/p/dockerfile_inside_introduction.html