要想使用docker完成一些项目的构建，或者学习，首先需要自己来构建一些基础的镜像，以便于使用。

## 1，构建CentOS7.4。

需要四个文件。

```
[root@localhost centos7]$ls
aliyun-epel.repo  aliyun-mirror.repo  Dockerfile  supervisord.conf
```

### 1，Dockerfile。

```
#
# Dockerizing CentOS7: Dockerfile for building CentOS images
#
#需要一个基础镜像，这里从国内的daocloud下载，速度比较快。
FROM       daocloud.io/library/centos:centos7.4.1708

#维护者
MAINTAINER eryajf <Linuxlql@163.com>

#设置一个时区的环境变量
ENV TZ "Asia/Shanghai"

#虚拟终端
ENV TERM xterm

#dockerfile中有2条命令可以复制文件，1.COPY 2.ADD， ADD比COPY多2个功能，可以写成连接 直接COPY到container，如果是压缩文件，add能自动解压
ADD aliyun-mirror.repo /etc/yum.repos.d/CentOS-Base.repo
ADD aliyun-epel.repo /etc/yum.repos.d/epel.repo

RUN yum install -y curl wget tar bzip2 unzip vim-enhanced passwd sudo yum-utils hostname net-tools rsync man && \
    yum install -y gcc gcc-c++ git make automake cmake patch logrotate python-devel libpng-devel libjpeg-devel && \
    yum install -y --enablerepo=epel pwgen python-pip python-setuptools.noarch lrzsz ntp docker-client && \
    yum clean all

#配置supervisor 进程管理工具，运行单个进程可以不使用
RUN easy_install supervisor && \
    mkdir -m 755 -p /etc/supervisor && \
    mkdir -m 755 /etc/supervisor/conf.d
ADD supervisord.conf /etc/supervisor/supervisord.conf

EXPOSE 22

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/supervisord.conf"]
```

### 2，两个yum源。

`其一：`

```
[root@localhost centos7]$cat aliyun-epel.repo
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
baseurl=http://mirrors.aliyun.com/epel/7/$basearch
#mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch
failovermethod=priority
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[epel-debuginfo]
name=Extra Packages for Enterprise Linux 7 - $basearch - Debug
baseurl=http://mirrors.aliyun.com/epel/7/$basearch/debug
#mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-debug-7&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0

[epel-source]
name=Extra Packages for Enterprise Linux 7 - $basearch - Source
baseurl=http://mirrors.aliyun.com/epel/7/SRPMS
#mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-source-7&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0
```

`其二：`

```
[root@localhost centos7]$cat aliyun-mirror.repo
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/contrib/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
```

### 3，supervisor配置文件。

```
[root@localhost centos7]$cat supervisord.conf
[unix_http_server]
file=/tmp/supervisor.sock   ; the path to the socket file

[supervisord]
logfile=/tmp/supervisord.log ; main log file; default $CWD/supervisord.log
logfile_maxbytes=50MB        ; max main logfile bytes b4 rotation; default 50MB
logfile_backups=10           ; # of main logfile backups; 0 means none, default 10
loglevel=info                ; log level; default info; others: debug,warn,trace
pidfile=/tmp/supervisord.pid ; supervisord pidfile; default supervisord.pid
nodaemon=false               ; start in foreground if true; default false
minfds=1024                  ; min. avail startup file descriptors; default 1024
minprocs=200                 ; min. avail process descriptors;default 200

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; use a unix:// URL  for a unix socket

[include]
files = /etc/supervisor/conf.d/*.ini
```

过多关于supervisor配置的问题，这里就不多做解释了。

开始构建。

	docker build -t idocker.io/eryajf/centos:7.4 .

构建的时候，就将名称定义为自己私服地址。

构建完成查看一下：

```
[root@localhost ~]$docker images
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
idocker.io/eryajf/centos   7.4                 13fb619afd8c        21 hours ago        519 MB
```

之后可以push到自己的私服当中。

	docker push idocker.io/eryajf/centos:7.4

启动：

	docker run -d --name centos idocker.io/eryajf/centos:7.4

## 2，构建jdk基础镜像。

这个镜像事实上在dockerhub上有官方发布的，只不过其底层镜像都是基于deebin的，并不适合日常使用，因此这里就自己来制作一下。

