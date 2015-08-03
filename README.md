# Install tsuru with docker

## Install first consul machine
  ```bash
  $ eval "$(docker-machine env consul01)"
  $ docker run -d -v /data/consul:/data/consul \
      -p 8300:8300 \
      -p 8301:8301 \
      -p 8301:8301/udp \
      -p 8302:8302 \
      -p 8302:8302/udp \
      -p 8400:8400 \
      -p 8500:8500 \
      -p 53:53/udp \
      progrium/consul -server -advertise `docker-machine ip consul01` -bootstrap
  ```

## Insltall docker machines
  ```bash
  $ docker-machine create --swarm --swarm-discovery consul://`docker-machine ip consul01`:8500/swarm --engine-opt dns=172.17.42.1 --engine-opt dns=8.8.8.8 --engine-opt dns-search=service.consul -d virtualbox docker01
  $ docker-machine create --swarm --swarm-discovery consul://`docker-machine ip consul01`:8500/swarm --engine-opt dns=172.17.42.1 --engine-opt dns=8.8.8.8 --engine-opt dns-search=service.consul -d virtualbox docker02
  $ docker-machine create --swarm --swarm-discovery consul://`docker-machine ip consul01`:8500/swarm --engine-opt dns=172.17.42.1 --engine-opt dns=8.8.8.8 --engine-opt dns-search=service.consul -d virtualbox docker03
  $ docker-machine create --swarm --swarm-master --swarm-discovery consul://`docker-machine ip consul01`:8500/swarm --engine-opt dns=172.17.42.1 --engine-opt dns=8.8.8.8 --engine-opt dns-search=service.consul -d virtualbox swarm-master
  ```
## Install consul cluster and registrator

  ```bash
  $ eval "$(docker-machine env swarm-master)"
  $ docker run -d -v /data/consul:/data/consul \
      -p 8300:8300 \
      -p 8301:8301 \
      -p 8301:8301/udp \
      -p 8302:8302 \
      -p 8302:8302/udp \
      -p 8400:8400 \
      -p 8500:8500 \
      -p 53:53/udp \
      progrium/consul -server -advertise `docker-machine ip swarm-master` -join `docker-machine ip consul01`

  $ docker run -d -v /var/run/docker.sock:/tmp/docker.sock progrium/registrator -resync 3 consul://`docker-machine ip swarm-master`:8500
  ```
  ```bash
  $ eval "$(docker-machine env docker01)"
  $ docker run -d -v /data/consul:/data/consul \
    -p 8300:8300 \
    -p 8301:8301 \
    -p 8301:8301/udp \
    -p 8302:8302 \
    -p 8302:8302/udp \
    -p 8400:8400 \
    -p 8500:8500 \
    -p 53:53/udp \
    progrium/consul -server -advertise `docker-machine ip docker01` -join `docker-machine ip consul01`

  $ docker run -d -v /var/run/docker.sock:/tmp/docker.sock progrium/registrator consul://`docker-machine ip docker01`:8500
  ```
  ```bash
  $ eval "$(docker-machine env docker02)"
  $ docker run -d -v /data/consul:/data/consul \
      -p 8300:8300 \
      -p 8301:8301 \
      -p 8301:8301/udp \
      -p 8302:8302 \
      -p 8302:8302/udp \
      -p 8400:8400 \
      -p 8500:8500 \
      -p 53:53/udp \
      progrium/consul -server -advertise `docker-machine ip docker02` -join `docker-machine ip consul01`

  $ docker run -d -v /var/run/docker.sock:/tmp/docker.sock progrium/registrator consul://`docker-machine ip docker02`:8500
  ```
  ```bash
  $ eval "$(docker-machine env docker03)"
  $ docker run -d -v /data/consul:/data/consul \
    -p 8300:8300 \
    -p 8301:8301 \
    -p 8301:8301/udp \
    -p 8302:8302 \
    -p 8302:8302/udp \
    -p 8400:8400 \
    -p 8500:8500 \
    -p 53:53/udp \
    progrium/consul -server -advertise `docker-machine ip docker03` -join `docker-machine ip consul01`

  $ docker run -d -v /var/run/docker.sock:/tmp/docker.sock progrium/registrator consul://`docker-machine ip docker03`:8500
  ```
## Start tsuru tears

### MongoDB
  ```bash
  $ docker run -d --name mongodb -h mongodb -p 27017:27017 -v /data/db:/data/db mongo
  ```
### Redis
  ```bash
  $ docker run -d --name redis -h redis -p 6376:6376 redis
  ```
### Tsuru API
  ```bash
  $ docker run -d --name api -h api -p 8080:8080 -v /data/api:/data/api wagnersza/tsuru-api
  ```
### Docker Registry
```bash
  $ docker run -d --name registry -h registry -e STORAGE_PATH=/data/registry -p 5000:5000 -v /data/registry:/data/registry registry
  ```
### Router
```bash
  $ docker run -d --name api -h api -p 80:8080 -v wagnersza/router
  ```
### Gandalf
```bash
  $ docker run -d --name gandalf -h gandalf -p 8081:8081 -v /data/gandalf:/data/gandalf wagnersza/gandalf
  ```
