FROM jenkins/jenkins:lts

USER root
# force Jenkins to be a sudoer
RUN apt-get update \
      && apt-get install -y vim sudo curl\
      && rm -rf /var/lib/apt/lists/*
RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers

ARG JENKINS_HOME=/var/jenkins_home
ENV JENKINS_HOME $JENKINS_HOME
ENV JAVA_OPTS -Djenkins.install.runSetupWizard=false

# install docker-ce
RUN apt -y  update && \
    apt -y  upgrade && \
    apt -y install sudo apt-transport-https ca-certificates curl software-properties-common gnupg2 && \
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add - && \
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" && \
    apt -y update && \
    apt -y install docker-ce

# install Maven
RUN cd /usr/local && \
    wget https://www-us.apache.org/dist/maven/maven-3/3.6.0/binaries/apache-maven-3.6.0-bin.tar.gz && \
    tar -zxvf apache-maven-3.6.0-bin.tar.gz  && \
    mv apache-maven-3.6.0 apache-maven && \
    rm apache-maven-3.6.0-bin.tar.gz

COPY maven.sh /etc/profile.d
RUN chmod +x /etc/profile.d/maven.sh

# install Oracle Java 11
WORKDIR /usr/local
RUN wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; \
    oraclelicense=accept-securebackup-cookie" \
    https://download.oracle.com/otn-pub/java/jdk/11.0.1+13/90cf5d8f270a4347a95050320eef3fb7/jdk-11.0.1_linux-x64_bin.tar.gz && \
    tar -zxvf jdk-11.0.1_linux-x64_bin.tar.gz && \
    rm jdk-11.0.1_linux-x64_bin.tar.gz


# install Scala and sbt 
RUN wget https://scala-lang.org/files/archive/scala-2.12.8.deb -P /tmp  && \
    dpkg -i /tmp/scala-2.12.8.deb 

RUN echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list && \
    apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823 && \
    apt-get -y  update && \
    apt-get -y  install sbt       
   

COPY install-plugins.sh /usr/local/bin/install-plugins.sh
COPY jenkins-support /usr/local/bin/jenkins-support

# install all plugins
COPY plugins.txt /var/jenkins_home/plugins.txt
RUN chmod +x /usr/local/bin/install-plugins.sh
RUN xargs /usr/local/bin/install-plugins.sh  < /var/jenkins_home/plugins.txt

# Install all jobs, pipelines, users and environment
ADD jenkins.tar.gz /usr/share/jenkins/ref/

# copy script-security disable
COPY groovy/disable-script-security.groovy /var/jenkins_home/init.groovy.d