需要两个文件。

```
[root@localhost jdk8]$ls
Dockerfile  jdk.tar.gz
```

### 1，Dockerfile。

```
FROM       idocker.io/eryajf/centos:7.4
MAINTAINER eryajf <Linuxlql@163.com>

# Install jdk
ADD  jdk.tar.gz   /usr/local/

ENV JAVA_HOME /usr/local/jdk1.8.0_144
ENV PATH $PATH:$JAVA_HOME/bin
```

剩下那个是jdk的包，可以在[官网](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html "官网")进行下载。

构建。

	docker build -t idocker.io/eryajf/jdk:1.8 .

启动验证。

```
[root@localhost ~]$docker run -d --name jdk idocker.io/eryajf/jdk:1.8
17c9180d892f2406bb256113ec241843ac1e18f7e20aeb52de67ad8eaef2c724
[root@localhost ~]$docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
17c9180d892f        36a4fcd3d962        "/usr/bin/supervis..."   2 seconds ago       Up 1 second         22/tcp              jdk
[root@localhost ~]$docker exec -it jdk bash
[root@17c9180d892f /]# java -version
java version "1.8.0_144"
Java(TM) SE Runtime Environment (build 1.8.0_144-b01)
Java HotSpot(TM) 64-Bit Server VM (build 25.144-b01, mixed mode)
```

## 3，构建tomcat镜像。

需要用到如下三个文件。

```
[root@localhost tomcat]$ls
Dockerfile  tomcat.ini  tomcat.tar.gz
```

### 1，Dockerfile。

```
[root@localhost tomcat]$cat Dockerfile
FROM       idocker.io/eryajf/jdk:1.8
MAINTAINER eryajf <Linuxlql@163.com>


# Install jdk
ADD  tomcat.tar.gz   /usr/local/
ADD  tomcat.ini /etc/supervisor/conf.d
```

### 2，tomcat.ini。

```
[root@localhost tomcat]$cat tomcat.ini
[program:tomcat]
environment=JAVA_HOME="/usr/local/jdk1.8.0_144",JAVA_BIN="/usr/local/jdk1.8.0_144/bin"
command=/usr/local/tomcat/bin/catalina.sh run
autostart=true
autorestart=true
startsecs=60
priority=1
stopasgroup=true
killasgroup=true
stderr_logfile=/usr/local/tomcat/logs/catalina.out
```

构建镜像。

	docker build -t idocker.io/eryajf/tomcat:8.5 .

启动。

	docker run -d -p 8080:8080 --name tomcat idocker.io/eryajf/tomcat:8.5

## 4，构建Jenkins镜像。

之前使用过从dockerhub上下载的Jenkins官方发布的镜像，然而那些镜像的底层也都不是centos系统的，因此打算自己制作一个Jenkins镜像，以便于使用。

所需原料如下：

```
[root@localhost jenkins]$ls
Dockerfile  maven.tar.gz  ROOT.war
```

其中maven是配置好了的，ROOT.war是Jenkins的包。

`Dockerfile：`

```
[root@localhost jenkins]$cat Dockerfile
FROM       idocker.io/eryajf/tomcat:8.5
MAINTAINER eryajf <Linuxlql@163.com>

ADD  maven.tar.gz /usr/local/
COPY ROOT.war /usr/local/tomcat/webapps/

ENV JAVA_HOME /usr/local/jdk1.8.0_144
ENV MAVEN_HOME=/usr/local/maven
ENV JENKINS_HOME=/home/.jenkins_home
ENV PATH $PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin
```

构建。

	docker build -t idocker.io/eryajf/jenkins:2.138 .

启动验证。

	docker run -d -p 8080:8080 --name jenkins idocker.io/eryajf/jenkins:2.138

这种启动方式只能够正常的将Jenkins启动起来，但是如果想要继续利用Jenkins进行持续集成，那么就需要将宿主机当中的docker命令挂载到容器当中，这时可以使用如下命令：


```
docker run -d -p 8080:8080 --name jenkins -v /home/.jenkins_home:/home/.jenkins_home -v /usr/bin/docker:/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock -v /etc/sysconfig/docker:/etc/sysconfig/docker  idocker.io/eryajf/jenkins:2.138
```

启动之后。
