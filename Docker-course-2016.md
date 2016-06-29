# Day 1
[pre-reqs](https://gist.github.com/spkane/98ac0c87a019f9305c831966e4d5f0b8)
[slides](https://gist.github.com/spkane/6b38360ae1464d9bd3f0e4cf1280d3c1)

## What are Linux containers?
### History
* Goes back to `chroot`
	* admins needed way to hide parts of filesystem, expose only what they wanted to expose
* FreeBSD 4.0 released in 2000 with `jail` command
	* launch process with a bunch of limitations/"walls" around it
	* restrict sharing across processes etc.
* Solaris Zones (used by JoyentOS)
* 2008: linux 2.6.something: **Linux containers** (LXC)
* March 2013: Docker released

## Docker
Why did docker succeed?

* Easy(er) to use
* Simplified deployment story
	* A bunch of tools to get deps, deploy, OS version, special deps (specific version of ruby SSL e.g.), have to communicate this between teams
	* Docker allowed bundling of *all* dependencies for deployment
	* Ops just needs to manage & expose **docker host**, devs can push containers to that
* Tests more consistent, as tests are running against same env
	* Send build artifact (container) to testing env.
	* Send that *exact artifact* to production
* Abstracts software from hardware

What *isn't* docker?

* Virtualization platform (virtualbox)
* Cloud platform (AWS)
* Config management tool (Chef, Puppet)
* Deployment framework (Capistrano)
* Workload management tool (Mesos, Fleet)
* ... see [slides](https://gist.github.com/spkane/6b38360ae1464d9bd3f0e4cf1280d3c1)

### Parts of docker
* **Docker client (Docker Engine)** - The docker command used to control most of the Docker workflow and talk to remote Docker servers.
* **Docker server (Docker Engine)** - The docker command run in daemon mode. This turns a Linux system into a Docker server that can have containers deployed, launched, and torn down via a remote client.
* **Virtual Machine (Docker Machine)** - In general, the docker server can be only directly run on Linux. Because of this, it is common to utilize a Linux virtual machine to run Docker on other development platforms. Docker Machine makes this very easy.
	* *Docker Machine ~= Boot To Docker, a virtualbox docker host*.
* **Docker images** - Docker images consist of one or more filesystem layers and some important metadata that represent all the files required to run a Dockerized application. A single Docker image can be copied to numerous hosts. A container will typically have both a name and a tag. The tag is generally used to identify a particular release of an image.
	* **"This thing *represents* my application"**
	* This is a definition/class
* **Docker containers** - A Docker container is a Linux container that has been instantiated from a Docker image. A specific container can only exist once; however, you can easily create multiple containers from the same image.
	* **This is an *instance of* my application**

# Commands
## Docker Machine
Creating machine: 

```sh
docker-machine create --driver virtualbox --virtualbox-cpu-count "2" testing
```

Start **Docker *Host*** machine:

```sh
docker-machine start testing
```

Setting docker machine stuff in your local environment so you can connect to docker machine:

```sh
docker-machine env testing
# export DOCKER_TLS_VERIFY="1"
# export DOCKER_HOST="tcp://192.168.99.100:2376"
# export DOCKER_CERT_PATH="/Users/sequoia/.docker/machine/ machines/testing"
# export DOCKER_MACHINE_NAME="testing"

# to load these settings into your local env
eval $(docker-machine env testing)
# ^^ ONLY FOR THAT SHELL!!! ^^
```

## Docker
Connect to **Host** machine

```sh
docker-machine ssh testing
```

Start a **container**

```sh
# list images
docker images
# Start, remove when done, run /bin/bash (open shell)
docker run --rm -ti fedora:22 /bin/bash
# Run & `rm` when done && open TTY, Interactive, run /bin/bash
```

Run as `nobody` to reduce escalation risk (default is run container as root)

```sh
docker run -u 99 [image-name]
```

Useful alias

```
alias dm='docker-machine'
```

## `Dockerfile`

* `FROM image-name:version` : base image
* `RUN command` : run a single command
	* these create "layers" internally (*?maybe?*) that can be reused across containers
	* Put app config in environment vars...
* `ADD local-path instance-path`
* `CMD ["command", "arg", "--switch"]` :

### Layers
Remove what you can in each run step. E.G.

```
RUN yum install foo bar # caches 200MB of packages
# vs
RUN yum install foo bar && yum clear-cache # cleans up cache
```

must be on same line!!

```
RUN yum install foo bar # caches 200MB of packages
# then...
RUN yum clear-cache # HIDES cache files
```

Put most stable layers at top of file, more likely to change, lower.

### [example](https://github.com/spkane/hubot-docker/blob/master/Dockerfile)
```Dockerfile
FROM node:5.7.1

RUN mkdir -p /data/app/bin && mkdir -p /data/app/scripts

RUN apt-get -y update
RUN apt-get -y install supervisor python-pip
RUN easy_install -U pip
RUN pip install supervisor-stdout
RUN mkdir -p /var/log/supervisor

# Supervisor Configuration
ADD ./supervisord/conf.d/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
ADD ./supervisord/conf.d/hubot.conf /etc/supervisor/conf.d/hubot.conf

ADD ./bin /data/app/bin
ADD ./scripts /data/app/scripts
ADD ./*.json /data/app/

RUN cd /data/app && npm install

CMD ["supervisord", "-n"]

```