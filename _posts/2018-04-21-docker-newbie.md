---
layout: post
title: Docker Newbie
---

1. toc
{:toc}

# Architecture #

In general, [Docker works in client-server mode](https://docs.docker.com/get-started/docker-overview/#docker-architecture) illustrated in the figure below.

![docker-architecture.png](/assets/docker-architecture.png)

1. Server-side [Docker Engine](#docker-engine), also called Docker Daemon, is the *dockerd* that manages the containers, images, volumes, networks, etc.

   The daemon serves requests by RESTful API [over local Unix socket or over remote network interface](https://docs.docker.com/engine/daemon/remote-access/).
2. Client-sdie Docker CLI are *docker*, *docker-compose*, etc.

   CLI options and arguments are consolidated and transformed to REST API. If we know the API spec, we can even [use curl to manage Docker objects](#dockersock).

The term "Docker" most of the time means the overall architecture.

## Docker Engine ##

Most of the time, Docker Daemon is merely the daemon part. But occasionally, it may refer to the collection of Docker daemon, the Docker REST API and the Docker client. In this section, we focus on the daemon.

Docker Engine relies on Linux kernel under the hood, like the [namespace](https://man7.org/linux/man-pages/man7/namespaces.7.html) (containers isolation), [cgroup](https://man7.org/linux/man-pages/man7/cgroups.7.html) (resources management), libraries, etc. So, by "container", we actually mean "Linux container". But for convenience, we just call it "container".

At the very beginning, Docker Engine is an integrated piece of daemon including all capabilities to manage containers, images, volumes, networks, etc.

```
+--------------+            +-----------------+        +----------+   
|              | REST API   |  Docker Engine  |        |+----------+  
|  docker CLI  +----------->|                 +------->+|+----------+ 
|              |            |     dockerd     |         +|+----------+
+--------------+            +-----------------+          +|container+|
                                                          +----------+
```

Later on (2017), in order to expand its adoption and add neutrality and modularity, the core capabilities are donated to CNCF as a seperate daemon [containerd](https://www.docker.com/blog/containerd-vs-docker/) (i.e. *docker-containerd*). The Docker Engine only focuses on developer experience like serving login, build, inspect, log, etc. 

```
                                                       +--------------+
+--------------+            +-----------------+        |              |       +----------+
|              | REST API   |  Docker Engine  |  gRPC  | containerd   |       |+----------+
|  docker CLI  +----------->|                 +------->|     +------+ +------>+|+----------+ 
|              |            |     dockerd     |        |     | runc | |        +|+----------+
+--------------+            +-----------------+        |     +------+ |         +|container+|
                                                       +--------------+          +----------+
```

In the meantime, the [Open Container Initiative (OCI)](https://opencontainers.org/) standardizes the *containerd*. According to the standard, the responsibility of creating containers was removed from *containerd* in faviour of *runtime* (e.g. *runc*). The interaction with Linux namespace and Linux cgroup has shifted from *containerd* to *runtime*. We call *containerd* the high-level runtime, managing container lifecycle, images, volumes, network, etc.

```
                                                       +--------------+
+--------------+            +-----------------+        |              |       +----------+
|              | REST API   |  Docker Engine  |  gRPC  | containerd   |       |+----------+
|  docker CLI  +----------->|                 +------->|     +------+ +------>+|+----------+ 
|              |            |     dockerd     |        |     | runc | |        +|+----------+
+--------------+            +-----------------+        |     +------+ |         +|container+|
                                                       +--------------+          +----------+
```

With OCI, everyone can build his own containerization system. Nowadays, there exist [multiple OCI runtimes](https://docs.docker.com/engine/daemon/alternative-runtimes/). In order to support those runtimes, Docker inserts a new component between *containerd* and *runtime*, namely the [containerd-shim](https://docs.docker.com/engine/daemon/alternative-runtimes/) (e.g. *docker-containerd-shim*). The *containerd-shim* invokes the *runtime* (e.g. *runc*) to create the container. Once the container is created, the *runtime* exits and the lifecycle management is handed over to *containerd* and *container-shim*.

Some of the runtimes are fully compatibile with *docker-containerd-shim* like the [youki](https://github.com/youki-dev/youki), while others are not like the [Wasmtime](https://wasmtime.dev/). Runtimes compatibile with *docker-containerd-shim* can be a drop-in replacement for *runc*. Runtimes incompatibile with *docker-containerd-shim* must implements their own *shim* according to [shim API](https://github.com/containerd/containerd/blob/main/core/runtime/v2/README.md).

```
                                                                                                   container lifecycle
                                                                                   +--------------------------------------------+
                                                                                   |                                            v
                                                                                   |                   +---------------+   +---------+
                                                                                   |                   | runtime youki +-->|container|
                                                                                   |                   |    (exit)     |   +---------+
                                                                                   |                   +---------------+
                                                                                   v                      ^
+--------------+            +-----------------+        +--------------+    +-----------------+            |
|              |  REST API  |  Docker Engine  |  gRPC  |  High-level  |    |                 |   image    |                     .
|  docker CLI  +----------->|                 +------->|   runtime    +--->| containerd-shim +------------+                     .
|              |            |     dockerd     |        |  containerd  |    |                 |   bundle   |                     .
+--------------+            +-----------------+        +--------------+    +-----------------+            |
                                                                                   ^                      v
                                                                                   |                   +--------------+
                                                                                   |                   | runtime runc |    +---------+
                                                                                   |                   |    (exit)    +--->|container|
                                                                                   |                   +--------------+    +---------+
                                                                                   |                                            ^
                                                                                   +--------------------------------------------+
                                                                                                   container lifecycle
```



More and more middle layers are added to the architecture, making it too complicated. [podman](https://podman.io/), on the other hand, is much more simpler as follows. *podman* talks directly to the runtime, without *dockerd*, *conainerd* or *containerd-shim*.

```
podman CLI -> runtime runc -> containers
```

You are strongly recommended to read <https://stackoverflow.com/q/46649592/2336707>. The following output is a demonstration in my dev environment. Two containers were created and they are the child processes of the `containerd-shim-runc-v2`.

```bash
ubuntu@ip-172-31-9-194:~/misc$ docker run --name httpbin -P -d kennethreitz/httpbin
4bd1077052750a2a7552e4347bcbba483d47f2555b89874606c0b04b93f7c2dc

ubuntu@ip-172-31-9-194:~/misc$ docker run --rm -itd ubuntu
18be920d5a998ceee5f438ae43c7d1171fec6d34d4742c66611ff7a63f9d6a68

ubuntu@ip-172-31-9-194:~/misc$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                                       NAMES
a598c760b207   ubuntu                 "/bin/bash"              29 minutes ago   Up 29 minutes                                               ubuntu
4bd107705275   kennethreitz/httpbin   "gunicorn -b 0.0.0.0…"   48 minutes ago   Up 48 minutes   0.0.0.0:32768->80/tcp, [::]:32768->80/tcp   httpbin

ubuntu@ip-172-31-9-194:~/misc$ ps  -eF --forest
# dockerd
root         530       1  0 570701 82248  3 Dec23 ?        00:00:29 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
root       15033     530  0 436282 3968   0 Dec27 ?        00:00:00  \_ /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 32768 -container-ip 172.17.0.2 -container-port 80
root       15040     530  0 399416 3840   0 Dec27 ?        00:00:00  \_ /usr/bin/docker-proxy -proto tcp -host-ip :: -host-port 32768 -container-ip 172.17.0.2 -container-port 80
# containerd
root         357       1  0 468968 48524  3 Dec23 ?        00:02:22 /usr/bin/containerd
# container httpbin
root       15063       1  0 309542 13748  0 Dec27 ?        00:00:04 /usr/bin/containerd-shim-runc-v2 -namespace moby -id 4bd1077052750a2a7552e4347bcbba483d47f2555b89874606c0b04b93f7c2dc -address /run/containerd/containerd.sock
root       15083   15063  0 21495 24480   0 Dec27 ?        00:00:05  \_ /usr/bin/python3 /usr/local/bin/gunicorn -b 0.0.0.0:80 httpbin:app -k gevent
root       15109   15083  0 32493 33912   3 Dec27 ?        00:00:08      \_ /usr/bin/python3 /usr/local/bin/gunicorn -b 0.0.0.0:80 httpbin:app -k gevent
# container ubuntu
root       15363       1  0 309542 13780  3 Dec27 ?        00:00:04 /usr/bin/containerd-shim-runc-v2 -namespace moby -id a598c760b2071f4951d35d255d3669ce3e9877534b36efcd21e2dfcb8727b136 -address /run/containerd/containerd.sock
root       15383   15363  0  1147  3840   2 Dec27 pts/0    00:00:00  \_ /bin/bash
```

### Docker Desktop ###

In order to run containers on Window and macOS, a Linux virtual machine is required to host *dockerd*, *containerd*, *containerd-shim*, *runtime*, etc. The Docker CLI remains on the host.

As such, Docker provides the Docker Desktop. Docker Desktop is an [all-in-one](https://docs.docker.com/desktop/) (including GUI) software and gives us a uniform expiernce accross Linux, Window and macOS.

![docker-desktop.png](/assets/docker-desktop.png)

Especially, Docker Desktop creates the Linux virtual machine for us, so that we are not bothered on this. On Window and macOS, Docker Desktop utilizes native virtualization framework (Hyper-V and WSL 2 of Windows; Hypervisor of macOS) to boost performance. On Linux system, Docker Desktop is not a must. However, if we choose it, the Virtual Machine is still created.

# Image Container and Registry #

Dockers comprises *image*, *container* and *registry*.

1. Image is *static*, *readonly* and a *minimal* root filesystem [bundle](#share-image). There are many highly qualified base iamge from official registry like *nginx*, *redis*, *php*, *python*, *ruby* etc. Especially, we have *ubuntu*, *centos*, etc. that are just OS minimal bare bones (like Gentoo stage tarball).

   An image consists of multiple incremental layers that are defined by a [Dockerfile](#docker-build-dockerfile). Correspondingly, we call the image storage [layer storage](https://docs.docker.com/storage/storagedriver/) based on Union Filesytem (FS). Recall that booting USB stick also uses Union FS. The most adopted Union FS are *overlay2*.
2. Container is a set of processes with added isolation (Linux namespace) and resource management (Linux cgroup).

   It is created on top of a base image with an additional layer storing *running* but volatile data. We can think of image and container as class and object in Object-oriented programming. 
3. Registry is *store* where users publicize, share and download *repostitory*. The default registry is *docker.io* or *registry-1.docker.io* with a frontend website [Docker Hub](https://hub.docker.com).

   Repository, on the other hand, actually refers to *name* of an image (e.g. *ubuntu*). We can specify *version* of a repository by a *tag* (label) like *ubuntu:16.04* (colon separator). The default tag is *latest*.

The [naming of an image](https://docs.docker.com/reference/cli/docker/image/tag/#description) follows the format as follows.

```
registry.fqdn[:port]/[namespace/]repository[:tag | @<image-ID>]
```

1. Default registry can be ommitted. Others like `quay.io` must be provided.
2. The *namespace* part means a registered account in the registry. It can be an individual user name or an organization name like "kong".
3. *repository* is the default *name* of an image like "ubuntu" and "kong-gateway".
4. *tag* is a string. Default to *latest*.
5. *image id* comprises a SHA256 *digest* like *@sha256:abea36737d98dea6109e1e292e0b9e443f59864b*.

Specifying an image by tag can always gets the latest updates. For example, every time a patch release is released (e.g. *kong/kong:3.4.5*), *kong/kong:3.4* refers to that patch version `3.4.5` with a different digest. On the other hand, an image specified by digest always is pinned to that specific image. See [Pin actions to a full length commit SHA for security concerns](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions#using-third-party-actions).

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

# Start Daemon #

Start Docker.

```bash
~ $ sudo systemctl enable docker
~ $ systemctl status docker
~ $ sudo systemctl start docker

~ $ docker info
~ $ docker ps
~ $ docker compose ls
```

All data is located in the `/var/lib/docker` directory.

```bash
ubuntu@ip-172-31-9-194:~/misc$ sudo -E PATH=$PATH ls /var/lib/docker/
buildkit  containers  engine-id  image  network  overlay2  plugins  runtimes  swarm  tmp  volumes
```

## docker.sock ##

Docker daemon creates a Unix socket file at [/var/run/docker.sock](https://stackoverflow.com/q/35110146/2336707) by which we can communicate with the daemon via RESTful API.

```bash
~ $ curl --unix-socket /var/run/docker.sock --no-buffer http://localhost/events
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

However, mounting *docker.sock* would make your host [vulnerable to attack](https://dev.to/pbnj/docker-security-best-practices-45ih#docker-engine) as Docker daemon within the container is ran as *root*.

# Docker Context #

Recall that Docker works [in client-server mode](#architecture), where the client connects to the server via RESTful API. As a developer, it is not unusual that we have different environment of different purposes such as development environment, staging environment, buildx environment, production environment, etc.

For a single *docker* CLI to communicate with different Docker Engines, we have [Docker Context](https://docs.docker.com/engine/manage-resources/contexts/). A context is a profile recording the information of a Docker Engine like the IP address. To swtich between Docker Engines, just use *docker context* CLI.

By default, only the local Unix socket is enabled. The example below enables IP socket listening and only binds to localhost for [security concern](https://docs.docker.com/engine/security/protect-access/).

```bash
~ $ sudo systemctl editedit docker.service
# /etc/systemd/system/docker.service.d/override.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375 --containerd=/run/containerd/containerd.sock

~ $ sudo systemctl daemon-reload

~ $ sudo systemctl restart docker

~ $ ps -eFH | grep [d]ockerd
root       24927       1  0 552268 77412  3 07:29 ?        00:00:00   /usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375 --containerd=/run/containerd/containerd.sock

~ $ curl -sS http://localhost:2375/containers/json | jq -r '.[].Names'
[
  "/httpbin"
]
```

# CLI Sample #

```bash
~ $ fgrep -qa docker /proc/1/cgroup; echo $?                    # check if it is within a docker or on the host

~ $ docker info                                                 # display the outline of docker environment
~ $ docker image/container ls [-a]                              # list images/containers
~ $ docker [image] history                                      # show layers of an image
~ $ docker inspect [ name | ID ]                                # display low-level details on any docker objects
~ $ docker logs <container>
```

Here is the full list of docker CLI: [Docker CLI](https://docs.docker.com/engine/reference/commandline/docker/).

# Pull Image #

It is highly recommended to *pull* the [docker/getting-started](https://hub.docker.com/r/docker/getting-started) image, *run* and visit `http://localhost`.

```bash
~ $ docker search -f is-official=true ubuntu                    # search only official image

~ $ docker pull ubuntu:16.04                                    # specify a tag
~ $ docker pull ubuntu@sha256:<hash>                            # specify an image ID

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

   This also applies to [docker build](#docker-build-dockerfile).
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

# Runtime metrics #

Please see <https://docs.docker.com/engine/containers/runmetrics/>.

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

Apart from *docker exec*, [docker attach \<container\>](https://docs.docker.com/engine/reference/commandline/attach/) is also recommended.

This command attaches the host terminal's STDIN, STDOUT and STDERR files to a running container, allowing interactive control or inspect as if the container was running directly in the host's terminal. It will display the output of the ENTRYPOINT/CMD process.

For example, a container can be shutdown by `C-c` shortcut, sending the SIGINT signal (identical to SIGTERM) to the container by default. Rather, `C-p C-q` detaches from the container and leave it running in the background again.

If the process is running as PID 1 (like */usr/bin/init*), it ignores any signal and will not terminate on SIGINT or SIGTERM unless it is coded to do so.

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
   
   To use host network, just add `network_mode: host` to [Dockder compose file](#docker-compose), and but must remove the [ports mappings](https://docs.docker.com/compose/compose-file/05-services/#ports) as containers share the same network as the host. Alternatively, add `--network=host` option to `docker run`.
   
   Unfortunately, [host mode does not work on macOS](https://github.com/docker/for-mac/issues/1031).
3. Overlay

   Overlay connects multiple Docker daemons together, creating a distributed network among multiple Docker daemon hosts. This network sits on top of (overlays) the host-specific networks, allowing containers connected to it.
4. macvlan

   As the name implied, maclan assignes a MAC address to a container, making it be a physical device on the same network as the host - counterpart of VMWare/VirtualBox's 'bridge'.
5. none

   Disable networking.

## DNS ##

Docker Destkop has multiple built-in DNS servers as follows.

![docker-desktop-dns.png](/assets/docker-desktop-dns.png)

When resolving containers within the the same Docker network, the internal DNS within *dockerd* is utilized. However, resolving hostnames outside of the Docker network, would be forwarded to host via CoreDNS.

Please read [How Docker Desktop Networking Works Under the Hood](https://www.docker.com/blog/how-docker-desktop-networking-works-under-the-hood/) regarding how Docker Destop achieves DNS, HTTP Proxy, TCP/IP stack, Port Forwarding, etc. network features via *vpnkit*.

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

## SSH Agent Forwarding ##

In order not to set up a new SSH environment within containers, we can [forward SSH agent on host to container](https://docs.docker.com/desktop/networking/#ssh-agent-forwarding).

For [Docker Desktop](https://docs.docker.com/desktop/) on macOS and Linux. If this does not work on macOS, see the workaround at [macOS SSH agent forwarding not working any longer](https://github.com/docker/for-mac/issues/7204).

```bash
~ $ docker run --rm -it -u root \
--mount "type=bind,src=/run/host-services/ssh-auth.sock,target=/run/host-services/ssh-auth.sock,ro" \
-e SSH_AUTH_SOCK="/run/host-services/ssh-auth.sock" \
--entrypoint /bin/bash kong/kong-gateway:latest

root@3442a4bc63cd:/# ssh-add -l
```

For [Docker engine](https://docs.docker.com/engine/):

```bash
~ $ docker run --rm -it -u root \
--mount "type=bind,src=$SSH_AUTH_SOCK,target=/run/host-services/ssh-auth.sock,ro" \
-e SSH_AUTH_SOCK="/run/host-services/ssh-auth.sock" \
--entrypoint /bin/bash kong/kong-gateway:latest

root@3442a4bc63cd:/# ssh-add -l
```

Attention that, if you are SSH into Linux VPS from macOS, the SSH agent might be forwarded to the Linux VPS, depending on the SSH config on macOS. This is totally a different topic. Containers in the Linux VPS has no access to the forwarded macOS SSH agent, and we should launch a new one in the Linux VPS.

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
   <instruction> cmd arg1 arg2 ...
   ```

   By default, */bin/sh* will be used to run the *cmd* as:

   ```
   /bin/sh -c 'cmd arg1 arg2 ...'
   ```

   It is the preferred form of the RUN instruction to install package in the image and exit.
2. exec form.

   The *cmd* is ran directly without any Shell involvement, which is the preferred form of CMD and ENTRYPOINT as we usually launch a daemon background within the container. No need to maintain a Shell process.

   ```
   <instruction> ["cmd", "arg1", "arg2", ...]
   ```

   We can hack by changing the *cmd* to */bin/bash*:

   ```
   <instruction> ["/bin/bash", "-c", "cmd", "arg1", "arg2", ...]
   ```

Refer to [exec form or sh form](https://www.cnblogs.com/sparkdev/p/8461576.html) for more details.

When there are multiple CMD or ENTRYPOINT instructions inherited from different image layers, only that of the topmost layer is respected! We can use *docker container inspect* to show the instructions and their forms. For example, *nginx* image has `CMD ["nginx", "-g", "daemon off;"]`.

We can pass custom commands and arguments when invoking *docker run*, which will override the CMD instruction and arguments thereof. If there exists the ENTRYPOINT instruction in exec form, then custom arguments would be appended to the ENTRYPOINT *cmd*. By default, ENTRYPOINT exec form will take extra arguments from CMD instruction in shell form. Custom arguments when *docker run* will override those in the CMD instruction. ENTRYPOINT in shell form would ignore custom arguments from CMD or *docker run*.

We can override ENTRYPOINT and/or CMD as follows.

```bash
# docker run --entrypoint /path/to/cmd <image> -a arg1 -b arg2 arg3

~ $ docker run --entrypoint /bin/bash -it nginx

~ $ docker run --rm --entrypoint /bin/bash kong/kong-gateway:latest -c "kong version"
Kong Enterprise 3.6.1.0

# docker run --entrypoint '' <image> /path/to/cmd -a arg1 -b arg2 arg3

~ $ docker run --rm --entrypoint '' -it kong/kong-gateway:latest /bin/bash -c "kong version"
Kong Enterprise 3.6.1.0
```

Here is an illustration between CMD and ENTRYPOINT:

![cmd-entrypoint](/assets/cmd-entrypoint.png)

Refer to [Dockerfile reference](https://docs.docker.com/engine/reference/builder/) for more details.

# Custom Image #

## Docker Commit ##

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

   "ContainerConfig" is the config of current container from within this image is committed, while the "Config" is the exact configuration of the image. Pay attention to the [Cmd](#exec-and-shell) parts. If we [build by Dockerfile](#docker-build-dockerfile), then they looks different. The ContainerConfig this is the temporary container spawned to create the image. Check [what-is-different-of-config-and-containerconfig-of-docker-inspect](https://stackoverflow.com/q/36216220).

## Docker Build Dockerfile ##

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

Just two lines! FROM imports the [base image](https://docs.docker.com/build/building/base-images/) on which we will create the new layer. If we do not want any base image, use the [special null image scratch](https://docs.docker.com/build/building/base-images/#create-a-minimal-base-image-using-scratch).

Now we build the image. Please refer to [Docker Build Overview](https://docs.docker.com/build/concepts/overview/) to check the difference between *docker buildx* and *docker build*. To put it simple, *docker build* is wrapper of *docker buildx* with default arguments.

```bash
~ $ BUILDKIT_PROGRESS=plain docker buildx build --no-cache --load -t nginx:v3 -t nginx -f Dockerfile .
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
10. Some commands may have ask interactive questions. Please check [DEBIAN_FRONTEND=noninteractive](https://serverfault.com/q/949991).

Refer to [Best practice for writing Dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/).

# Share Images #

We can [share](https://docs.docker.com/get-started/04_sharing_app/) our own docker image to a registry (e.g. `docker.io`) by *docker push*.

Suppose we have a nginx image got from Docker hub, and want to re-push it to Docker Hub under a personal account.

Pull official image. If we do not specify a tag, the *latest* image is pulled.

```bash
~ $ docker pull nginx
Using default tag: latest
...
```

Login to the personal account.

```bash
~ $ docker login -u myaccount
```

Push to my personal account. If we do not specify a tag, the image is tagged to *latest*.

```bash
~ $ docker tag <nginx|sha256> myaccount/nginx

~ $ docker push myaccount/nginx
Using default tag: latest
...
```

If we want to use a different registry rather than the default `docker.io`, then add the registry to the new tag as well as follows.

```bash
~ $ docker tag <nginx|sha256> myregistry.com:5000/myaccount/nginx

~ $ docker push myregistry.com:5000/myaccount/nginx
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

If the imange is multi-platform (e.g. AMD64 and ARM64) capable, we have to repeat the pull, tag and push for each platform via option `--platform`. The more advanced tool [regctl](https://github.com/regclient/regclient/blob/main/docs/regctl.md) takes care of all platforms with just one command. Please read [1](https://stackoverflow.com/a/68576882/2336707), [2](https://stackoverflow.com/a/68317548/2336707) and [3](https://stackoverflow.com/q/71470604/2336707).

```bash
~ $ regctl registry login
~ $ regctl registry config

~ $ regctl image manifest kong/kong-gateway:latest
~ $ regctl image inspect kong/kong-gateway:latest

~ $ regctl image manifest kong/kong-gateway:3.4.1.0

# pull, tag and push
~ $ regctl image copy kong/kong-gateway:3.4.1.0 kong/kong-gateway:latest
~ $ regctl image copy kong/kong-gateway:3.4.1.0-ubuntu kong/kong-gateway:latest-ubuntu
```

Apart from pushing a personal account, we can [share Docker images via a tar file](https://gist.github.com/outsinre/d2b58b289425fbdd2d0f0294f3fdf0c9).

```bash
~ $ docker images 'kong-wp'

~ $ docker image save -o kong-wp-3501.tar kong-wp:3.5.0.1

# for file transmission
~ $ tar -cJvf kong-wp-3501.tar.xz kong-wp-3501.tar
~ $ tar -xJvf kong-wp-3501.tar.xz

~ $ docker image load -i kong-wp-3501.tar
```

# Docker Compose #

The compose project name by default is named after `PWD`. The name of containers share the same prefix (i.e. name of the project).

We can [share compose configurations](https://docs.docker.com/compose/extends) between files and/or projects by including other compose files or extending a service from from another service of another compose file. Check "biji/archive/kong-dev-compose.yaml".

When bind-mount a file, pay attention to provide the absolute path. Check [data share](#data-share).

In Docker Compose file, we can also use [buildx](#docker-build-dockerfile) to [build an image from Dockerfile](https://stackoverflow.com/q/57840820/2336707). Alternatively, we can also [run multiple commands](https://stackoverflow.com/q/30063907/2336707).
