FROM centos:7
MAINTAINER "Francisco Rodriguez" <frodriguez@bluecedar.com>

# Java Env Variables
ENV JAVA_VERSION=jdk1.8.0_181
ENV JAVA_TARBALL=jdk-8u181-linux-x64.tar.gz
ENV JAVA_HOME=/opt/java/${JAVA_VERSION}

# Swarm Env Variables (defaults)
ENV SWARM_MASTER=http://jenkins:8080
ENV SWARM_USER=jenkins
ENV SWARM_PASSWORD=jenkins

# Slave Env Variables
ENV SLAVE_NAME="jenkins_agent"
ENV SLAVE_LABELS="docker aws ldap"
ENV SLAVE_MODE="exclusive"
ENV SLAVE_EXECUTORS=2
ENV SLAVE_DESCRIPTION="Core Jenkins Slave"

# Pre-requisites
RUN yum -y install epel-release
RUN yum install -y which \
    git \
    wget \
    tar \
    zip \
    unzip \
    openldap-clients \
    openssl \
    python-pip \
    libxslt && \
    yum clean all 

RUN pip install --upgrade pip

RUN pip install awscli==1.10.19


# Docker versions Env Variables
ENV DOCKER_ENGINE_VERSION=1.10.3-1.el7.centos
ENV DOCKER_COMPOSE_VERSION=1.21.2
ENV DOCKER_MACHINE_VERSION=v0.14.0

#COPY scripts/get-docker.sh /tmp/.
#RUN chmod 755 /tmp/get-docker.sh
#RUN /tmp/get-docker.sh

RUN curl -fsSL https://get.docker.com/ | sed "s/docker-engine/docker-engine-${DOCKER_ENGINE_VERSION}/" | sh

RUN curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose && \
    chmod +x /usr/local/bin/docker-compose
RUN curl -L https://github.com/docker/machine/releases/download/${DOCKER_MACHINE_VERSION}/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine && \
    chmod +x /usr/local/bin/docker-machine

# Install Java

RUN  mkdir -p /opt/java
ADD java/${JAVA_TARBALL} /opt/java/.

RUN alternatives --install /usr/bin/java java /opt/java/${JAVA_VERSION}/bin/java 100 && \
    rm -rf /var/log/*

# Make Jenkins a slave by installing swarm-client
RUN curl -s -o /bin/swarm-client.jar -k http://repo.jenkins-ci.org/releases/org/jenkins-ci/plugins/swarm-client/3.12/swarm-client-3.12.jar

# Install Maven
RUN cd /usr/local/src/ && \
    wget http://www-us.apache.org/dist/maven/maven-3/3.5.3/binaries/apache-maven-3.5.3-bin.tar.gz && \
    tar -xf apache-maven-3.5.3-bin.tar.gz && \
    rm -f apache-maven-3.5.3-bin.tar.gz && \
    mv apache-maven-3.5.3/ apache-maven/

ADD scripts/maven.sh /etc/profile.d/maven.sh
RUN chmod +x /etc/profile.d/maven.sh && \
    source /etc/profile.d/maven.sh

# Install Docker
#RUN yum check-update
RUN curl -fsSL https://get.docker.com/ | sh
RUN systemctl enable docker

# Install Node
RUN curl --silent --location https://rpm.nodesource.com/setup_8.x | bash -
RUN yum -y install nodejs gcc-c++ make 

# Install helm
ENV HELM_VERSION v2.8.2
ENV HELM_FILENAME helm-${HELM_VERSION}-linux-amd64.tar.gz
ENV KUBE_VERSION="v1.10.2"

WORKDIR /
RUN curl -L https://kubernetes-helm.storage.googleapis.com/${HELM_FILENAME} | tar zxv -C /tmp \
  && cp /tmp/linux-amd64/helm /usr/local/bin/helm \
  && chmod +x /usr/local/bin/helm

RUN curl -LO https://storage.googleapis.com/kubernetes-release/release/${KUBE_VERSION}/bin/linux/amd64/kubectl \
-o /usr/local/bin/kubectl
RUN mv ./kubectl /usr/local/bin/kubectl
RUN chmod +x /usr/local/bin/kubectl
RUN helm init --client-only

# Install Gradle
RUN wget https://services.gradle.org/distributions/gradle-4.9-bin.zip \
  && mkdir /opt/gradle \
  && unzip -d /opt/gradle gradle-4.9-bin.zip \
  && export PATH=$PATH:/opt/gradle/gradle-4.9/bin \
  && ln -s /opt/gradle/gradle-4.9/bin/gradle /usr/bin/gradle \
  && gradle -v

# Start Swarm-Client
CMD java -jar /bin/swarm-client.jar -executors ${SLAVE_EXECUTORS} -description "${SLAVE_DESCRIPTION}" -master ${SWARM_MASTER} -username ${SWARM_USER} -password ${SWARM_PASSWORD} -name "${SLAVE_NAME}" -labels "${SLAVE_LABELS}" -mode ${SLAVE_MODE}

RUN cp /etc/hosts /tmp/hosts

RUN echo "10.42.16.17 stash.corp.bluecedar.com stash" >> /tmp/hosts
RUN echo "104.16.175.166 services.gradle.org" >> /tmp/hosts

RUN cat /tmp/hosts
RUN cp /tmp/hosts /etc/hosts

RUN mkdir -p -- /lib-override && cp /lib64/libnss_files.so.2 /lib-override
RUN sed -i 's:/etc/hosts:/tmp/hosts:g' /lib-override/libnss_files.so.2
ENV LD_LIBRARY_PATH /lib-override

