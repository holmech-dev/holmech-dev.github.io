---
layout: post
---
This post outlines how to create a Docker environment with ROS middleware setup.

Docker offers a great amount of portability when it comes to preparing software environments, this lends well when developing applications with middleware such as ROS (Robotics Operating System) where managing a large amount of software libraries and portability maybe necessary. 

To create the Docker environment a Docker image is needed, this will serve as our template or environment snapshot for creating our Docker container. A container is the instantiation of this image that we can work in. To build the image, we first need a way to instruct the build process what should be contained in the image, this is done with a Dockerfile. 

![Docker Image](/docker_image_container.png)

I use knowledge about my system requirements to create the Dockerfile. For this build, I have the following requirements. 
<ul>
  <li>Operating system compatibility - Ubuntu 20</li>
  <li>Nvidia GPU utilisation - Native PC driver and CUDA version</li>
  <li>ROS Version - Noetic</li>
</ul>

First, let's create a Dockerfile. Create this file in a suitable directory necessary to store the docker files for you project. 
```console
holmech@linux-desktop:~$ touch Dockerfile.base
```
## The Dockerfile
In a text editor if your choosing, the Dockerfile can now be edited. I'm using VS Code.
Using the ```FROM``` command, a base image is specified as a starting point, this can be identified from [Docker hub](https://hub.docker.com/). Then to assist with the build process, the ```USER``` is specified. The last argument in the file facilitates install of some libraries without necessity to interact with the user during the process. More information on this can be found below. 

<https://stackoverflow.com/questions/63476497/docker-build-with-ubuntu-18-04-image-hangs-after-prompting-for-country-of-origin>

Then, we can run ```apt-get``` to try and install a system update to validate packages can be successfully installed.

```
FROM nvidia/cuda:11.1.1-devel-ubuntu20.04

USER root

# disable interactive configuration mode
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update \
    && \
    apt-get install -y
```
Save the Dockerfile. 

## The Image

We then need to try and build the image for the first time. Using the ```build``` command, the ```-t``` argument is used to name the image. This name takes the username first followed by the image name. The ```-f``` argument points the build process to the file including the desired build instructions, in this case the ```Dockerfile.base``` file just created.

```console
holmech@linux-desktop:~$ docker build -t holmec/noetic-base -f Dockerfile.base .
```

The command may take some time to complete. A successful build will display no errors in the terminal window output. Then the existence of our new image can be validated using the following command. 
```console
holmech@linux-desktop:~$ docker image ls
```
This will show the image name, in this case ```holmec/noetic-base```, the tag is shown in the next column but as we didn't explicitly define the tag, this will probably show as ```latest```. There will also be an image ID, the time period passed since the image was created and finally the size of the image. 

## The Container

As a quick test, a container can easily be instantiated from our image using the ```docker run``` command, the full command is shown below. The ```--rm``` argument instructs docker to remove the container after it is closed and the ```-it``` argument is short for interactive with tty, this allows your terminal window to interact with the container. 

```console
holmech@linux-desktop:~$ docker run --rm -it holmec/noetic-base
```

The terminal window will then update and the prompt will display ```root``` as the user. In a seperate terminal the ```docker container ls``` command can be used to display all current running containers. ```ctrl + d``` will exit the container, notice that running ```docker container ls``` again will show that the container is no longer in existence. Take care, any changes that will have been made whilst the container was open will now have been lost.

Great, we have a full working pipeline and a good starting point. Reviewing the system requirements, the first requirement to use Ubuntu 20 as the operating system is now complete as we used this for our base image. This can be validated by running ```uname -a``` inside the container.
The next requirement is to get access to the native Nvidia GPU driver and CUDA toolkit. See my driver and CUDA versions below for reference. 
  <li>Driver Version: 525.147.05</li>
  <li>CUDA Version: 12.0</li>
We can use a command to access the Nvidia GPU when running a container, but first we need to ensure the NVIDIA Container Toolkit is installed inside the linux operating system. Follow the ([Installing the NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)) link for instructions if you're unsure. 

As some additional arugments will now be add when running a contrainter, it's easier to create a shell script for this. ```touch run.sh```.

This is my run file. 

```
#!/bin/bash

xhost +

docker run -it --rm\
    --runtime=nvidia \
    --gpus all \
    --net=host \
    --env="DISPLAY" \
    --env=TERM="xterm-color" \
    --env=HOME="/home/" \
    --name="holmec/noetic-base-test" \
    --privileged=True \
    holmec/noetic-base
```
I have included a few additional commands to ease installations of libraries but most importantly are the ```--runtime=nvidia``` and ```--gpus all``` arguments. These faciliate access to the host operating system Nvidia driver and runtime. Save the ```run.sh``` file and execute as below. 
```console
holmech@linux-desktop:~$ ./run.sh
```

When inside the container, check the output of the Nvidia driver and CUDA with ```nvidia-smi``` command. The output should look like below. 

```
 +-----------------------------------------------------------------------------+
| NVIDIA-SMI 525.147.05   Driver Version: 525.147.05   CUDA Version: 12.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:01:00.0  On |                  Off |
|  0%   27C    P8    20W / 450W |    802MiB / 24564MiB |      6%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
```

## ROS Install

All we need to do now to fullful our remaining system requirements is update the Dockerfile and rebuild the image. Let's start with the ROS middleware. Below is my updated Dockerfile including the ROS installation. Once built, validation the image by running ```roscore``` inside the container. You may need to first source it ```source /opt/ros/noetic/setup.bash```. All being well you should now be able to run a instance of roscore inside the container.

Below is the full Dockerfile. 

```
FROM nvidia/cuda:11.1.1-devel-ubuntu20.04

USER root

# disable interactive configuration mode
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update \
    && \
    apt-get install -y \
    build-essential \
        ca-certificates \
        cmake-qt-gui \
        file \
        git \
        gnome-terminal \
        hicolor-icon-theme \
        language-pack-en \
        locales \
        nano \
        pulseaudio \
        software-properties-common \
        sudo \
        tar \
        unzip \
        vim \
        wget \
        xz-utils \
        net-tools \
        ifstat \
        chrony \
        curl \
        libncurses-dev \
        build-essential \
        cmake \
        git \
        pkg-config \
        python3-dev \
        linux-headers-generic \
        python3-pip \
    && \
    rm -rf /var/lib/apt/lists/* \
    && \
    update-locale LANG=en_US.UTF-8 LC_MESSAGES=POSIX

RUN echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list
RUN curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -

 RUN apt-get update \
     && \
     apt-get install -y \
        ros-noetic-desktop-full \
        ros-noetic-rosdoc-lite \
        python3-rosdep && \
    apt-get clean -y && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/*
```