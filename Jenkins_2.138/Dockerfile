# Docker version : 1.13.1
# this Dockerfile is build jenkins images

FROM       idocker.io/eryajf/tomcat:8.5

MAINTAINER eryajf <Linuxlql@163.com>

#将已经配置完善了的maven部署进去。
ADD  maven.tar.gz /usr/local/

#将最新稳定版本的Jenkins（2.138）包拷贝进去。
COPY ROOT.war /usr/local/tomcat/webapps/

#设置环境变量。
ENV JAVA_HOME /usr/local/jdk1.8.0_144
ENV MAVEN_HOME=/usr/local/maven
ENV JENKINS_HOME=/opt/jenkins
ENV PATH $PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin
