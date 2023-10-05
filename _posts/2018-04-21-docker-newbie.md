---
layout: post
title: Docker Newbie
---

1. toc
{:toc}

# ABCs #

1. Application layer isolation.

   It is not and lighter than virtual machine. Docker just provides application dependencies while VM virtualizes everything including hardware and OS.
2. Dockers comprises *image*, *container* and *registry*.
   1. Image is *static* and *readonly* like a minimal root filesystem, a daemon etc. There are many highly qualified base iamge from official registry like *nginx*, *redis*, *php*, *python*, *ruby* etc. Especially, we have *ubuntu*, *centos*, etc. that are just OS minimal bare bones (like Gentoo stage tarball).

      An image consists of multiple filesystem layers. We can define an image by Dockerfile.
   2. Container is created on top of an image with the topmost filesystem layer storing *running* data. Processes within different containers are isolated - namespace.

      We can think of image and container as class and object in Object-oriented programming. 
   3. Registry is *store* where users publicize, share and download *repostitory*. The default registry is *docker.io* with a frontend website [Docker Hub](https://hub.docker.com).
   
      Repository, on the other hand, actually refers to *name* of an image (e.g. *ubuntu*). We can specify *version* of a repository by a *tag* (label) like *ubuntu:16.04* (colon separator). The default tag is *latest*.
3. Naming of an image

   ```
   registry.fqdn[:port]/[user/]repository[:tag | @<image-ID>]
   ```

   1. Default registry can be ommitted.
   2. The *user* part means a registered user account in the registry.
   3. *repository* is the default *name* of an image.
   4. *image id* comprises a SHA256 *digest* like *ubuntu@abea36737d98dea6109e1e292e0b9e443f59864b* (at sign separator).
4. C/S mode.
   1. Client: user command line (i.e. *docker image ls*)
   2. Server: local/remote *docker-engine* (i.e. *systemctl start docker*).
5. [Layer storage](https://docs.docker.com/storage/storagedriver/) uses Union FS (recall that Live CD on USB stick requires Union FS). Only the topmost (container storage layer) is writable and volatile.

   Of the Union FS, *overlay2* is recommended over *aufs*. Either enable *overlay2* in kernel or build external module. *devicemapper* is also used in CentOS/RHEL. Pay attention to [CentOS/RHEL 的用户需要注意的事项](https://yeasy.gitbooks.io/docker_practice/content/image/rm.html#centosrhel-%E7%9A%84%E7%94%A8%E6%88%B7%E9%9C%80%E8%A6%81%E6%B3%A8%E6%84%8F%E7%9A%84%E4%BA%8B%E9%A1%B9) if *devicemapper* driver *loop-lvm* mode is used.

# Installation #

We only [install](https://docs.docker.com/engine/install/) Docker CE version. For Windows and MacOS, Docker Desktop includes both Docker and [Docker Compose](#docker-compose). On Linux, it is highly recommended to install Docker and Docker Compose by [official repo](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository), or install by [downloaded packages](https://docs.docker.com/engine/install/ubuntu/#install-from-a-package).

On Amazon Linux 2, we can install Docker by package manager, and then install Docker Compose by [downloading binary file manually](https://docs.docker.com/compose/install/#install-the-binary-manually).

Install Docker by package manager:

```bash
~ $ sudo yum update -y

# CentOS
~ $ sudo yum install docker

# Amazon Linux 2 - https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-container-image.html
~ $ sudo amazon-linux-extras install docker

~ $ docker version
```

In order to run *docker* as a normal user, add the account to *docker* group:

```bash
~ $ sudo usermod -aG docker <username>

~ $ reboot
```

Install Docker Compose manually:

```bash
~ $ sudo mkdir -p /usr/local/lib/docker/cli-plugins
~ $ sudo curl -sSL https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/lib/docker/cli-plugins/docker-compose

~ $ sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
~ $ docker compose version

~ $ PATH="/usr/local/lib/docker/cli-plugins:$PATH"
~ $ docker-compose version
```

# Daemon #

Start Docker:

```bash
~ $ sudo systemctl enable docker
~ $ systemctl status docker
~ $ sudo systemctl start docker

~ $ docker info
~ $ docker ps
~ $ docker compose ls
```

The daemon manages everything!

# docker.sock #

Docker daemon creates a Unix socket file at [/var/run/docker.sock](https://stackoverflow.com/q/35110146/2336707) by which we can communicate with the daemon via RESTful API.

```bash
~ $ curl --unix-socket --no-buffer /var/run/docker.sock http://localhost/events
~ $ curl --unix-socket /var/run/docker.sock http://localhost/version
~ $ curl --unix-socket /var/run/docker.sock http://localhost/images/json | jq
~ $ curl --unix-socket /var/run/docker.sock http://localhost/containers/json | jq
```

We can also create containers inside another container by [mounting](#data-share) the socket file, as long as [docker CLI is available](https://askubuntu.com/a/1388299/226303).

```bash
# on host
~ $ docker run --name docker-sock --rm -it -v /var/run/docker.sock:/var/run/docker.sock docker sh

# within docker-sock
/ # docker run --name docker-in-docker --rm -it ubuntu bash

# within docker-in-docker
root@978d8704d548:/#
```

On the host terminal, we can retrieve the *nested* containers

```bash
~ $ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS           NAMES
978d8704d548   ubuntu    "bash"                   5 seconds ago    Up 4 seconds                    docker-in-docker
a3b93da63e90   docker    "dockerd-entrypoint.…"   17 seconds ago   Up 16 seconds   2375-2376/tcp   docker-sock
```

We can also get all containers from within *docker-sock*.

```bash
/ # docker ps -a
CONTAINER ID   IMAGE                    COMMAND                  CREATED         STATUS                       PORTS                NAMES
a3b93da63e90   docker                   "dockerd-entrypoint.…"   4 minutes ago   Up 4 minutes                 2375-2376/tcp        docker-sock
0dadcbdeb804   862614378b4c             "docker-entrypoint.s…"   13 months ago   Exited (0) 13 months ago                          eloquent_jones
f3f0faf83ac5   docker/getting-started   "/docker-entrypoint.…"   14 months ago   Exited (255) 13 months ago   0.0.0.0:80->80/tcp   romantic_colden
```

However, mounting *docker.sock* would make your host [vulnerable to attack](https://dev.to/pbnj/docker-security-best-practices-45ih#docker-engine) as Docker daemon is ran as *root*.

# CLI Sample #

```bash
~ $ fgrep -qa docker /proc/1/cgroup; echo $?                    # check if it is a docker
~ $ docker info                                                 # display the outline of docker environment
~ $ docker image/container ls [-a]                              # list images/containers
~ $ docker [image] history                                      # show layers of an image
~ $ docker inspect [ name | ID ]                                # display low-level details on any docker objects
~ $ docker logs <container>
```

Here is the full list of docker CLI: [Docker CLI](https://docs.docker.com/engine/reference/commandline/docker/).

# Pull Images #

It is highly recommended to *pull* the [docker/getting-started](https://hub.docker.com/r/docker/getting-started) image, *run* and visit `http://localhost`.

```bash
~ $ docker search -f is-official=true ubuntu                    # search only official image

~ $ docker pull ubuntu:16.04                                    # specify a tag
~ $ docker pull ubuntu@sha256:<hash>                            # specify an image ID

~ $ docker pull amd64/amazonlinux                               # AMD64
~ $ docker pull arm64v8/amazonlinux                             # Apple M1 ARM64

~ $ docker images
```

1. *docker search* search docker images from registries defined in */etc/container/registries.conf*.

   Unfortunately, it does *not* suport tags or IDs. Instead, go to the registry website or check third party tool [DevOps-Python-tools](https://github.com/HariSekhon/DevOps-Python-tools).
2. We can offer *docker pull* a tag (i.e. 'ubuntu:latest') or an *image ID*.
3. When pulling an image without its digest, we can update the image with the same *pull* command again.

   On the contrary, with digest, the image is fixed and pinned to that exact version. This makes sure you are interacting with the exact image. However, upcoming security fixes are also missed.

   To get image digest, we either go to the official registry, use `images --digests`, or even *inspect* command.
4. Any any time, Ctrl-C terminates the pull process.
5. Docker support [proxy configuration](https://docs.docker.com/network/proxy/) when feteching the images.

# Launch Containers #

Syntax:

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

Example:

```bash
~ $ docker run --name centos-5.8 \
-d \
-it \
--rm \
--mount type=bind,src=/home/jim/workspace/,dst=/home/jim/workspace/,ro \
-w /home/jim/workspace/ \
--net host
-u $(id -u):$(id -g) \
7a126f3dba08 \
bash

#
root@docker ~ # cat /etc/os-release
root@docker ~ # exit 13
root@docker ~ # echo $?
```

1. When we run an image, a container is created with an extra layer of writable filesystem.
2. To be compatible with AMD64/ARM64, we can add the `--platform linux/x86_64` or `--platform linux/arm64`. Check [multi-platform-docker-build](https://github.com/BretFisher/multi-platform-docker-build).

   This also applies to [docker build](#build-image-by-dockerfile).
3. By default, the root process of a container (PID 1), namely the [CMD/ENTRYPOINTWITH](#exec-and-shell) is started in the *forground* mode. The host terminal is [attached](#get-into-container) to the process's STDOUT/STDERR, but *not* STDIN. So we can see the output (error message included) of the root process as follows:

   ```bash
   ~ $ docker run -t --rm ubuntu ls /
   bin   dev  home  media  opt   root  sbin  sys  usr
   boot  etc  lib   mnt    proc  run   srv   tmp  var
   
   ~ $ docker run --rm ubuntu ls /
   bin
   boot
   dev
   etc
   home
   lib
   media
   mnt
   opt
   proc
   root
   run
   sbin
   srv
   sys
   tmp
   usr
   var
   ```

   We can use multiple `-a` options to control the attachment combinations of STDIN/STDOUT/STDERR. To following example, we can even input to the root process as long as its STDIN is open. Refer to [confused-about-docker-t-option-to-allocate-a-pseudo-tty](https://stackoverflow.com/q/30137135) and [Attach to STDIN/STDOUT/STDERR](https://docs.docker.com/engine/reference/commandline/run/#attach-to-stdinstdoutstderr--a).
   
   ```bash
   ~ $ docker run -a stdin -a stdout ...
   ```

   If we want to start the process in *background* mode, namely the *detach* mode, then add the `-d` option. Containers runs in this mode will print the container ID and release host terminal immediately. So we cannot input to the root process or see the output or error message: STDIN/STDOUT/STDERR detached. If the root process exits, then the container exits as well. So we cannot do like this:
   
   ```bash
   ~ $ docker run -d -p 80:80 my_image service nginx start
   ```
   
   As the root process *service* exits immediately after *nginx* is started. The next example will keep the container running as the *tail* command persists:
   
   ```
   ~ $ docker run -d ubuntu bash -c "tail -f /dev/null"
   ```

4. `-t` allocates a pseudo-TTY connected for the root process, especially useful when the process is an *interactive* Shell. The `-i` option forces the process's STDIN to be open and runs the container interactively, so we can *input* some data to the process directly, which works even when `-d` is present. Here is an example:

   ```bash
   echo test | docker run -i ubuntu cat -
   test
   ```

   The two options are usually used together for Shell: `-t` creates a pseudo-TTY, while `-i` makes the STDIN of the pseudo-TTY open for data input.
   
   ```bash
   # STDIN not open by default, so keeps waiting for input
   ~ $ dockder run -t --rm ubuntu bash
   
   # can input/output, but no standalone STDOUT, reuse that of host terminal
   ~ $ docker run -i --rum ubunty bash
   
   # standalone stdin/stdout/stderr, also interactive for input
   ~ $ docker run -it --rum ubunty bash
   ```

5. `--rm` automatically remove the container when it exits.
6. Check [Data Share](#data-share) for `--mount`.
7. `-w` lets the root process running inside the given directory that is created on demand.
8. `--net, --network` connects the container to a network. By default, it is *bridge*. Details are discussed in later sections.
9. `-u, --user` runs the root process as a non-root user. Attention that, the username is that within the container. So the image creator should create that name in Dockerfile.
10. *bash* overrides the CMD/ENTRYPOINT instructions of the image.

Here is a note about the different options:

![docker run](/assets/docker-run-adit.jpg)

Please also follow this post [Cannot pipe to docker run with stdin attached](https://stackoverflow.com/q/71761103/2336707).

# Data Share #

Docker containers can read from or write to pathnames, either on host or on memory filesystem - to share data. There are [three storage types](https://docs.docker.com/storage/):

1. Volume, Data Volume, or Named Volume.

   By default, running data of a container is layered on top of the image used to create it. A *volume* decouples that data from both the host or the container. Just think of a Windows partition or removable disk drive.

   Volumes are managed by Docker and persist. Data within can be shared among multiple containers, as well as between the host and a container.
   
   >Beofre you restart a docker compose project, please run "docker volume prune -a", otherwise history data might interrupt new containers.

2. Bind Mount.

   Bind-mount a file or directory in the host to a file or directory in the container. The target can be read-only or read-write. For example, bind host */etc/resolv.conf* to a container, sharing name servers.
   
   Attention please; to bind-mount a file, please provide the absolute path, otherwise the dest pathname in the container might be a directory! Check [How to mount a single file in a volume](https://stackoverflow.com/q/42248198/2336707).
3. `tmpfs` Mount.

   Needless to say, *tmpfs* is a memory filesystem that let container stores data in host memory.

We use option `--volume , -v` and `--mount` to share data between containers and hosts. `--mount` is recommended as it support all 3 kinds of data sharing and is more verbose. `--volume` will be deprecated soon.

If the file or directory on the host does not exist. `--volume` and `--mount` behaves differently. `--volume` would create the pathname as a *directory*, NOT a file, while `--mount` would report error.

On the other hand, if the target pathname already exists in the container, both options would *obsecure* contents over there. This is useful if we'd like to test a new version of code without touching the original copies.

## ENV Variables ##

To pass environment variables to containers, we can:

1. `-e, --env` applies [only](https://stackoverflow.com/q/49293967/2336707) to `docker run`. This method [reveal](https://stackoverflow.com/a/30494145/2336707) sensitive values in Shell history. We can firstly *export* the variable on CLI, then pass `--env VAR` without the value part.
2. [--env-file](https://docs.docker.com/compose/env-file/) (default `$PWD/.env`) applies both to `docker run` and [docker compose](#docker-compose). This fits when there are a lot of variables to pass in.
3. [docker compose](#docker-compose) can pick up [a few compose-specific variables](https://docs.docker.com/compose/reference/envvars/) from CLI, so just *export* it.
4. CLI variables can also be for [substitution in compose file](https://docs.docker.com/compose/compose-file/compose-file-v3/#variable-substitution).

## SELinux ##

When bind-mount a file or mount a directory of host, SELinx policy in the container [may restrict access](https://stackoverflow.com/q/24288616) to the shared pathname.

1. Temporarily turn off SELinux policy: 

   ```bash
   ~ $ su -c "setenforce 0"
   ~ $ docker restart container-ID
   ```

2. Adding a SELinux rule for the shared pathname:

   ```bash
   ~ $ chcon -Rt svirt_sandbox_file_t /path/to/
   ```

3. Pass argument `:z` or `:Z` to `--volume` option:

   ```bash
   -v /root/workspace:/root/workspace:z
   ```

   Attention please, `--mount` does not support this.
4. Pass `--privileged=true` to *docker run*.

   However, this method is discouraged as privileged containers bring in security risks. If it is the last resort, first create a privileged container and then create a non-priviledged container inside.

   Privilege permissions can have fine-grained control by `--cap-add` or `--cap-drop`, which is recommended!

# Nginx Container Sample #

```bash
~ $ docker run --name webserver \
-d \
--net host
--mount type=bind,source=/tmp/logs,target=/var/log/nginx \
-p 8080:80 nginx

~ $ docker container ls
~ $ docker container logs webserver
~ $ docker container stop/kill webserver
~ $ docker container start webserver
~ $ docker container rm webserve          # remove one or more container (even running instances)
~ $ docker container prune                # remove all stopped container
```

1. `-p` maps host port 8080 to container port 80 that is already bound to host Nginx process.
2. Check the Dockerfile, there is a line telling how Nginx should be started:

   ```
   CMD ["nginx", "-g", "daemon off;"]
   ```

3. The `--mount` type is a [Bind Mount](#data-share) directory.
4. Visit the Nginx container page at *http://host-ip:8080*.
5. *stop* attempts to trigger a [*graceful*](https://superuser.com/a/757497) shutdown by sending the standard POSIX signal SIGTERM, whereas *kill* just kills the process by sending SIGKILL signal.

## Get into Container ##

Exec a simple command in container:

```bash
~ $ docker exec webserver sh -c 'echo $PATH'
~ $ docker exec webserver ps
```

Sometimes, we want to exec the command in the background when we do not care about its output or need not any input:

```bash
~ $ docker exec -d webserver touch /tmp/test.txt
```

If we want to interactively and continuously control the container:

```bash
~ $ docker exec -it webserver bash
```

Apart from *docker exec*, [docker attach <container>](https://docs.docker.com/engine/reference/commandline/attach/) is also recommended.

This command attaches the host terminal's STDIN, STDOUT and STDERR files to a running container, allowing interactive control or inspect as if the container was running directly in the host's terminal. It will display the output of the ENTRYPOINT/CMD process.

For example, a container can be shutdown by `C-c` shortcut, sending the SIGINT signal (identical to SIGTERM) to the container by default. Rather, `C-p C-q` detaches from the container and leave it running in the background again.

If the process is running as PID 1 (like */usr/bin/init*), it ignores any signal and will not terminate on SIGINT or SIGTERM unless it is coded to do so.

# Docker Commit #

```bash
~ $ docker exec -it webserver bash
#
root@docker ~ # echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
root@docker ~ # exit
#
~ $ docker diff webserver
~ $ docker container commit -a 'jim' -m 'change front page' webserver nginx:v2
~ $ docker image ls nginx
~ $ docker history nginx:v2
```

1. We modified container layer storage. Use *diff* to check details.
2. Visit the Nginx container page again.
3. *commit* records modification as a new layer and create a new image.

   Avoid *commit* command as miscellaneous operations (garbage) are recorded either. As discussed in "Data Share" section, we can make use of Volume, Bind Mount, or tmpfs, or resort to 'Build Image by Dockerfile' section below.
4. To verify the new image:

   ```bash
   ~ $ docker run --name web2 -d -p 8081:80 --rm nginx:v2
   ```

   The `--rm` tells to remove the container upon exit.
5. We can *inspect* the target image, and will find "ContainerConfig" and "Config". They are almost identical.

   "ContainerConfig" is the config of current container from within this image is committed, while the "Config" is the exact configuration of the image. Pay attention to the [Cmd](#exec-and-shell) parts. If we [build by Dockerfile](#build-image-by-dockerfile), then they looks different. The ContainerConfig this is the temporary container spawned to create the image. Check [what-is-different-of-config-and-containerconfig-of-docker-inspect](https://stackoverflow.com/q/36216220).

# Networking Drivers #

The Docker's networking subsystem is *pluggable*, using drivers. Docker provides multiple built-in networks, based on which we can [define custom networks](#self-define-network).

Below is a simple explanation:

1. Bridge

   The *default* driver if none is given upon *run*, providing network isolation from the outside. It is to bridge traffic among multiple containers and the host. Check the image below, *docker0* is the bridge interface.

   ![docker netowrk](/assets/docker-net.png)

   If we define a [custom](#self-define-network) *bridge* network, containers within can communicate with each other by alias or name, otherwise they can [only](https://docs.docker.com/network/bridge/#differences-between-user-defined-bridges-and-the-default-bridge) communicate by IP addresses.

   Pay attention please; this is different from the *bridge* mode of VMWare or VirtualBox (real Virtual Machine). VMWare and VirtualBox's bridge is deployed directly on the host's interface and appears to be a real physical device parallel to the host and can be connected to from within LAN directly.
2. [Host](https://docs.docker.com/network/host/)

   Share the host's networking directly without isolation. However, LAN devices cannot differntiate between containers and the host as there is not individual IP addressed assigned to containers. The host mode is preferred when the service exposes a port publicly like Nginx servers.
   
   To use host network, just add `network_mode: host` to [Dockder compose file](#docker-compose), and but must remove the [ports mappings](https://docs.docker.com/compose/compose-file/05-services/#ports) as containers share the same network as the host.
   
   Unfortunately, [host mode does not work on macOS](https://github.com/docker/for-mac/issues/1031).
3. Overlay

   Overlay connects multiple Docker daemons together, creating a distributed network among multiple Docker daemon hosts. This network sits on top of (overlays) the host-specific networks, allowing containers connected to it.
4. macvlan

   As the name implied, maclan assignes a MAC address to a container, making it be a physical device on the same network as the host - counterpart of VMWare/VirtualBox's 'bridge'.
5. none

   Disable networking.

## host.docker.internal ##

Occasionally, we want to access to host services from within containers.

1. If the container is booted with [host network](#networking-drivers), then use `localhost` or `127.0.0.1`.
2. If the container is booted with [bridge network](#networking-drivers), then use `host.docker.internal`. Depending on the platform, we might [need a bit setup](https://stackoverflow.com/a/62431165/2336707).
   1. On macOS, `host.docker.internal` is intuitively supported.
   2. On Linux, we should manually add `host.docker.internal:host-gateway` to `docker run --add-host` or to [extra\_hosts](https://docs.docker.com/compose/compose-file/05-services/#extra_hosts) of [docker compose](#docker-compose). This would add an entry in "/etc/hosts".

The hostname "host.docker.internal" is used only for connection, so please set the correct "Host" header. See example below.

```bash
~$ docker exec -it alpine sh

/ # curl -v http://localhost/anything --connect-to localhost:80:host.docker.internal:80
* processing: http://localhost/anything
* Connecting to hostname: host.docker.internal
* Connecting to port: 80
*   Trying 192.168.65.254:80...
* Connected to host.docker.internal (192.168.65.254) port 80
> GET /anything HTTP/1.1
> Host: localhost
> User-Agent: curl/8.2.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: openresty/1.21.4.2rc1
< Date: Sat, 05 Aug 2023 06:45:34 GMT
< Content-Type: application/json
< Transfer-Encoding: chunked
< Connection: close
<
{"message":"hello, world"}
* Closing connection
```

## Self-define Network ##

Docker automates network configuration upon container startup. We can customize that on purpose.

```bash
~ $ docker network ls
~ $ docker network create -d bridge mynet
~ $ docker run -it --name ubt1604 -v /src/hostdir:/opt/condir:rw --network mynet ubuntu:16.04
~ $ docker network inspect mynet
```

Use the `--net` or `--network` option. To the 'host' networking driver, just pass `--net host` option to *docker run*.

## link ##

The legacy communication method is `--link`. Docker copies information (e.g. [ENV Variables](#env-variables)) from *source* container to *receipt* (*target*) container, and provides network access from receipt container to source container.

Take `docker run --name web --link postgres:alias ...` for example, the newly created *web* is receipt container and the existing *postgres* is source container.

`--link` is amost [deprecated](https://docs.docker.com/network/links/) as we can use other features to accomplish the same functionalities. For example, for info copy, use [ENV Variables](#env-variables) or [data share](#data-share). For network communication, just follow [Networking Drivers](#network-drivers) and [Self-define Network](#self-define-network).

Here is an example:

```bash
## source container

13:47:23 zachary@Zacharys-MacBook-Pro ~$ docker run --name postgres -e HELLO=world -e POSTGRES_HOST_AUTH_METHOD=trust -d postgres:14
25732d58f238b1d3e83fc52ea3c8b91f75290385066e6541d616e68eecd6cfdd

13:47:39 zachary@Zacharys-MacBook-Pro ~$ docker exec -it postgres bash
root@25732d58f238:/# echo $HELLO
world


## receipt/target container

13:48:11 zachary@Zacharys-MacBook-Pro ~$ docker run --name ubuntu -it --link postgres:db ubuntu bash

# src in hosts
root@8e95191dba0e:/# cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.3      db 25732d58f238 postgres
172.17.0.4      8e95191dba0e

# src ENVs
root@8e95191dba0e:/# env | grep DB_ | sort
DB_ENV_GOSU_VERSION=1.14
DB_ENV_HELLO=world
DB_ENV_LANG=en_US.utf8
DB_ENV_PGDATA=/var/lib/postgresql/data
DB_ENV_PG_MAJOR=14
DB_ENV_PG_VERSION=14.4-1.pgdg110+1
DB_ENV_POSTGRES_HOST_AUTH_METHOD=trust
DB_NAME=/ubuntu/db
DB_PORT=tcp://172.17.0.3:5432
DB_PORT_5432_TCP=tcp://172.17.0.3:5432
DB_PORT_5432_TCP_ADDR=172.17.0.3
DB_PORT_5432_TCP_PORT=5432
DB_PORT_5432_TCP_PROTO=tcp

# src network
root@8e95191dba0e:/# ping db
PING db (172.17.0.3) 56(84) bytes of data.
64 bytes from db (172.17.0.3): icmp_seq=1 ttl=64 time=0.210 ms
64 bytes from db (172.17.0.3): icmp_seq=2 ttl=64 time=0.452 ms
64 bytes from db (172.17.0.3): icmp_seq=3 ttl=64 time=0.136 ms
```

Attention please; `--link` is one-way link only. Info is transferred from source containers to receipt containers but source containers know nothing about receipt containers. To achieve bi-directional communication, please use [network](#networking-drivers).

# netshoot #

To debug containers' network issues, we can make use of [netshoot](https://github.com/outsinre/netshoot).

The netshoot container has a world of built-in network troubleshooting tools like *nmap*, [tcpdump](http://www.tcpdump.org/tcpdump_man.html), etc. We just need to attach the netshoot container to the target container's network namespace.

```bash
~ $ docker run --name netshoot \
--rm \
-it \
--network container:<target-name|target-ID> \
--mount type=bind,src=./data/,dst=/data \
-d nicolaka/netshoot

~ $ docker exec -it netshoot zsh

# capture packets of target container
~ # tcpdump -i eth0 port 6379 -w /data/redis.pcap
```

# exec and shell #

Usually, in the end of image, we have three kinds of *instruction*:

1. RUN. Creates a new layer of image. Usually used to install package. It's recommended to install multiple packages in a single RUN instruction so that we have less image layers.
2. CMD. Set the default command to run when *run* a container.
3. ENTRYPOINT. *run* a container as a command, so that we can provide extra arguments when *run*.

Refer to [RUN, ENTRYPOINT and CMD](http://goinbigdata.com/docker-run-vs-cmd-vs-entrypoint/) for more details.

Each instruction has two kinds of running forms:

1. shell form.

   ```
   <instruction> cmd para1 para2 ...
   ```

   By default, */bin/sh* will be used to run the *cmd* as:

   ```
   /bin/sh -c 'cmd para1 para2 ...'
   ```

   It is the preferred form of the RUN instruction to install package in the image and exit.
2. exec form.

   The *cmd* is ran directly without any Shell involvement, which is the preferred form of CMD and ENTRYPOINT as we usually launch a daemon background within the container. No need to maintain a Shell process.

   ```
   <instruction> ["cmd", "para1", "para2", ...]
   ```

   We can hack by changing the *cmd* to */bin/bash*:

   ```
   <instruction> ["/bin/bash", "-c", "cmd", "para1", "para2", ...]
   ```

Refer to [exec form or sh form](https://www.cnblogs.com/sparkdev/p/8461576.html) for more details.

When there are multiple CMD or ENTRYPOINT instructions inherited from different image layers, only that of the topmost layer is respcted! We can use *docker container inspect* to show the instructions and their forms. For example, *nginx* image has `CMD ["nginx", "-g", "daemon off;"]`.

We can pass custom commands and arguments when invoking *docker run*, which will override the CMD instruction and arguments thereof. If there exists the ENTRYPOINT instruction in exec form, then custom arguments would be appended to the ENTRYPOINT *cmd*. By default, ENTRYPOINT exec form will take extra arguments from CMD instruction in shell form. Custom arguments when *docker run* will override those in the CMD instruction. ENTRYPOINT in shell form would ignore custom arguments from CMD or *docker run*.

Here is an illustration between CMD and ENTRYPOINT:

![cmd-entrypoint](/assets/cmd-entrypoint.png)

Refer to [Dockerfile reference](https://docs.docker.com/engine/reference/builder/) for more details.

# Docker Build Dockerfile #

From [commit](#docker-commit) example above, we can create new image layer but many negligible commands like *ls*, *pwd*, etc. are recourded as well.

Similar to Makefile, Docker uses Dockerfile to define image with specified *instruction*s like FROM, COPY, RUN etc. Each instruction creates a new intermediate layer and a new intermediate image. In order to minimize number of layers and images, we'd better merge instructions as much as possible.

In this section, we use Dockerfile to create image *nginx:v2*.

```bash
~ $ mkdir mynginx
~ $ cd mynginx
~ $ vim Dockerfile
#
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

Just two lines! FROM imports the base image on which we will create the new layer. If we do not want any base image, use the special null image *scratch*.

Now we build the image:

```bash
~ $ docker build -t nginx:v3 -t nginx .
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

1. The `-t` option of *docker build* actually refers to name of the target image. We can leave the tag part to use the default *latest*. We can also [supply multiple tags](https://stackoverflow.com/a/36398336). From example above, the second tag is the default *latest*.

   If we specify a name that exists already, we detach that name from an existing image and associate it with the new image. The old image still exist and can be checked the "RepoTags" field of `docker inspect <image-id>`.
2. During the building process, both an intermediate container (d5baea5c6341) and image (18cc3a3480f0) is created for 'RUN' instruction.

   The intermediate container defines a new layer which is then committed to create a new image. Afterwards, the intermediate container is removed, but the intermediate image is kept.
3. The trailing dot means the current directory is the building *context* directory. It is also the default location of the Dockerfile. Sometimes, we exclude some context files from the durectory by the *.dockerignore* file, as below:

   ```bash
   ~ $ echo ".git" >> .dockerignore
   ```

   Docker sends all files within context directory to remote Docker engine (daemon). Image can be built without a context directory if we don't have any supplementary files.

   ```bash
   ~ $ docker build -t nginx:v3 - < /path/to/Dockerfile
   ```

   The hypen character cannot be omitted!
4. If there exist multiple CMD/ENTRYPOINT instructions from different layers, only that of the topmost layer will be executed upon container start. All the rest CMD/ENTRYPOINT are overriden.
5. After the building, we can run *nginx:v3* image:

   ```bash
   ~ $ docker run --name web3 -d -p 8081:80 --rm nginx:v3
   ```

6. Apart from builing a new docker image for the web server, we can utilize 'Data Share' to attach a Volume or Bind Mount to the base docker image. Build the web server within the attached storage instead.
7. Sometimes, we may want to remove the cache of *build*, which can be accomplished by *docker builder prune -a*.
8. Here is another Dockerfile instance:

   ```
   FROM centos:latest
   RUN [/bin/bash, -c, 'groupadd -g 1000 username ; useradd -ms /bin/bash -u 1000 -g 1000 username; echo "username:1C2B3A" | chpasswd']
   CMD ["/bin/bash"]
   ```

   1. Recall that in section 'Run an Image', `-u username:groupname` requires that the username and groupname exist when creating the image. Change the account password immedately after the container is created as the initial password is explicitly written in the Dockerfile.
   2. By default, the 'RUN' instruction uses */bin/sh* form. It is replaced by */bin/bash* in this example.

      Also, multiple relevant shell commands are merged into one single 'RUN' instruction. We can also the split the command by line continuation like:

      ```
      RUN /bin/bash -c 'useradd -ms /bin/bash -u 1000 -g 1000 username ; \
      echo "username:1C2B3A" | chpasswd'
      ```

9. Check [multi-platform-docker-build](https://github.com/BretFisher/multi-platform-docker-build) for AMD64/ARM64 arch issue. This also applies to [Create a Container](#launch-containers).

Refer to [Best practice for writing Dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/).

# Share Images #

We can [share](https://docs.docker.com/get-started/04_sharing_app/) our own docker image to a registry (e.g. docker.io) by *docker push*.

Suppose we have a nginx image got from Docker hub, and want to re-push it to Docker Hub under our own account.

Get the official image. If we do not specify a tag, the *latest* image is pulled.

```bash
~ $ docker pull nginx
Using default tag: latest
...
```

Push to our own account. If we do not specify a tag, the image is tagged to *latest*.

```bash
~ $ docker login -u myaccount

~ $ docker tag <nginx|sha256> myaccount/nginx

~ $ docker push myaccount/nginx
Using default tag: latest
...
```

If we want to use a different registry rather than the default *docker.io*, then add the registry to the new tag as well as follows.

```bash
~ $ docker tag <nginx|sha256> myregistry:5000/myaccount/nginx

~ $ docker push myregistry:5000/myaccount/nginx
```

We can assign mutiple tages to the image.

```bash
~ $ docker tag <nginx|sha256> myaccount/nginx:2.0.0
~ $ docker push myaccount/nginx:2.0.0

~ $ docker tag <nginx|sha256> myaccount/nginx:2.0
~ $ docker push myaccount/nginx:2.0

~ $ docker tag <nginx|sha256> myaccount/nginx:2
~ $ docker push myaccount/nginx:2
```

We can re-assign the *latest* tag to a new version.

```bash
~ $ docker tag myaccount/nginx:2.1.0 myaccount/nginx:latest
~ $ docker push myaccount/nginx:latest
```

A more advanced tool than *docker tag* is [regctl](https://github.com/regclient/regclient/blob/main/docs/regctl.md) as follows.

```bash
~ $ regctl registry login
~ $ regctl registry config

~ $ regctl image manifest kong/kong-gateway:latest
~ $ regctl image inspect kong/kong-gateway:latest

~ $ regctl image manifest kong/kong-gateway:3.4.1.0

~ $ regctl image copy kong/kong-gateway:3.4.1.0 kong/kong-gateway:latest
~ $ regctl image copy kong/kong-gateway:3.4.1.0-ubuntu kong/kong-gateway:latest-ubuntu
```

*regctl image copy* pulls the containers (all architectures), retags them, and pushed them again (all architectures). Please read <https://stackoverflow.com/a/68576882/2336707>, <https://stackoverflow.com/a/68317548/2336707> and <https://stackoverflow.com/q/71470604/2336707>.

# Docker Compose #

the compose project name by default is named after `PWD`. The name of containers share the same prefix (i.e. name of the project).

we can [share compose configurations](https://docs.docker.com/compose/extends) between files and/or projects by:

1. multiple compose files
2. extending services from another compose file.

when bind-mount a file, pay attention to provide the absolute path. check [data share](#data-share).

In Docker Compose file, we can use *build* to build an image from Docker file (https://stackoverflow.com/q/57840820/2336707). Alternatively, we can also provide multiple commands to *command* (https://stackoverflow.com/q/30063907/2336707).
