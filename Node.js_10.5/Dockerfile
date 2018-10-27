FROM       idocker.io/eryajf/centos:7.4
MAINTAINER eryajf <Linuxlql@163.com>

# Install node
ADD  node.tar.gz   /usr/local/

ENV NODE /usr/local/node
ENV PATH $PATH:$NODE/bin

RUN npm i -g pm2 gulp yarn --registry=https://registry.npm.taobao.org
