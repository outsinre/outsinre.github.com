---
layout: post
title: Docker newbie
---

* ToC
{:toc}

# [ABCs](https://yeasy.gitbooks.io/docker_practice/content/introduction/what.html)

1. Application layer isolation.

   It is not virtual machine. Docker just provides application dependencies while VM virtualizes a whole bunch of hardware and OS.
2. Dockers comprises *image*, *container* and *registry*.
   1. Image is *static* dependencies like a minimal root filesystem, a daemon etc. There are many highly qualified base iamge from official registry like *nginx*, *redis*, *php*, *python*, *ruby* etc. Especially, we have *ubuntu*, *centos*, etc. that are just OS minimal bare bones (like Gentoo stage tarball).
   2. Container is *running* instance with namespace - the isolated application process. We can think of image and container as class and object in Object-oriented programming. 
   3. Registry is online *store* where users public, share *repostitory* which comprises images of different versions. We use *registry* and *repository* interchangebly.
3. C/S mode.
   1. Client: user command line (i.e. *docker image ls*)
   2. Server: local/remote *docker-engine* (i.e. *systemctl start docker*).
4. [Layer storage](https://docs.docker.com/storage/storagedriver/) uses Union FS (recall that Live CD on USB stick requires Union FS). Only the topmost (container storage layer) is writable and volatile.

   Storage Driver prefers *overlay2* over *aufs*. Either enable *overlay2* in kernel or build external module.

   Pay attention to [CentOS/RHEL 的用户需要注意的事项](https://yeasy.gitbooks.io/docker_practice/content/image/rm.html#centosrhel-%E7%9A%84%E7%94%A8%E6%88%B7%E9%9C%80%E8%A6%81%E6%B3%A8%E6%84%8F%E7%9A%84%E4%BA%8B%E9%A1%B9) if *devicemapper* driver *loop-lvm* mode is used.
5. Technically, Dockerfile defines image construction.

# info

```bash
user@tux ~ $ docker info
user@tux ~ $ docker image/container ls [-a]
user@tux ~ $ docker inspect [ image ID | container ID | network ID ]
user@tux ~ $ docker search ubuntu
```

1. We can not *search* tags of an image. Find them on registry web page instead.

# run

```bash
user@tux ~ $ docker pull ubuntu:16.04
user@tux ~ $ docker image ls ubuntu
user@tux ~ $ docker run -it --rm ubuntu:16.04 bash
#
user@docker ~ $ cat /etc/os-release
user@docker ~ $ exit
#
user@tux ~ $ docker image rm ubuntu:16.04
user@tux ~ $ docker image prune
```

1. *run* creates a container. `--rm` deletes it upon exit without which we can delete manually (discussed next).

   By default, container object remains even it stops.

# container

```bash
user@tux ~ $ docker run --name webserver -d -p 8080:80 nginx
user@tux ~ $ docker container ls -a
user@tux ~ $ docker container logs webserver
user@tux ~ $ docker container stop/kill webserver
user@tux ~ $ docker container rm webserver
user@tux ~ $ docker container prune
```

1. `-d` runs container in daemon mode. `-p` maps host port 8080 to container port 80 that is already bound to host Nginx process.
2. Visit the Nginx container page at *http://ip:8080*.
3. *stop* attempts to trigger a [*graceful*](https://superuser.com/a/757497) shutdown by sending the standard POSIX signal SIGTERM, whereas *kill* just kills the process by default.

# Get into container

*exec* or *attch* a command in a running container. Avoid *attch* as much as possible because it stops container after exit. Alternatively, add `-it` to *run* getting interactive shell.

```bash
user@tux ~ $ docker exec -it webserver bash
#
user@docker ~ $ echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
user@docker ~ $ exit
#
user@tux ~ $ docker diff webserver
user@tux ~ $ docker commit -a 'jim' -m 'change front page' webserver nginx:v2
user@tux ~ $ docker image ls nginx
user@tux ~ $ docker history nginx:v2
```

1. We modified container layer storage. Use *diff* to check details.
2. Visit the Nginx container page again.
3. *commit* records modification as new image layer.

   >Avoid *commit* command as miscellaneous operations (garbage) are recorded either.

To verify the new image:

```bash
user@tux ~ $ docker run --name web2 -d -p 8081:80 --rm nginx:v2
```

# Build Dockerfile

From *commit* example, we can create new image layer though many negligible commands like *ls*, *pwd*, etc are recourded as well.

Similar to Makefile, Docker uses Dockerfile to define image with specified *instruction*s like *FROM*, *COPY* etc. Each instruction defines a layer. In order to minimize image number of layers and maintain clear logics, we can merge instructions.

In this section, we use Dockerfile to create image *nginx:v2*.

```bash
user@tux ~ $ mkdir mynginx
user@tux ~ $ cd mynginx
user@tux ~ $ vim Dockerfile
#
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

Just two lines! FROM imports the base image on which we will create new layer. If we do not want any base image, use the special null image *scratch*.

Usually, in the end of image, *CMD* or *ENTRYPOINT* instruction sets commands and arguments. For example, *nginx* image has `CMD ["nginx", "-g", "daemon off;"]`. We can pass custom commands and arguments when invoking *docker run*.

Now we build the image:

```bash
user@tux ~ $ docker build -t nginx:v3 .
#
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM nginx
 ---> b175e7467d66
Step 2/2 : RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
 ---> Running in d5baea5c6341
Removing intermediate container d5baea5c6341
 ---> 18cc3a3480f0
Successfully built 18cc3a3480f0
Successfully tagged nginx:v3
```

1. During the building process, an intermediate container is launched for RUN instruction.
2. The tailing dot means building *context* directory. It is also Dockerfile's default location.

   Docker sends all files within context directory to remote Docker engine (daemon).

   >Context directory is NOT the *pwd* where we execute *docker build* command though usually we switch to it before building.

After the building, we can run *nginx:v3* image:

```bash
user@tux ~ $ docker run --name web3 -d -p 8081:80 --rm nginx:v3
```

# Network

![docker netowrk](/assets/docker-net.png)

```bash
user@tux ~ $ docker network create -d bridge mynet
user@tux ~ $ docker run -dit --name ubt1604 -v /src/hostdir:/opt/condir:rw --network mynet ubuntu:16.04
user@tux ~ $ docker network ls
user@tux ~ $ docker network inspect mynet
```

1. Docker automates network configuration upon container startup. We can customize that on purpose.
