# PanteraS
# Platform as a Service based on Paas in a box project
#
FROM ubuntu:14.04.2

MAINTAINER Wojciech Sielski "wsielski@team.mobile.de"

RUN locale-gen en_US.UTF-8

ENV DEBIAN_FRONTEND noninteractive
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8
ENV TERM xterm
ENV HOME /root
ENV GOPATH ${HOME}/go

RUN apt-get update \
    && apt-get install -y \
      python-pip \
      wget \
      curl \
      unzip \
      dnsutils \
      vim \
      golang \
      git \
      lolcat \
      toilet

RUN pip install supervisor-stdout
RUN pip install https://github.com/Supervisor/supervisor/archive/3.2-branch.zip

ENV DOCKER_APP_VERSION 1.5.0
ENV CONSUL_APP_VERSION 0.5.0
ENV CONSUL_TEMPLATE_APP_VERSION 0.9.0
ENV ENVCONSUL_APP_VERSION 0.3.0
ENV HAPROXY_APP_VERSION 1.5.3-1~ubuntu14.04.1
ENV MESOS_APP_VERSION 0.21.1-1.1.ubuntu1404
ENV MARATHON_APP_VERSION 0.8.1-1.0.171.ubuntu1404
ENV REGISTRATOR_APP_VERSION v5

ENV DOCKER_HOST unix:///tmp/docker.sock


# CONSUL
#
RUN mkdir /opt/consul && mkdir /etc/consul.d/
ENV PATH $PATH:/opt/consul
WORKDIR /opt/consul

RUN wget https://dl.bintray.com/mitchellh/consul/${CONSUL_APP_VERSION}_linux_amd64.zip \
    && wget https://dl.bintray.com/mitchellh/consul/${CONSUL_APP_VERSION}_web_ui.zip \
    && wget https://github.com/hashicorp/envconsul/releases/download/v${ENVCONSUL_APP_VERSION}/envconsul_${ENVCONSUL_APP_VERSION}_linux_amd64.tar.gz \
    && unzip ${CONSUL_APP_VERSION}_linux_amd64.zip \
    && unzip ${CONSUL_APP_VERSION}_web_ui.zip \
    && tar zxf envconsul_${ENVCONSUL_APP_VERSION}_linux_amd64.tar.gz \
    && ln -s envconsul_${ENVCONSUL_APP_VERSION}_linux_amd64/envconsul .


# DNSMASQ
#
RUN apt-get install -y dnsmasq dnsutils


# HAPROXY
#
RUN mkdir /opt/consul-template
ENV PATH $PATH:/opt/consul-template
WORKDIR /opt/consul-template

RUN sed -i 's/^# \(.*-backports\s\)/\1/g' /etc/apt/sources.list \
    && apt-get update \
    && apt-get install -y \
    haproxy="${HAPROXY_APP_VERSION}"

RUN sed -i 's/^ENABLED=.*/ENABLED=1/' /etc/default/haproxy \
    && rm -rf /var/lib/apt/lists/*


# consul-haproxy
RUN wget https://github.com/hashicorp/consul-template/releases/download/v${CONSUL_TEMPLATE_APP_VERSION}/consul-template_${CONSUL_TEMPLATE_APP_VERSION}_linux_amd64.tar.gz \
    && tar zxf consul-template_${CONSUL_TEMPLATE_APP_VERSION}_linux_amd64.tar.gz \
    && ln -s consul-template_${CONSUL_TEMPLATE_APP_VERSION}_linux_amd64/consul-template .

#RUN go get -u github.com/hashicorp/consul-template \
#    && ln -s ${GOPATH}/bin/consul-template .

# Add files
ADD haproxy.cfg /etc/haproxy/haproxy.cfg
# same file for an "empty template"
ADD haproxy.cfg        /opt/consul-template/
ADD template.conf      /opt/consul-template/
ADD haproxy_watcher.sh /opt/consul-template/
ADD haproxy_reload.sh  /opt/consul-template/


# ZOOKEEPER MESOS MARATHON
#
RUN echo "deb http://repos.mesosphere.io/ubuntu/ trusty main" > /etc/apt/sources.list.d/mesosphere.list

#RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv E56151BF
RUN gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv E56151BF && gpg --export --armor E56151BF | apt-key add -

RUN apt-get update \
    && apt-get -y install \
      mesos=${MESOS_APP_VERSION} \
      marathon=${MARATHON_APP_VERSION}

ADD zkStart.sh /opt/zkStart.sh
RUN rm /etc/mesos/zk

RUN echo 'deb http://get.docker.io/ubuntu docker main' > /etc/apt/sources.list.d/docker.list
#RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D8576A8BA88D21E9
RUN gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv D8576A8BA88D21E9 && gpg --export --armor D8576A8BA88D21E9 | apt-key add -
RUN apt-get update \
    && apt-get install -y lxc-docker-${DOCKER_APP_VERSION}

# Adding docker wrapper, so CONTAINER_NAME
# will be available for every mesos containers
ADD docker /usr/local/sbin/docker


# REGISTARTOR
#
RUN mkdir /opt/registrator
ENV PATH ${PATH}:/opt/registrator
WORKDIR /opt/registrator

# [DISABLED temporaly]
#
# You can setup .golangrc with proxy setup if you wanna build behind proxy
#RUN [ -f ${HOME}/.golangrc ] && . ${HOME}/.golangrc; \
#    go get -u -tags ${REGISTRATOR_APP_VERSION} github.com/gliderlabs/registrator \
#    && cp ${GOPATH}/bin/registrator .

# Binaries are not supported anymore
#  && wget https://github.com/progrium/registrator/releases/download/v${REGISTRATOR_APP_VERSION}/registrator_${REGISTRATOR_APP_VERSION}_linux_x86_64.tgz \
#  && tar zxf registrator_${REGISTRATOR_APP_VERSION}_linux_x86_64.tgz

# Providing binaries by us - temporaly
RUN wget https://github.com/eBayClassifiedsGroup/registrator/releases/download/${REGISTRATOR_APP_VERSION}/registrator.tgz \
    && tar zxf registrator.tgz

# Font for logo
RUN wget https://raw.githubusercontent.com/patorjk/figlet.js/master/fonts/Elite.flf -O /usr/share/figlet/Elite.flf

ADD consul_config.json /etc/consul.d/
ADD supervisord.conf /etc/supervisord.conf
ADD supervisord.sh /opt/
ADD version /opt/

WORKDIR /opt
ENTRYPOINT [ "/opt/supervisord.sh" ]
