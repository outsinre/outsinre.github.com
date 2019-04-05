---
layout: post
title: Docker newbie
---

1. toc
{:toc}

# [ABCs](https://yeasy.gitbooks.io/docker_practice/content/introduction/what.html)

1. Application layer isolation.

   It is not and lighter than virtual machine. Docker just provides application dependencies while VM virtualizes everything including hardware and OS.
2. Dockers comprises *image*, *container* and *registry*.
   1. Image is *static* and *readonly* like a minimal root filesystem, a daemon etc. There are many highly qualified base iamge from official registry like *nginx*, *redis*, *php*, *python*, *ruby* etc. Especially, we have *ubuntu*, *centos*, etc. that are just OS minimal bare bones (like Gentoo stage tarball).

      An image consists of multiple filesystem layers.
   2. Container is created on top of an image with the topmost filesystem layer storing *running* data. Processes within different containers are isolated - namespace.

      We can think of image and container as class and object in Object-oriented programming. 
   3. Registry is *store* where users publicize, share and download *repostitory* which comprises different images of the same name. We will find different image versions are referenced to by *tag* or *digest*. The official registry is *docker.io* with a frontend website [Docker Hub](https://hub.docker.com).
3. C/S mode.
   1. Client: user command line (i.e. *docker image ls*)
   2. Server: local/remote *docker-engine* (i.e. *systemctl start docker*).
4. [Layer storage](https://docs.docker.com/storage/storagedriver/) uses Union FS (recall that Live CD on USB stick requires Union FS). Only the topmost (container storage layer) is writable and volatile.

   Of the Union FS, *overlay2* is recommended over *aufs*. Either enable *overlay2* in kernel or build external module. *devicemapper* is also used in CentOS/RHEL. Pay attention to [CentOS/RHEL 的用户需要注意的事项](https://yeasy.gitbooks.io/docker_practice/content/image/rm.html#centosrhel-%E7%9A%84%E7%94%A8%E6%88%B7%E9%9C%80%E8%A6%81%E6%B3%A8%E6%84%8F%E7%9A%84%E4%BA%8B%E9%A1%B9) if *devicemapper* driver *loop-lvm* mode is used.
5. Technically, Dockerfile defines image construction.

# Daemon

```bash
~ # systemctl enable docker
~ # systemctl status docker
~ # systemctl start docker
```

The daemon manages everything!

# [Metadata](https://docs.docker.com/engine/reference/commandline/docker/)

```bash
root@tux ~ # fgrep -qa docker /proc/1/cgroup; echo $?                    # check if it is a docker
root@tux ~ # docker info                                                 # display the outline of docker environment
root@tux ~ # docker image/container ls [-a]                              # list images/containers
root@tux ~ # docker [image] history                                      # show layers of an image
root@tux ~ # docker inspect [ name | ID ]                                # display low-level details on any docker objects
```

# Pull

```bash
root@tux ~ # docker search ubuntu                                        # search docker images by name on Docker Hub
root@tux ~ # docker pull ubuntu:16.04                                    # specify a tag
root@tux ~ # docker pull ubuntu@<sha256>
root@tux ~ # docker image ls ubuntu
```

1. *docker search* search docker images from registries defined in */etc/container/registries.conf*.

   An image name is composed of three parts:

   ```
   registry.domain[:port]/[user/]name
   ```

   The search command line does not print imange tags or digests (SHA256 hash). Instead, go to the registry website or check third party tool [DevOps-Python-tools](https://github.com/HariSekhon/DevOps-Python-tools).
2. By default, if only a name is specified, *pull* uses tag *latest* (i.e. 'ubuntu:latest'). Otherwise, provide either a specified tag (i.e. *ubuntu:16.04*) or digest (i.e. 'ubuntu@<sha256-value>').

   The official registry domain *docker.io* can be omitted.
3. When pulling an image without its digest, we can update the image with the same *pull* command again.

   On the contrary, with digest, the image if fixed and pinned to that exact version. This makes sure you are interacting with the exact image. However, upcoming security fixes are also missed.

   To get image digest, we should firstly pull down a image, and use *inspect* list digests included.
4. Any any time, Ctrl-C terminates the pull process.

# [Run](https://docs.docker.com/engine/reference/run/) an Image and Create a Container

Syntax:

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

Example:

```bash
root@tux ~ # docker run --name test-ubuntu \
-i --rm \
-t \
-v /root/workspace:/root/workspace:rw \
-w /root/workspace/ \
-u $(id -u):$(id -g) \
ubuntu:16.04 \
bash

#
root@docker ~ # cat /etc/os-release
root@docker ~ # exit 13
root@docker ~ # echo $?
```

1. When we run an image, a container is created.
2. `-i --rm` runs interactively and automatically remove the container when it exits.
3. `-t` allocates a pseudo-TTY.
4. `-w` lets the COMMAND (i.e. *bash*) be executed inside the given directory (created on demand).
5. `-u --user` runs the image as a non-root user. Attention that, the username is that within the container. So the image creator should create that name in Dockerfile.

# Data Share

Docker containers can read from or write to pathnames, either on host or on memory filesystem - to share data. There are three types:

1. Volume, Data Volume, or Named Volume.

   By default, running data of a container is layered on top of the image used to create it. A volume decouples that data from both the host or the container. Just think of a Windows partition or removable disk drive.

   Volumes are managed by Docker and persist. Data within can be shared among multiple containers, as well as between the host and a container.
2. Bind Mount.

   Bind a file or a mount directory of the host within a running container. The target can be read-only or read-write. For example, bind host */etc/resolv.conf* to a container, sharing name servers.
3. Tmpfs.

   Needless to say, *tmpfs* is a memory filesystem.
4. The `--volume , -v` or `--mount` can be used. `--mount` is more powerful and recommended though `--volume, -v` won't be deprecated.

   In the example above, the source directory */root/workspace* will be created on demand.

# Manage a Nginx Container

```bash
root@tux ~ # docker run --name webserver \
-d \
--net host
--mount type=bind,source=/tmp/logs,destination=/var/log/nginx \
-p 8080:80 nginx

root@tux ~ # docker container ls
root@tux ~ # docker container logs webserver
root@tux ~ # docker container stop/kill webserver
root@tux ~ # docker container start webserver
root@tux ~ # docker container rm webserve          # remove one or more container (even running instances)
root@tux ~ # docker container prune                # remove all stopped container
```

1. `-d` runs container in detached mode and terminal is released immediately. `-p` maps host port 8080 to container port 80 that is already bound to host Nginx process.
2. Check the Dockerfile, there is a line telling how Nginx should be started:

   ```
   CMD ["nginx", "-g", "daemon off;"]
   ```

3. The `--mount` type is a Bind Mount directory.
4. Visit the Nginx container page at *http://host-ip:8080*.
5. *stop* attempts to trigger a [*graceful*](https://superuser.com/a/757497) shutdown by sending the standard POSIX signal SIGTERM, whereas *kill* just kills the process by default.

# Get into container

*exec* or *attch* sends a command into a running container. However, avoid *attch* as much as possible because it stops container after exit. Most ofen, we add `-it` to *run* or *exec*, getting an interactive shell.

```bash
root@tux ~ # docker exec -it webserver bash
#
root@docker ~ # echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
root@docker ~ # exit
#
root@tux ~ # docker diff webserver
root@tux ~ # docker commit -a 'jim' -m 'change front page' webserver nginx:v2
root@tux ~ # docker image ls nginx
root@tux ~ # docker history nginx:v2
```

1. We modified container layer storage. Use *diff* to check details.
2. Visit the Nginx container page again.
3. *commit* records modification as a new layer and create a new image.

   Avoid *commit* command as miscellaneous operations (garbage) are recorded either. As discussed in "Data Share" section, we can make use of Volume, Bind Mount, or tmpfs, or resort to 'Build Image by Dockerfile' section below.
4. To verify the new image:

   ```bash
   root@tux ~ # docker run --name web2 -d -p 8081:80 --rm nginx:v2
   ```

   The `--rm` tells to remove the container upon exit.

# [Networking Drivers](https://docs.docker.com/network/)

The Docker's networking subsystem is *pluggable*, using drivers. Below is a simple explanation:

1. Bridge

   The *default* driver if none is given upon *run*, providing network isolation from the outside. It is to bridge traffic among multiple containers and the host. Check the image below, *docker0" is the bridge interface.

   ![docker netowrk](/assets/docker-net.png)

   Please pay attention: this is different from the *bridge* mode of VMWare or VirtualBox (real Virtual Machine). VMWare and VirtualBox's bridge is deployed directly on the host's interface and appears to be a real physical device parallel to the host and can be connected to directly from within LAN.
2. Host

   Share the host's networking directly without isolation. However, LAN devices cannot differntiate between containers and the host.
3. Overlay

   Overlay connects multiple Docker daemons together, creating a distributed network among multiple Docker daemon hosts. This network sits on top of (overlays) the host-specific networks, allowing containers connected to it.
4. macvlan

   As the name implied, maclan assignes a MAC address to a container, making it be a physical device on the same network as the host - counterpart of VMWare/VirtualBox's 'bridge'.
5. none

   Disable networking.

## Manual Network Configuration

Docker automates network configuration upon container startup. We can customize that on purpose.

```bash
root@tux ~ # docker network ls
root@tux ~ # docker network create -d bridge mynet
root@tux ~ # docker run -it --name ubt1604 -v /src/hostdir:/opt/condir:rw --network mynet ubuntu:16.04
root@tux ~ # docker network inspect mynet
```

Use the `--net` or `--network` option. To the 'host' networking driver, just pass `--net host` option to *docker run*.

# Build Image by Dockerfile

From *commit* example above, we can create new image layer but many negligible commands like *ls*, *pwd*, etc. are recourded as well.

Similar to Makefile, Docker uses Dockerfile to define image with specified *instruction*s like FROM, COPY, RUN etc. Each instruction defines a layer. In order to minimize image number of layers and maintain clear logics, we can merge instructions.

In this section, we use Dockerfile to create image *nginx:v2*.

```bash
root@tux ~ # mkdir mynginx
root@tux ~ # cd mynginx
root@tux ~ # vim Dockerfile
#
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

Just two lines! FROM imports the base image on which we will create the new layer. If we do not want any base image, use the special null image *scratch*.

Usually, in the end of image, [exec form or sh (/bin/sh) form](https://www.cnblogs.com/sparkdev/p/8461576.html) instruction sets commands and arguments. *docker container inspect* show the default commands and arguments. For example, *nginx* image has `CMD ["nginx", "-g", "daemon off;"]`. We can pass custom commands and arguments when invoking *docker run*. Apart from the difference between *exec* form and *sh* form, there are command instructions like [RUN, ENTRYPOINT and CMD](http://goinbigdata.com/docker-run-vs-cmd-vs-entrypoint/). ENTRYPOINT configures a container that will run as an executable in that we can pass explicit arguments when running the container, making it looks like a normal command. The RUN instruction prefers the *sh* form while ENTRYPOINT and CMD prefer the *exec* form. If */bin/bash* is preferred to */bin/sh*, then choose the *exec* form like `["/bin/bash", "-c", "echo 'Hello, world.'"]`.

Now we build the image:

```bash
root@tux ~ # docker build -t nginx:v3 .
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

1. During the building process, an intermediate container is created for 'RUN' instruction, adding a new layer.

   The new layer is then automatically committed.
2. The trailing dot means building *context* directory. It is also Dockerfile's default location.

   Docker sends all files within context directory to remote Docker engine (daemon). Context directory is NOT the PWD where we execute *docker build* command though usually we switch to it before building.
3. After the building, we can run *nginx:v3* image:

   ```bash
   root@tux ~ # docker run --name web3 -d -p 8081:80 --rm nginx:v3
   ```

Apart from builing a new docker image for the web server, we can utilize 'Data Share' to attach a Volume or Bind Mount to the base docker image. Build the web server within the attached storage instead.
