docker-debian-cuda
==================

***docker-debian-cuda*** is a minimal [*Docker*](http://www.docker.com/) image built from *Debian 9* (amd64) with [*CUDA Toolkit*](http://developer.nvidia.com/cuda-toolkit) and *cuDNN* using only Debian packages.

Although the vendor specific *nvidia-docker* tool can run CUDA inside Docker images, it performs the same thing in a less transparent way and is incompatible with other Docker tools. Instead of using yet another wrapper command, we explicitly expose GPU devices and inject the host's *CUDA Driver* library. The latest image starts from the official Debian image and follows the NVIDIA `deb (network)` instalation steps for *Ubuntu 17.04*.

Open source project:

- <i class="fa fa-fw fa-home"></i> home: <http://gw.tnode.com/docker/debian-cuda/>
- <i class="fa fa-fw fa-github-square"></i> github: <http://github.com/gw0/docker-debian-cuda/>
- <i class="fa fa-fw fa-laptop"></i> technology: *debian*, *cuda toolkit*, *cudnn*, *opencl*
- <i class="fa fa-fw fa-database"></i> docker hub: <http://hub.docker.com/r/gw000/debian-cuda/>

Available tags (based on *Debian 9/stretch* and NVIDIA `deb (network)` installation without *CUDA Driver*):

- `latest` points to `9.1_7.0`
- `9.1_7.0`, `9.1.85-1_7.0.5.15-1`, `9.0_7.0`, `9.0.176-1_7.0.5.15-1` [2018-02-15]: *CUDA Toolkit* <small>(9.1.85-1/9.0.176-1)</small> + *cuDNN* <small>(7.0.5.15-1)</small> ([*Dockerfile*](http://github.com/gw0/docker-debian-cuda/blob/master/Dockerfile))

Available tags (based on only *Debian 9/stretch* packages, also for Nvidia CUDA Toolkit):

- `8.0.44-4_6.0.21-1_375.82-1`, `8.0_6.0`, `8.0.44-4_7.0.4.31-1_375.82-1`, `8.0_7.0` [2017-12-01]: *CUDA Toolkit* <small>(8.0.44-4)</small> + *cuDNN* <small>(6.0.21-1/7.0.4.31-1)</small> + *CUDA Driver* <small>(375.82-1)</small> ([*Dockerfile*](http://github.com/gw0/docker-debian-cuda/blob/master/Dockerfile))
- `8.0.44-3_5.1.10-1_375.66-1`, `8.0_5.1` [2017-05-31]: *CUDA Toolkit* <small>(8.0.44-3)</small> + *cuDNN* <small>(5.1.10-1)</small> + *CUDA Driver* <small>(375.66-1)</small> ([*Dockerfile*](http://github.com/gw0/docker-debian-cuda/blob/master/Dockerfile))
- `8.0.44-3_5.1.10-1_375.39-1` [2017-03-27]: *CUDA Toolkit* <small>(8.0.44-3)</small> + *cuDNN* <small>(5.1.10-1)</small> + *CUDA Driver* <small>(375.39-1)</small>
- `8.0.44-2_5.1.5-1_375.20-4` [2016-12-21]: *CUDA Toolkit* <small>(8.0.44-2)</small> + *cuDNN* <small>(5.1.5-1)</small> + *CUDA Driver* <small>(375.20-4)</small>
- `7.5.18-4_5.1.3_361.45.18-2`, `7.5_5.1` [2016-09-19]: *CUDA Toolkit* <small>(7.5.18-4)</small> + *cuDNN* <small>(5.1.3)</small> + *CUDA Driver* <small>(361.45.18-2)</small>
- `7.5.18-2` [2016-07-20]: *CUDA Toolkit* <small>(7.5.18-2)</small> + *cuDNN* <small>(4.0.7)</small> + *CUDA Driver* <small>(352.79-8)</small>


Makefile Usage
==============

If you want to use the GPU, first `make build-cuda` then `make build-gpu`.

If you just want to use CPU, `make build-cpu`.

```bash
$ make help
help                           This help.
build-cuda                     Build Debian + Cuda Docker
build-gpu                      Build Python3 + TensorFlow + Jupyter + GPU Docker
build-cpu                      Build Python3 + TensorFlow + Jupyter + CPU Docker
```


Usage
=====

Host system requirements (eg. Debian 9 or similar Ubuntu):

- GPU card with CUDA Compute Capability 3.5 or higher
- *NVIDIA Kernel Driver* (`nvidia-kernel-dkms`)
- *CUDA Driver* library (`libcuda1`, same version as *NVIDIA Kernel Driver*)
- optionally `nvidia-smi`, `nvidia-opencl-icd`

To utilize your GPUs this Docker image needs access to your `/dev/nvidia*` devices and also the correct version of *CUDA Driver*, like:

```bash
$ docker run -it --rm $(ls /dev/nvidia* | xargs -I{} echo '--device={}') $(ls /usr/lib/x86_64-linux-gnu/{libcuda,libnvidia}* | xargs -I{} echo '-v {}:{}:ro') gw000/debian-cuda
```

The additional parameters in above command explicitly expose your GPU devices and *CUDA Driver* library from the host system into the container. The vendor specific *nvidia-docker* tool performs the same thing in a less transparent way and is incompatible with other Docker tools.


Host system
===========

List of devices that should be present on the host system:

```bash
$ ll /dev/nvidia*
crw-rw---- 1 root video 250,   0 Jul 13 15:56 /dev/nvidia-uvm
crw-rw---- 1 root video 250,   1 Jul 13 15:56 /dev/nvidia-uvm-tools
crw-rw---- 1 root video 195,   0 Jul 13 15:56 /dev/nvidia0
crw-rw---- 1 root video 195, 255 Jul 13 15:56 /dev/nvidiactl
```

In case `/dev/nvidia0` and `/dev/nvidiactl` are not present, ensure the kernel module `nvidia` is automatically loaded, properly configured, and there is a *udev* rule to create the devices:

```bash
$ echo 'nvidia' > /etc/modules-load.d/nvidia.conf
$ cat > /etc/udev/rules.d/70-nvidia.rules << __EOF__
KERNEL=="nvidia", RUN+="/bin/bash -c '/usr/bin/nvidia-smi -L && /bin/chmod 0660 /dev/nvidia* && /bin/chgrp video /dev/nvidia*'"
__EOF__
```

For *OpenCL* support the devices `/dev/nvidia-uvm` and `/dev/nvidia-uvm-tools` are needed. Ensure the kernel module `nvidia-uvm` is automatically loaded, and add a custom *udev* rule to create the device:

```bash
$ echo 'nvidia-uvm' > /etc/modules-load.d/nvidia-uvm.conf
$ cat > /etc/udev/rules.d/70-nvidia-uvm.rules << __EOF__
KERNEL=="nvidia_uvm", RUN+="/bin/bash -c '/usr/bin/nvidia-modprobe -c0 -u && /bin/chmod 0660 /dev/nvidia-uvm* && /bin/chgrp video /dev/nvidia-uvm*'"
__EOF__
```

If you would like to monitor real-time temperatures on your host system use something like:

```bash
$ watch -n 5 'nvidia-smi; echo; sensors; for hdd in /dev/sd?; do echo -n "$hdd  "; smartctl -A $hdd | grep Temperature_Celsius; done'
```

In case your *NVIDIA Kernel Driver* and *CUDA Driver* versions differ an error appears in kernel messages (`dmesg`) or using `nvidia-smi` inside the container. Possible solutions:

- upgrade your Nvidia kernel driver on the host directly from *Debian 9* packages: [nvidia-kernel-dkms](https://packages.debian.org/stretch/amd64/nvidia-kernel-dkms), [nvidia-alternative](https://packages.debian.org/stretch/amd64/nvidia-alternative), [libnvidia-ml1](https://packages.debian.org/stretch/amd64/libnvidia-ml1), [nvidia-smi](https://packages.debian.org/stretch/amd64/nvidia-smi)
- upgrade your Nvidia kernel driver on the host by compiling it yourself
- inject the correct version of CUDA Driver into the container as mentioned above (if it is installed on the host)


Decision against *nvidia-docker*
--------------------------------

It is true, that Nvidia recommends to use their `nvidia-docker` command as part of their vendor lock-in strategy. In reality the `nvidia-docker` is nothing more than a fancy wrapper that runs the `docker` command with the additional parameters to mount the device and host libraries into the container. Latest Docker introduced runtimes and `nvidia-container-runtime` should make things work, but unfortunately this is not supported by `docker-compose` and other tools.

Pros for *nvidia-docker* tool:

- shorter command (no need to remember those additional parameters)

Cons for *nvidia-docker* tool:

- yet another tool that administrators need to learn (why bother administrators to learn anything more than `docker run`?)
- less transparent what is being executed (some believe some "black magic" happens behind `nvidia-docker` that handles 2 instances on same GPU better, although it works exactly the same)
- not possible to use with `docker-compose` and other tools for managing Docker containers
- only Nvidia GPUs are supported (what if someone would want to use a GPU from another vendor? or a FPGA device?)
- no support for OpenCL
- vendor lock-in


Feedback
========

If you encounter any bugs or have feature requests, please file them in the [issue tracker](http://github.com/gw0/docker-debian-cuda/issues/) or even develop it yourself and submit a pull request over [GitHub](http://github.com/gw0/docker-debian-cuda/).


License
=======

Copyright &copy; 2016-2018 *gw0* [<http://gw.tnode.com/>] &lt;<gw.2018@ena.one>&gt;

All code is licensed under the [GNU Affero General Public License 3.0+](LICENSE_AGPL-3.0.txt) (AGPL-3.0+). Note that it is mandatory to make all modifications and complete source code publicly available to any user.
