FROM ubuntu:14.04

MAINTAINER Wagner Souza <wagnersza@gmail.com>
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys B0DE9C5DEBF486359EB255B03B0153D0383F073D
RUN echo "deb http://ppa.launchpad.net/tsuru/ppa/ubuntu trusty main"  > /etc/apt/sources.list.d/tsuru.list
RUN apt-get update
RUN apt-get install -y tsuru-server unzip libzmq-dev libevent-dev python-dev python-virtualenv

ADD https://github.com/hashicorp/consul-template/releases/download/v0.10.0/consul-template_0.10.0_linux_amd64.tar.gz /tmp/
RUN tar -zxf /tmp/consul-template_0.10.0_linux_amd64.tar.gz -C /bin/ --strip-components=1 && rm -rf /tmp/consul-template_0.10.0_linux_amd64.tar.gz

RUN virtualenv /tmp/circus && cd /tmp/circus && bin/pip install circus

EXPOSE      8080

VOLUME	/data/api

ADD ./tsuru-defaults /etc/default/tsuru-server
ADD ./tsuru.ctmpl /tmp/tsuru.ctmpl
ADD ./tsuru-server-api /etc/init.d/tsuru-server-api
RUN chmod +x /etc/init.d/tsuru-server-api
ADD ./circus.ini /tmp/circus.ini

ENTRYPOINT /tmp/circus/bin/circusd /tmp/circus.ini --log-output /var/log/circus.log
