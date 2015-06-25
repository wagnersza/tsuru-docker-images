FROM tsuru/base:latest
MAINTAINER Wagner Souza <wagnersza@gmail.com>

RUN         apt-get update && apt-get install -y redis-server  unzip libzmq-dev libevent-dev python-dev python-virtualenv

ADD https://github.com/hashicorp/consul-template/releases/download/v0.10.0/consul-template_0.10.0_linux_amd64.tar.gz /tmp/
RUN tar -zxf /tmp/consul-template_0.10.0_linux_amd64.tar.gz -C /bin/ --strip-components=1 && rm -rf /tmp/consul-template_0.10.0_linux_amd64.tar.gz

RUN virtualenv /tmp/circus && cd /tmp/circus && bin/pip install circus

EXPOSE      6379

ADD ./redis.ctmpl /tmp/redis.ctmpl
ADD ./redis-server /etc/init.d/redis-server
RUN chmod +x /etc/init.d/redis-server
ADD ./circus.ini /tmp/circus.ini

ENTRYPOINT /tmp/circus/bin/circusd /tmp/circus.ini --log-output /var/log/circus.log