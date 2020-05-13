#### 使用 Dockerfile 定制镜像

#####  基础命令

* FROM: 定制的镜像都是基于 FROM 的镜像，这里的 nginx 就是定制需要的基础镜像。后续的操作都是基于 nginx。




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