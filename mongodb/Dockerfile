FROM tsuru/base:latest

MAINTAINER Wagner Souza <wagnersza@gmail.com>
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
RUN echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | tee /etc/apt/sources.list.d/10gen.list
RUN apt-get update && apt-get install -y mongodb-org unzip libzmq-dev libevent-dev python-dev python-virtualenv

ADD https://github.com/hashicorp/consul-template/releases/download/v0.10.0/consul-template_0.10.0_linux_amd64.tar.gz /tmp/
RUN tar -zxf /tmp/consul-template_0.10.0_linux_amd64.tar.gz -C /bin/ --strip-components=1 && rm -rf /tmp/consul-template_0.10.0_linux_amd64.tar.gz

RUN virtualenv /tmp/circus && cd /tmp/circus && bin/pip install circus

EXPOSE 27017
VOLUME /data/db

ADD ./mongodb.ctmpl /tmp/mongodb.ctmpl
ADD ./mongodb /etc/init.d/mongodb
RUN chmod +x /etc/init.d/mongodb
ADD ./circus.ini /tmp/circus.ini

ENTRYPOINT /tmp/circus/bin/circusd /tmp/circus.ini --log-output /var/log/circus.log
