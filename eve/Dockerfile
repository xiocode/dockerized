FROM phusion/baseimage:latest
MAINTAINER Tony.Shao "xiocode@gmail.com"

# Init
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && apt-get install -y wget git curl zip unzip python lib32z1 lib32ncurses5 lib32bz2-1.0 python-dev python-software-properties software-properties-common

# JAVA
RUN apt-get update
RUN add-apt-repository -y ppa:webupd8team/java
RUN apt-get update
RUN echo oracle-java7-installer shared/accepted-oracle-license-v1-1 select true | /usr/bin/debconf-set-selections
RUN apt-get install -y oracle-java7-installer

ENV JAVA_HOME /usr/bin/java
ENV PATH $JAVA_HOME:$PATH


# Jenkins
ENV JENKINS_HOME /var/jenkins_home

# Jenkins is ran with user `jenkins`, uid = 1000
# If you bind mount a volume from host/vloume from a data container,
# ensure you use same uid
RUN useradd -d "$JENKINS_HOME" -u 1000 -m -s /bin/bash jenkins

# Jenkins home directoy is a volume, so configuration and build history
# can be persisted and survive image upgrades
VOLUME /var/jenkins_home

# `/usr/share/jenkins/ref/` contains all reference configuration we want
# to set on a fresh new installation. Use it to bundle additional plugins
# or config file with your custom jenkins Docker image.
RUN mkdir -p /usr/share/jenkins/ref/init.groovy.d


COPY init.groovy /usr/share/jenkins/ref/init.groovy.d/tcp-slave-angent-port.groovy

ENV JENKINS_VERSION 1.565.3

# could use ADD but this one does not check Last-Modified header
# see https://github.com/docker/docker/issues/8331
RUN curl -L http://mirrors.jenkins-ci.org/war-stable/1.565.3/jenkins.war -o /usr/share/jenkins/jenkins.war

ENV JENKINS_UC https://updates.jenkins-ci.org
RUN chown -R jenkins "$JENKINS_HOME" /usr/share/jenkins/ref

# Android
WORKDIR /usr/local/
RUN curl -O https://dl.google.com/android/android-sdk_r23.0.2-linux.tgz
RUN tar xzf android-sdk_r23.0.2-linux.tgz

ENV ANDROID_HOME /usr/local/android-sdk-linux
ENV PATH $PATH:$ANDROID_HOME/tools
ENV PATH $PATH:$ANDROID_HOME/platform-tools

# Android API
RUN echo y | android update sdk --filter tools --no-ui --force -a
RUN echo y | android update sdk --filter platform-tools --no-ui --force -a
RUN echo y | android update sdk --filter platform --no-ui --force -a
RUN echo y | android update sdk --filter build-tools-21.1.1 --no-ui -a
RUN echo y | android update sdk --filter sys-img-x86-android-19 --no-ui -a
RUN echo y | android update sdk --filter sys-img-x86-android-20 --no-ui -a
RUN echo y | android update sdk --filter sys-img-x86-android-21 --no-ui -a

# Install Maven
RUN wget -nv http://ftp.tsukuba.wide.ad.jp/software/apache/maven/maven-3/3.1.1/binaries/apache-maven-3.1.1-bin.tar.gz && tar xf apache-maven-3.1.1-bin.tar.gz

# Install Gradle
RUN wget -nv https://downloads.gradle.org/distributions/gradle-2.2.1-all.zip && unzip -oq gradle-2.2.1-all.zip

# Cleanup
RUN rm -rf android-sdk_r23.0.2-linux.tgz
RUN rm -rf apache-maven-3.1.1-bin.tar.gz
RUN rm -rf gradle-1.9-all.zip
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Environment variables
ENV MAVEN_HOME /usr/local/apache-maven-3.1.1
ENV GRADLE_HOME /usr/local/gradle-2.2
ENV PATH $PATH:$MAVEN_HOME/bin
ENV PATH $PATH:$GRADLE_HOME/bin


WORKDIR "$JENKINS_HOME"

# for main web interface:
EXPOSE 8080

# will be used by attached slave agents:
EXPOSE 50000

USER jenkins

COPY jenkins.sh /usr/local/bin/jenkins.sh
ENTRYPOINT ["/usr/local/bin/jenkins.sh"]

# from a derived Dockerfile, can use `RUN plugin.sh active.txt` to setup /usr/share/jenkins/ref/plugins from a support bundle
COPY plugins.sh /usr/local/bin/plugins.sh
