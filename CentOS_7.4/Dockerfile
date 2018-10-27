#
# Dockerizing CentOS7: Dockerfile for building CentOS images
#
FROM       idocker.io/centos:7.4
MAINTAINER eryajf <Linuxlql@163.com>

ENV TZ "Asia/Shanghai"
ENV TERM xterm

ADD aliyun-mirror.repo /etc/yum.repos.d/CentOS-Base.repo
ADD aliyun-epel.repo /etc/yum.repos.d/epel.repo

RUN yum install -y curl wget tar bzip2 unzip vim-enhanced passwd sudo yum-utils hostname net-tools rsync man && \
    yum install -y gcc gcc-c++ git make automake cmake patch logrotate python-devel libpng-devel libjpeg-devel && \
    yum install -y --enablerepo=epel pwgen python-pip python-setuptools.noarch lrzsz ntp docker-client && \
    yum clean all

RUN easy_install supervisor && \
    mkdir -m 755 -p /etc/supervisor && \
    mkdir -m 755 /etc/supervisor/conf.d
ADD supervisord.conf /etc/supervisor/supervisord.conf

EXPOSE 22

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/supervisord.conf"]
