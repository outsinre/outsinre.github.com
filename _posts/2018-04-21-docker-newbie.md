---
layout: post
title: Docker Newbie
---

1. toc
{:toc}

# [ABCs](https://yeasy.gitbooks.io/docker_practice/content/introduction/what.html)

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
   2. The *user* part means a registered user account in the regirstry.
   3. *repository* is the default *name* of an image. Occasionally, we may see a repository containing a slash like "docker/getting-started". This is not unusual.
   4. *image id* comprises a SHA256 *digest* like *ubuntu@abea36737d98dea6109e1e292e0b9e443f59864b* (at sign separator).
4. C/S mode.
   1. Client: user command line (i.e. *docker image ls*)
   2. Server: local/remote *docker-engine* (i.e. *systemctl start docker*).
5. [Layer storage](https://docs.docker.com/storage/storagedriver/) uses Union FS (recall that Live CD on USB stick requires Union FS). Only the topmost (container storage layer) is writable and volatile.

   Of the Union FS, *overlay2* is recommended over *aufs*. Either enable *overlay2* in kernel or build external module. *devicemapper* is also used in CentOS/RHEL. Pay attention to [CentOS/RHEL 的用户需要注意的事项](https://yeasy.gitbooks.io/docker_practice/content/image/rm.html#centosrhel-%E7%9A%84%E7%94%A8%E6%88%B7%E9%9C%80%E8%A6%81%E6%B3%A8%E6%84%8F%E7%9A%84%E4%BA%8B%E9%A1%B9) if *devicemapper* driver *loop-lvm* mode is used.

# Daemon

Install [docker-ce instead of docker-ee](https://docs.docker.com/install/linux/docker-ce/centos/)

```bash
~ # systemctl enable docker
~ # systemctl status docker
~ # systemctl start docker
```

The daemon manages everything!

# Command Sample #

```bash
root@tux ~ # fgrep -qa docker /proc/1/cgroup; echo $?                    # check if it is a docker
root@tux ~ # docker info                                                 # display the outline of docker environment
root@tux ~ # docker image/container ls [-a]                              # list images/containers
root@tux ~ # docker [image] history                                      # show layers of an image
root@tux ~ # docker inspect [ name | ID ]                                # display low-level details on any docker objects
root@tux ~ # docker logs <container>
```

Here is the full list of docker CLI: [Docker CLI](https://docs.docker.com/engine/reference/commandline/docker/).

# Pull

It is highly recommended to *pull* the [docker/getting-started](https://hub.docker.com/r/docker/getting-started) image, *run* and visit `http://localhost`.

```bash
root@tux ~ # docker search -f is-official=true ubuntu                    # search only official image

root@tux ~ # docker pull ubuntu:16.04                                    # specify a tag
root@tux ~ # docker pull ubuntu@<sha256>                                 # specify an image ID

root@tux ~ # docker pull amd64/amazonlinux                               # AMD64
root@tux ~ # docker pull arm64v8/amazonlinux                             # Apple M1 ARM64

root@tux ~ # docker images
```

1. *docker search* search docker images from registries defined in */etc/container/registries.conf*.

   Unfortunately, it does *not* suport tags or IDs. Instead, go to the registry website or check third party tool [DevOps-Python-tools](https://github.com/HariSekhon/DevOps-Python-tools).
2. We can offer *docker pull* a tag (i.e. 'ubuntu:latest') or an *image ID*.
3. When pulling an image without its digest, we can update the image with the same *pull* command again.

   On the contrary, with digest, the image is fixed and pinned to that exact version. This makes sure you are interacting with the exact image. However, upcoming security fixes are also missed.

   To get image digest, we either go to the official registry, use `images --digests`, or even *inspect* command.
4. Any any time, Ctrl-C terminates the pull process.
5. Docker support [proxy configuration](https://docs.docker.com/network/proxy/) when feteching the images.

# Create a Container

Syntax:

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

Example:

```bash
root@tux ~ # docker run --name centos-5.8 \
-d \
-it \
--rm \
--mount type=bind,source=/home/jim/workspace/,target=/home/jim/workspace/ \
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
   root@tux ~ # docker run -t --rm ubuntu ls /
   bin   dev  home  media  opt   root  sbin  sys  usr
   boot  etc  lib   mnt    proc  run   srv   tmp  var
   
   root@tux ~ # docker run --rm ubuntu ls /
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
   root@tux ~ # docker run -d ubuntu bash -c "tail -f /dev/null"
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
6. `-w` lets the root process running inside the given directory that is created on demand.
7. `--net, --network` connects the container to a network. By default, it is *bridge*. Details are discussed in later sections.
8. `-u, --user` runs the root process as a non-root user. Attention that, the username is that within the container. So the image creator should create that name in Dockerfile.
9. *bash* overrides the CMD/ENTRYPOINT instructions of the image.

Here is a note about the different options:

![docker run](/assets/docker-run-adit.jpg)

Please also follow this post [Cannot pipe to docker run with stdin attached](https://stackoverflow.com/q/71761103/2336707).

# Data Share

Docker containers can read from or write to pathnames, either on host or on memory filesystem - to share data. There are [three storage types](https://docs.docker.com/storage/):

1. Volume, Data Volume, or Named Volume.

   By default, running data of a container is layered on top of the image used to create it. A *volume* decouples that data from both the host or the container. Just think of a Windows partition or removable disk drive.

   Volumes are managed by Docker and persist. Data within can be shared among multiple containers, as well as between the host and a container.
2. Bind Mount.

   Bind-mount a file or directory in the host to a file or directory in the container. The target can be read-only or read-write. For example, bind host */etc/resolv.conf* to a container, sharing name servers.
3. `tmpfs` Mount.

   Needless to say, *tmpfs* is a memory filesystem that let container stores data in host memory.

Option `--volume , -v` and `--mount` both support volume and (bind/tmpfs) mount, though we recommend `--mount` as it is more verbose.

If the file or directory on the host does not exist. `--volume` and `--mount` behaves differently. `--volume` would create the pathname as a *directory*, NOT a file, while `--mount` would report error. On the contrary, if the target pathname already exists in the container, both options would *obsecure* contents over there. This is useful if we'd like to test a new version of file/directory without rebuild a new image. Another difference between the two options is `--mount` support all three storage types but `--volume` only support Bind Mound.

## [SELinux](https://stackoverflow.com/q/24288616)

When bind-mount a file or mount a directory of host, SELinx policy in the container may restrict access to the shared pathname.

1. Temporarily turn off SELinux policy: 

   ```bash
   root@tux ~ # su -c "setenforce 0"
   root@tux ~ # docker restart container-ID
   ```

2. Adding a SELinux rule for the shared pathname:

   ```bash
   root@tux ~ # chcon -Rt svirt_sandbox_file_t /path/to/
   ```

3. Pass argument `:z` or `:Z` to `--volume` option:

   ```bash
   -v /root/workspace:/root/workspace:z
   ```

   Attention please, `--mount` does not support this.
4. Pass `--privileged=true` to *docker run*.

   However, this method is discouraged as privileged containers bring in security risks. If it is the last resort, first create a privileged container and then create a non-priviledged container inside.

   Privilege permissions can have fine-grained control by `--cap-add` or `--cap-drop`, which is recommended!

# Manage a Nginx Container

```bash
root@tux ~ # docker run --name webserver \
-d \
--net host
--mount type=bind,source=/tmp/logs,target=/var/log/nginx \
-p 8080:80 nginx

root@tux ~ # docker container ls
root@tux ~ # docker container logs webserver
root@tux ~ # docker container stop/kill webserver
root@tux ~ # docker container start webserver
root@tux ~ # docker container rm webserve          # remove one or more container (even running instances)
root@tux ~ # docker container prune                # remove all stopped container
```

1. `-p` maps host port 8080 to container port 80 that is already bound to host Nginx process.
2. Check the Dockerfile, there is a line telling how Nginx should be started:

   ```
   CMD ["nginx", "-g", "daemon off;"]
   ```

3. The `--mount` type is a [Bind Mount](#data-share) directory.
4. Visit the Nginx container page at *http://host-ip:8080*.
5. *stop* attempts to trigger a [*graceful*](https://superuser.com/a/757497) shutdown by sending the standard POSIX signal SIGTERM, whereas *kill* just kills the process by sending SIGKILL signal.

# Get into container

Exec a simple command in container:

```bash
root@tux ~ # docker exec webserver sh -c 'echo $PATH'
root@tux ~ # docker exec webserver ps
```

Sometimes, we want to exec the command in the background when we do not care about its output or need not any input:

```bash
root@tux ~ # docker exec -d webserver touch /tmp/test.txt
```

If we want to interactively and continuously control the container:

```bash
root@tux ~ # docker exec -it webserver bash
```

Apart from *docker exec*, [docker attach <container>](https://docs.docker.com/engine/reference/commandline/attach/) is also recommended.

This command attaches the host terminal's STDIN, STDOUT and STDERR files to a running container, allowing interactive control or inspect as if the container was running directly in the host's terminal. It will display the output of the ENTRYPOINT/CMD process.

For example, a container can be shutdown by `C-c` shortcut, sending the SIGINT signal (identical to SIGTERM) to the container by default. Rather, `C-p C-q` detaches from the container and leave it running in the background again.

If the process is running as PID 1 (like */usr/bin/init*), it ignores any signal and will not terminate on SIGINT or SIGTERM unless it is coded to do so.

# Create Image from Container #

```bash
root@tux ~ # docker exec -it webserver bash
#
root@docker ~ # echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
root@docker ~ # exit
#
root@tux ~ # docker diff webserver
root@tux ~ # docker container commit -a 'jim' -m 'change front page' webserver nginx:v2
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
5. We can *inspect* the target image, and will find "ContainerConfig" and "Config". They are almost identical.

   "ContainerConfig" is the config of current container from within this image is committed, while the "Config" is the exact configuration of the image. Pay attention to the [Cmd](#exec-and-shell) parts. If we [build by Dockerfile](#build-image-by-dockerfile), then they looks different. The ContainerConfig this is the temporary container spawned to create the image. Check [what-is-different-of-config-and-containerconfig-of-docker-inspect](https://stackoverflow.com/q/36216220).

# [Networking Drivers](https://docs.docker.com/network/)

The Docker's networking subsystem is *pluggable*, using drivers. Below is a simple explanation:

1. Bridge

   The *default* driver if none is given upon *run*, providing network isolation from the outside. It is to bridge traffic among multiple containers and the host. Check the image below, *docker0* is the bridge interface.

   ![docker netowrk](/assets/docker-net.png)

   Please pay attention: this is different from the *bridge* mode of VMWare or VirtualBox (real Virtual Machine). VMWare and VirtualBox's bridge is deployed directly on the host's interface and appears to be a real physical device parallel to the host and can be connected to directly from within LAN.
2. Host

   Share the host's networking directly without isolation. However, LAN devices cannot differntiate between containers and the host as there is not individual IP addressed assigned to containers.

   The host mode is preferred when the service exposes a port publicly like Nginx servers.
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

# Build Image by Dockerfile

From *commit* example above, we can create new image layer but many negligible commands like *ls*, *pwd*, etc. are recourded as well.

Similar to Makefile, Docker uses Dockerfile to define image with specified *instruction*s like FROM, COPY, RUN etc. Each instruction creates a new intermediate layer and a new intermediate image. In order to minimize number of layers and images, we'd better merge instructions as much as possible.

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

Now we build the image:

```bash
root@tux ~ # docker build -t nginx:v3 -t nginx .
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
   root@tux ~ # docker build -t nginx:v3 - < /path/to/Dockerfile
   ```

   The hypen character cannot be omitted!
4. If there exist multiple CMD/ENTRYPOINT instructions from different layers, only that of the topmost layer will be executed upon container start. All the rest CMD/ENTRYPOINT are overriden.
5. After the building, we can run *nginx:v3* image:

   ```bash
   root@tux ~ # docker run --name web3 -d -p 8081:80 --rm nginx:v3
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

9. Check [multi-platform-docker-build](https://github.com/BretFisher/multi-platform-docker-build) for AMD64/ARM64 arch issue. This also applies to [Create a Container](#create-a-container).

Refer to [Best practice for writing Dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/).

# Share an Image

We can [share](https://docs.docker.com/get-started/04_sharing_app/) our own docker image to a registry (e.g. docker.io) by *docker push*.

When pushing an image to remote repository, we should login and specify the registry account.

```bash
# token recommended; avoid password
~ $ docker login -u myaccount

~ $ docker push myaccount/nginx
Using default tag: latest
The push refers to repository [docker.io/myaccount/docker-getting-started]
An image does not exist locally with the tag: myaccount/docker-getting-started
```

The error shows there is not such docker image. So, we should assign a new tag to an image:

```bash
~ $ docker tag nginx myaccount/nginx
```

You will find the two tags refer to the same image ID by *docker images*. Here is an example:

```bash
~ $ docker images
REPOSITORY                        TAG                                        IMAGE ID       CREATED             SIZE
docker-getting-started            latest                                     862614378b4c   About an hour ago   430MB
outsinre/docker-getting-started   latest                                     862614378b4c   About an hour ago   430MB
```

After new tagging, we can push: 

```bash
~ $ docker push myaccount/nginx
```

Remember that if we want to use a different registry rather than the default *docker.io*, then add the registry to the new tag as well as follows.

```bash
~ $ docker tag nginx myregistry:5000/myaccount/nginx
``

# Docker Compose #

todo https://docs.docker.com/compose/compose-file/
