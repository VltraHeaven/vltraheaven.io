---
title: Down the Rabbit Hole - Kata Containers
lang: en-US
display: home
cover: img/nihon_warning.jpg
description: An in-depth guide to installing and configuring Kata Containers from source.
date: 2019-07-31
tags:
    - containers
    - virtualization
    - linux
keywords:
    - devops
---
---

Source: [Kata Containers](https://katacontainers.io) | [@Github](https://github.com/kata-containers)


It's not uncommon for me on any given day to get completely pulled in by some emerging (or legacy) alternative technology. My rabbit hole *du jour* ended up being Kata Containers. The original objective was only to make a few settings modifications, which quickly turned into me tanking my entire install.

...fantastic.

The Computer Gods had not shown on me this day, or so I thought. The steps to reconstructing my Kata Containers installation ended up being a huge learning experience in disguise. Who says it doesn't pay to go off the beaten path.

If you're not familiar with the technology, here's a *"little"* introduction.
## Primer
---

Kata Containers is an open source project supported by the collaborative efforts of the [OpenStack](https://www.openstack.org/software) Foundation and Intel with the objective of creating a hybrid linux container technology; fusing the isolation and relative security of virtual machines with the lightweight profile and ephemeral instrumentation of linux containers. Built upon Intel's earlier virtualization effort "Clear Containers" and [Hyper.sh](https://github.com/hyperhq)'s container runtime `runv`, the Kata Containers technology is architected using QEMU as a hypervisor to spin up a lightweight virtual machine that within itself creates linux containers. The `kata-runtime` (or kata shim in the case of Kubernetes) sits on the host and communicates with the `kata-proxy`, which in turn issues commands through the hypervisor to the kata agent sitting on the kernel of the virtual machine. This chain of communication allows the host container management or orchestration software (Docker, Kubernetes, etc.) to create it's linux containers within the VM environment.

There are several deployment options for Kata Containers, providing [automatic](https://github.com/kata-containers/documentation/blob/master/install/installing-with-kata-manager.md) (via the `kata-manager`), [manual](https://github.com/kata-containers/documentation/blob/master/install/README.md#manual-installation) and [scripted](https://github.com/kata-containers/documentation/blob/master/install/README.md#scripted-installation) installation steps for [several linux distros](https://github.com/kata-containers/documentation/blob/master/install/README.md#supported-distributions) as well as using [snap](https://github.com/kata-containers/documentation/blob/master/install/README.md#snap-installation) as a universal installation method. Unfortunately, my distro of choice wasn't on the aformentioned list. So in lieu of installing the snap package I opted to do this on "[Dante must Die](https://www.goombastomp.com/dante-must-die-how-devil-may-cry-perfected-difficulty/)" difficulty and build the whole shebang from source. Can you tell i'm not the biggest fan of snap packages yet?

## Dependencies
---
As the [documentation](https://github.com/kata-containers/documentation/blob/master/Developer-Guide.md#initial-setup) states, first we gotta cover our dependencies, namely [`go`](https://golang.org/dl), `make`, `git` and `gcc`. All these should be easily obtainable using any  distro's package manager. Now, since we're working with `go`, we gotta set our `$GOPATH`.
>```zsh
>$ echo "export GOPATH=$HOME/go" >> ~/.zshrc
>```
Feel free to replace .zshrc with .bashrc if you use `bash`. The `fish` users can just go...
>```sh
>$ set -x -U GOPATH $HOME/go
>```
Chances are this setting was automatically set if you installed `go` from your package manager. If you're not sure just type...
>```sh
>$ echo $GOPATH
>```
...and if `/home/${user}/go` doesn't pop on the terminal screen then follow the former instructions.

## Installing the `kata-runtime`, shim and `kata-proxy`
---

Next we'll be pulling the `kata-runtime` sources from github using the `go get` command.
>```sh
>$ go get -d -u github.com/kata-containers/runtime
>```
Now, you may get an error telling you `...no Go files in...', but you can ignore that. Move to the source code's directory and compile the `kata-runtime` using `make`.
>```sh
>$ cd $GOPATH/src/github.com/kata-containers/runtime
>$ make && sudo -E PATH=$PATH make install
>```
This drops in the runtime as `/usr/local/bin/kata-runtime` and the kata configuration file as `/usr/share/defaults/kata-containers/configuration.toml` in your system. To make sure all is well run a quick 
>```sh
>$ sudo kata-runtime kata-check
>```
...to make sure were looking good. If not, you'll get a brief error and explanation on your terminal. If you get `command not found`, make sure that `/usr/local/bin` is displayed when you type
>```sh
>echo $PATH
>```
into the terminal. If not, do...
>```sh
>echo "export PATH=/usr/local/bin:$PATH" >> ~/.zshrc
>```
to add the directory to your `$PATH`, again replacing `.zshrc` with the configuration file for your shell. Fish is a little different...
>```sh
>set -U -x PATH /usr/local/bin $PATH
>```
but if you're using fish i'm probably preaching to the choir.

Next, let's provide the means of communications between the VM and the container management/orchestration software installed on the host, the `kata-proxy`
>```sh
>$ go get -d -u github.com/kata-containers/proxy
>$ cd $GOPATH/src/github.com/kata-containers/proxy && make && sudo make install
>```
...and the Kata `shim` incase we want to use Kata Containers with Kubernetes in the future.
>```sh
>$ go get -d -u github.com/kata-containers/shim
>$ cd $GOPATH/src/github.com/kata-containers/shim && make && sudo make install
>```

## Building the `rootfs` and guest kernel
---

Now, since we are isolating the guest containers away from our host kernel in a VM, we'll need to provide the essentials to get the VM and containers to run; a root file system image, a kernel and the hypervisor. For that we'll be using the `osbuilder` scripts to build our guest vm up from scratch.
>```sh
>$ go get -d -u github.com/kata-containers/osbuilder
>```
Whenever we run `go get` to fetch source code from github, the downloaded files are always deposited in the `$HOME/go/src/github.com` directory. This will come into play later...

In addition to the aformentioned dependencies, i'll be using `docker` as a build environment for the following steps. Chances are you at least have that installed. If not, dig through [here](https://duckduckgo.com/?q=docker+install&t=ffnt&ia=about) and see if you can find an appropriate install guide for your host. If you do have docker installed and you have taken advantage of other runtimes in the past, make sure that `runc` is set as your `default-runtime` in the `/etc/docker/daemon.json` file.
> ```json
> {
>      "default-runtime": "runc" 
> }
>``` 

Kata containers can operate using either an [initrd](https://en.wikipedia.org/wiki/Initial_ramdisk) or [rootfs](https://serverfault.com/questions/275988/what-is-rootfs) image. For the sake of relative simplicity, i'll be covering using rootfs. Here are instructions covering [initrd image creation](https://github.com/kata-containers/documentation/blob/master/Developer-Guide.md#create-an-initrd-image---optional). Let's set the configuration file.
>```sh
>$ sudo mkdir -p /etc/kata-containers/
>$ sudo install -o root -g root -m 0640 /usr/share/defaults/kata-containers/configuration.toml /etc/kata-containers
>$ sudo sed -i 's/^\(initrd =.*\)/# \1/g' /etc/kata-containers/configuration.toml
>```
Feel free to `$ sudo vim /etc/kata-containers/configuration.toml` and make sure the line beginning with `initrd =` is commented(`#`) out. Having both the `image =` line and the `initrd =` line uncommented will cause an error with the runtime upon operation. Now to create a local rootfs using the `rootfs.sh` script in `$GOPATH/src/github.com/kata-containers/osbuilder/rootfs-builder`.

>```sh
>$ export ROOTFS_DIR=${GOPATH}/src/github.com/kata-containers/osbuilder/rootfs-builder/rootfs
>$ sudo rm -rf ${ROOTFS_DIR}
>$ cd $GOPATH/src/github.com/kata-containers/osbuilder/rootfs-builder
>$ script -fec 'sudo -E GOPATH=$GOPATH USE_DOCKER=true SECCOMP=no ./rootfs.sh ${distro}'
>```
Now the documentation states that in the script above, `${distro}` must be replaced with one of the following linux distributions before running:

  * `alpine`

  * `fedora`
  
  * `euleros`
  
  * `centos`
  
  * `clearlinux`

...but what it __*doesn't*__ tell you is that when we build the rootfs image in the next step, the script defaults to `fedora`. I originally tried `alpine` for the local rootfs and had errors when building the rootfs image. Since then i've just defaulted to `fedora` to avoid random build issues. Feel free to experiment at your leisure though.

Now we'll build the image...
>```sh
>$ cd $GOPATH/src/github.com/kata-containers/osbuilder/image-builder
>$ script -fec 'sudo -E USE_DOCKER=true ./image_builder.sh ${ROOTFS_DIR}'
>```
and install it.
>```sh
>$ commit=$(git log --format=%h -1 HEAD)
>$ date=$(date +%Y-%m-%d-%T.%N%z)
>$ image="kata-containers-${date}-${commit}"
>$ sudo install -o root -g root -m 0640 -D kata-containers.img "/usr/share/kata-containers/${image}"
>$ (cd /usr/share/kata-containers && sudo ln -sf "$image" kata-containers.img)
>```
Done.


Now let's work on building us a kernel. Let's run `$ go get -d /github.com/kata-containers/packaging` to get the scripts we need for the build. We'll change directories into the `kernel` directory and pass the appropriate arguments to the `build-kernel.sh` script to get the job done.
>```sh
>$ cd ${GOPATH}/src/github.com/kata-containers/packaging/kernel
>$ ./build-kernel.sh setup
>$ ./build-kernel.sh build
>$ ./build-kernel.sh install
>```
Now, when you ran `./build-kernel.sh install`, did you have any issues? If you're doing this as a user or administrator on your machine, you may have. The script installs the kernel into `/usr/share/kata-containers`, a directory that you may not be able to change without root privileges. Running the script using `sudo` won't work either. My fix was...
>```sh
>$ sudo chown -R ${username}:${username} /usr/share/kata-containers
>$ ./build-kernel.sh install
>$ sudo chown -R root:root /usr/share/kata-containers
>```
replacing `${username}` with the name of your user account. This temporarily gives you ownership of the directory and its subdirectories and hands it back to root after the script is complete. Groovy. On to the real fun, installing QEMU, our hypervisor.

## Building QEMU from source
---
The Documentation states that we'll either need to have a QEMU directory with source code ready or we can opt to use the Kata Containers fork `qemu-lite` and build using the `configure-hypervisor.sh` script.
Seems easy enough, we just run `$ go get -d github.com/kata-containers/qemu`, switch branches using `git` and set a few aliases...
>```sh
>$ qemu_branch=$(grep qemu-lite- ${GOPATH}/src/github.com/kata-containers/runtime/versions.yaml | cut -d '"' -f2)
>$ cd ${GOPATH}/src/github.com/kata-containers/qemu
>$ git checkout -b $qemu_branch remotes/origin/$qemu_branch
>$ your_qemu_directory=${GOPATH}/src/github.com/kata-containers/qemu
>```
now `go get` the `github.com/kata-containers/packaging` repo and make the `kata.cfg` configuration file...
>```sh
>$ go get -d github.com/kata-containers/packaging
>$ cd $your_qemu_directory
>$ ${GOPATH}/src/github.com/kata-containers/packaging/scripts/configure-hypervisor.sh qemu > kata.cfg
>$ eval ./configure "$(cat kata.cfg)" --python=$(which python2)
>```
run `$ make -j $(nproc)` to compile the source code and...
```
util/memfd.c:40:12: error: static declaration of ‘memfd_create’ follows non-static declaration
   40 | static int memfd_create(const char *name, unsigned int flags)
      |            ^~~~~~~~~~~~
In file included from /usr/include/bits/mman-linux.h:111,
                 from /usr/include/bits/mman.h:34,
                 from /usr/include/sys/mman.h:41,
                 from /home/heaven/go/src/github.com/kata-containers/qemu/include/sysemu/os-posix.h:29,
                 from /home/heaven/go/src/github.com/kata-containers/qemu/include/qemu/osdep.h:104,
                 from util/memfd.c:28:
/usr/include/bits/mman-shared.h:50:5: note: previous declaration of ‘memfd_create’ was here
   50 | int memfd_create (const char *__name, unsigned int __flags) __THROW;
      |     ^~~~~~~~~~~~
make: *** [/home/heaven/go/src/github.com/kata-containers/qemu/rules.mak:66: util/memfd.o] Error 1
make: *** Waiting for unfinished jobs....
```
...wait, what? 

Man, I should have known documented steps pre-empted with the ominous disclaimer "Developers and hackers only" couldn't be this easy.
Now, depending on your host distro and kernel, that `make` might work for you, but it came out snakes eyes for me. So after spamming the aformentioned `make` command in disbelief a few times, then hitting the search engines, it looks like this is a [known issue](https://lists.gnu.org/archive/html/qemu-devel/2017-11/msg05016.html) in the build process for earlier versions of QEMU. This issue actually works in our best interests since we __should__ be using the latest sources for qemu. Having the most up-to-date sources is critical since our hypervisor is acting as a defense mechanism against untrusted and potentially malicious containers.
>```sh
>$ go get -d github.com/qemu/qemu
>$ cd $GOPATH github.com/qemu/qemu
>$ your_qemu_directory=${GOPATH}/src/github.com/qemu/qemu
>$ ${GOPATH}/src/github.com/kata-containers/packaging/scripts/configure-hypervisor.sh qemu > kata.cfg
>$ eval ./configure "$(cat kata.cfg)" --python=$(which python2)
>```

It's possible the command above will throw an error such as...
> ```bash
> ERROR: unknown option --disable-libssh2
> ```
If so, note the unknown flag and use reasoning to try to resolve the issue. For this particular error I ran `vim ./kata.cfg`
and switched `--disable-libssh2` to `--disable-libssh`, then saved and the configuration ran smoothly.

Now to make and install qemu.
>```sh
>$ make -j $(nproc)
>$ sudo -E make install
>```
Done! 

## Wrapping up
---
Well...*almost*. Since we're using docker we need to tell the daemon that we have a new runtime and where to find it. I'm also going to make the `kata-runtime` my default runtime and set my storage driver to `overlay2`. We can do this in `/etc/docker/daemon.json`
>```sh
>$ vim /etc/docker/daemon.json
>```
>```json
>    {
>        "default-runtime": "kata-runtime",
>        "storage-driver": "overlay2",
>        "runtimes": {
>           "kata-runtime": {
>                "path": "/usr/local/bin/kata-runtime"
>            }
>        }
>    }
>```
>```sh
>$ sudo systemctl daemon-reload
>$ sudo systemctl restart docker.service docker.socket
>```
Awesome. Now if we run a simple `docker run` command, we should be greeted with a fresh shell.
>```sh
>$ docker run -it --rm alpine:latest
>/ #_
>```
Nice.

Now that we've got Kata fully installed, we can [hack](https://blog.ropnop.com/docker-for-pentesters/) around with it, try out container images of [distros we've never used](https://hub.docker.com/r/opensuse/leap), deploy a [web application](https://hub.docker.com/_/wordpress) or set up a [VPN interface](https://github.com/activeeos/wireguard-docker) in a docker container all with increased isolation from the host kernel and filesystem. Also, since we are working with a VM here, feel free to tweak the `default_vcpus` and/or `default_memory` parameters within the `/etc/kata-containers/configuration.toml` config file, along with any other tweaks you see fit. There's a chance you *could* break something, but you can always delete the config file, copy `/usr/share/defaults/kata-containers/configuration.toml` into `/etc/kata-containers/`, comment out the `initrd =` parameter, run `sudo systemctl restart docker.service` and you're back in business. Who knows what you might discover...

Even more interesting is the possibility of leveraging Kata Containers with amazon's __[Firecracker](https://firecracker-microvm.github.io/)__ "microVM" virtualization technology that leverages a KVM hypervisor and reportedly has even more relative security than using the default QEMU hypervisor, though there are a few [limitations](https://github.com/kata-containers/documentation/issues/351). This is what powers AWS Lambda and AWS Fargate so it might be worth a gander.

Feel free to check out the [installation docs](https://github.com/kata-containers/documentation/wiki/Initial-release-of-Kata-Containers-with-Firecracker-support). I decided to save that for another time, I already went down my rabbit hole for the day. 

*-Heaven*
