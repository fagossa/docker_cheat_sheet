Docker
==========


## Basic mechanism

Definition of [Copy-on-write](http://en.wikipedia.org/wiki/Copy-on-write):

> Copy-on-write (sometimes referred to as "COW") is an optimization strategy used in computer programming. Copy-on-write stems from the understanding that when multiple separate tasks use initially identical copies of some information (i.e., data stored in computer memory or disk storage), treating it as local data that they may occasionally need to modify, then it is not necessary to immediately create separate copies of that information for each task


and [System Level Virtualization](http://en.wikipedia.org/wiki/Operating-system-level_virtualization): 

> Operating-system-level virtualization is a server virtualization method where the kernel of an operating system allows for multiple isolated user space instances, instead of just one. Such instances (often called containers, virtualization engines (VE), virtual private servers (VPS), or jails) may look and feel like a real server from the point of view of its owners and users.

and google's [cgroups](http://en.wikipedia.org/wiki/Cgroups):

> cgroups (abbreviated from control groups) is a Linux kernel feature that limits, accounts for and isolates the resource usage (CPU, memory, disk I/O, network, etc.) of a collection of processes.


## Run Commands


### Commands
To run an instance

	docker run busybox echo hello world
	
To run and name an image

	docker run --name ticktock jpetazzo/clock

To rename a running container [Since 1.5]

	docker rename ticktock tocktuck

To show the history of an image

	docker history busybox

To show the layer tree

	docker images --tree

To run an image and execute bash

	docker run -it ubuntu bash

(_Not related to Docker_) To get a public ip from an external site

	curl ifconfig.me/ip

To get the last running container

	docker ps -l

To detach me from a running instance

	^p^q

### Logs Commands

To attach and show logs

	docker logs --tail 3 47d62243sdsdf

To attach and show logs and continouslly updates

	docker logs --tail 1 --follow 47d62243sdsdf

### Stop/detach Commands

To stop a detached container with the possibility to startit later
	
	docker kill 47d62243sdsdf

ou

	docker stop 47d62243sdsdf

To restart an stoped container 

	docker start 47d62243sdsdf
	
Alternative to attach to a running container	
	
	docker exec -it 47d62243sdsdf bash	

To definitelly kill a docker instance
	
	docker rm 47d62243sdsdf


### Docker exec [docker 1.5]

The exec command runs a new command in a __running container__

	docker exec -it 47d62243sdsdf bash


## Images

To list images

	docker images -a


To create an image from an existing container

	docker commit

To create an image from a docker file
	
	docker build

To load a tarball into docker
	
	docker import

### Namespaces

* root-like `ubuntu`
* user `jpetazzo/clock`
* self-hosted `registry.toto.com:5000/my-private-image`

### Searches

Searches images in docker hub

	docker search zookeeper

However, this search does not do a pattern matching and in generel is kind of bad.

### Diff between containers

Searches images in docker hub

	docker diff 47d62243sdsdf

There are 3 events that are listed in the diff:

* A - Add
* D - Delete
* C - Change

To name a non named image

	docker tag <newImageId> mydistro


---

## Docker files

Creating a first docker image that does nothing

	FROM ubuntu
	RUN apt-get update
	RUN apt-get install -y wget

To build the image

	docker build -t myimage .

To build the image without cache

	docker build --no-cache -t myimage .

> Each __RUN__ creates an intermediate image however __FROM__ does not, it's just a starting point.  

To create an instance of the newly created image
	
	docker run -it myimage

To check the different layers of my image

	docker history myimage


### CMD vs ENTRYPOINT


#### CMD

Define the __default__ command in the docker file. And __always__ executed at the end. If there are several other CMS, only the last one is executed.

	FROM ubuntu
	RUN apt-get update
	RUN apt-get install -y wget
	CMD wget -O- -q http://ifconfig.me/ip

To build it

	docker build -t ifconfigme .

To runnit

	docker run ifconfigme


#### Entrypoint

Allow to override the default CMD speficied in the *Dockerfile*

	FROM ubuntu
	RUN apt-get update
	RUN apt-get install -y wget
	ENTRYPOINT ["wget", "-O-", "-q"]
	
>in this case ENTRYPOINT gets a JSON to avoid __bash__ to execute the command. And it does not make ENV variable substitutions.


To run the image

	docker run ifconfigme http://ifconfig.me/ua
	 
> Like CMD, ENTRYPOINT can appear anywhere, and replaces the previous value


#### Combining CMD + ENTRYPOINT


	FROM ubuntu
	RUN apt-get update
	RUN apt-get install -y wget
	ENTRYPOINT ["wget", "-O-", "-q"]
	CMD http://ifconfig.me/ip

By default it will call the _wget_ on _http://ifconfig.me/ip_

	docker run ifconfigme
	
It is also possible to override the default value

	docker run ifconfigme http://ifconfig.me/ua
	
	
### COPY vs ADD

#### COPY

Define an external file

	int main () {
	  puts("Hello, world!");
	  return 0;
	}

To build and include the image

	FROM ubuntu
	RUN apt-get update
	RUN apt-get install -y build-essential
	COPY hello.c /
	RUN make hello
	CMD /hello

To build an image

	docker build -t hello .

To runnit

	docker run hello

#### ADD

> This is an older version of _COPY_ that could potencially exclude files. Usefull for images whose 
> parent is __scratch__.

### Advanced Dockerfiles instructions

* __FROM__,	We can have several instructions
	
	```
	FROM ubuntu:14.04
	FROM fedora:20
	```
* __MAINTAINER__, Name of the person in charge
	
	```
	MAINTAINER Maintainer Group maintainer@group.com
	```
* __RUN__, install packages, execute processes inside the container. It won't automatically start daemons
	
	```
	RUN apt-get update // shell format, executes a shell which then executes the command
	RUN ["apt-get", "update"] // Json format, executes the command directly
	RUN [ "sh", "-c", "echo", "$HOME" ]
	```
	To execute processes you must use `CMD` or `ENTRYPOINT`
* __EXPOSE__, list the ports getting out of the container. By default it's explosed in `TCP`

	```
	EXPOSE 8080
	```
* __ADD__, extracts automatically compressed files if needed. It also download files but there is no cache, so it downloads those files every single time.

* __USER__, Sets the username or UID. It is herited from parent images.

* __ONBUILD__, is only useful for images that are going to be built FROM a given image. For example, you would use ONBUILD for a language stack image that builds arbitrary user software written in that language within the Dockerfile, as you can see in Ruby’s ONBUILD variants. Images built from ONBUILD should get a separate tag, for example: `ruby:1.9-onbuild` or `ruby: 2.0-onbuild`.

	```
	ONBUILD COPY . /go/src/app
	```

### Tools

To verify the `Effective Dockerfile` this [project](https://github.com/CenturyLinkLabs/dockerfile-from-image) could be used. To verify the `image hirarchy` there is [another project](https://github.com/CenturyLinkLabs/docker-image-graph).


There is a project called [docker-gc](https://github.com/spotify/docker-gc) to clean unused docker images.


The default configuration for the docker agent is found in 

	sudo vim /etc/default/docker

It allows to configure _http proxies_

---

## Other Commands

### Inspect

Inspect a running instance and coloring

	docker inspect ticktock | jq .

Getting a json attribute

	docker inspect --format '{{ json .Created }}' ticktock

---

## Port mapping

Running an image in background using _default port mapping_

	docker run -d -P jpetazzo/web

* `P`, expose all ports inside container
* `d`, daemon mode

To identify how the port is mapped in the host

	docker port a743653ed1c1 8000

Explicit port mapping

	docker run -t -p 80:8000 jpetazzo/web

Identify the private ip address of a container

	docker inspect --format '{{ .NetworkSettings.IPAddress }}' <yourContainerID>

Daemon (docker-proxy) to handle the network mapping per instance,

	ps aux | grep docker-proxy

To verify the current __iptables__ config
	
	sudo iptables -L


	Chain      DOCKER (1 references)
	target     prot opt source               destination
	ACCEPT     tcp  --  anywhere             ip-172-17-0-20.eu-central-1.compute.internal  tcp dpt:8000
	ACCEPT     tcp  --  anywhere             ip-172-17-0-23.eu-central-1.compute.internal  tcp dpt:8000


## Volumes

To execute an image that shares a volume between a host and the docker instance

	docker pull training/namer

	docker run -d \
      -v $(pwd):/opt/namer \
      -p 80:9292 \
      training/namer

* `v`, volume using the format `absolute_host_path:container_path`
* `p`, port mappings

Two alternatives to use volumes:

* Within a Dockerfile, with a _VOLUME_ instruction.

	`VOLUME /var/lib/postgresql`

* Directly in the __run__, however this volume is not shared with the host

	`docker run -d -v /var/lib/postgresql \
		training/postgresql	`

> None of the  elements inside a volume are handled by __docker commit__

It is also possible to share volumes from named instances by using the flag `--volumnes-from`

	docker run -it --name alpha -v /var/log ubuntu bash
	root@99020f87e695:/# date >/var/log/now

In another terminal, let's start another container with the same volume.

	docker run --volumes-from alpha ubuntu cat /var/log/now
	Fri May 30 05:06:27 UTC 2014

The volume data is stored nevertheless in the host in the `volumes` directory of the installation directory

	docker@52.28.55.218 ~: ls -l /mnt/docker
	total 56
	drwxr-xr-x  5 root root  4096 Apr 13 07:09 aufs
	drwx------  3 root root  4096 Apr 14 08:03 containers
	drwx------  3 root root  4096 Apr 13 07:09 execdriver
	drwx------ 84 root root 12288 Apr 13 14:46 graph
	drwx------  2 root root  4096 Apr 13 07:09 init
	-rw-r--r--  1 root root  5120 Apr 14 08:03 linkgraph.db
	-rw-------  1 root root  1490 Apr 13 15:20 repositories-aufs
	drwx------  2 root root  4096 Apr 13 13:55 tmp
	drwx------  2 root root  4096 Apr 13 07:09 trust
	drwx------  3 root root  4096 Apr 14 08:00 vfs
	drwx------  5 root root  4096 Apr 14 08:03 volumes

It is important to get scripts to clean `vfs`and `volumes` directories. It is also possible to remove used volumes in a delete hook using the flag `--rm` in the `run` and the `rm` commands.

### Data containers

> A data container is a container created for the sole purpose of referencing one (or many) volumes.

It is typically created with a no-op command:

	docker run --name wwwdata -v /var/lib/www busybox true
	docker run --name wwwlogs -v /var/log/www busybox true

* We created two data containers.
* They are using the busybox image, a tiny image.
* We used the command true, possibly the simplest command in the world!
* We named each container to reference them easily later.
* `true` is a commmand that does nothing

To use those containers (Note: Obviously those images don't exist)

	docker run -d --volumes-from wwwdata --volumes-from wwwlogs webserver
	docker run -d --volumes-from wwwdata ftpserver
	docker run -d --volumes-from wwwlogs pipestash

* The first container runs a webserver, serving content from __/var/lib/www__ and logging to __/var/log/www__.
* The second container runs a FTP server, allowing to upload content to the same __/var/lib/www__ path.
* The third container collects the logs, and sends them to logstash, a log storage and analysis system.

To mount volumes in Read-Only you must use the `ro` flag

	docker run -it -v $(pwd)/bindthis:/var/www/html/webapp:ro ubuntu

## Ambassadors

> Standard mechanism to allow instance communication among different host. However, this is just an adhoc implementation and not production ready. A more serious approach could be based on __HaProxy__.

From host 1

	docker run -P -d redis

From host 2
	
	docker run -d --name regisamba jpetazzo/hamba 6379 <host1> 49154
	docker run -p 8000:3000 -d --link regisamba:redis nathanlechaire/redisonrails


## Fig (aka Docker compose)

> Compose is a tool for defining and running complex applications with Docker. With Compose, you define a multi-container application in a single file, then spin your application up in a single command which does everything that needs to be done to get it running.

For example a `docker-compose.yml` could be as follows

	web:
	  build: .
	  links:
	   - db
	  ports:
	   - "8000:8000"
	db:
	  image: postgres

this file could also be executed by the following command 

	docker-compose up // alias to docker up

To execute and list the instance from the _fig_ file

	fig up // build and run the instances
	fig ps // list running instances

To execute several times a running fig file, however there the port mapping should not include the host port (e.g. `ports: - "5000"`)

	fig up
	fig scale web=2

To stop the running instances

	fig stop //graceful, 10 seconds
	fig kill //destroy all containers
	fig rm //kills and removes all containers



