# Docker version : 1.13.1
# Dockerizing CentOS7: Dockerfile for building CentOS images

FROM       idocker.io/eryajf/centos:7.4
MAINTAINER eryajf <Linuxlql@163.com>


# Install jdk
ADD  jdk.tar.gz   /usr/local/

ENV JAVA_HOME /usr/local/jdk1.8.0_144
ENV PATH $PATH:$JAVA_HOME/bin
