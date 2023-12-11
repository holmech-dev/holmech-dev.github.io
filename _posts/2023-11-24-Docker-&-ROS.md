---
layout: post
---
This post outlines how to create a Docker environment with ROS middleware setup.

Docker offers a great amount of portability when it comes to preparing software environments, this lends well when developing applications with middleware such as ROS (Robotics Operating System) where managing a large amount of software libraries and portability maybe necessary. 

To create the Docker environment a Docker image is needed, this will serve as our template or environment snapshot for creating our Docker container. A container is the instantiation of this image that we can work in. To build the image, we first need a way to instruct the build process what should be contained in the image, this is done with a Dockerfile. 

![Docker Image](/docker_image_container.png)

I use knowledge about my system requirements to create the Dockerfile. For this build, I have the following requirements. 
<ul>
  <li>Ubuntu 20 - Operating system compatibility</li>
  <li>Noetic - ROS Version</li>
  <li>OpenCV 4.4.0 - Library Version</li>
  <li>CUDA - Nvidia GPU utilisation</li>
</ul>

First, let's create a Dockerfile. Create this file in a suitable directory necessary to store the docker files for you project. 
```console
holmech@linux-desktop:~$ touch Dockerfile.base
```

## Updating the Dockerfile
In a text editor if your choosing, the Dockerfile can now be edited. I'm using VS Code.
Using the ```FROM``` command, a base image is specified as a starting point. Then to assist with the build process, the ```USER``` is specified. The last argument in the file facilitates install of some libraries without necessity to interact with the user during the process. More information on this can be found below. 

<https://stackoverflow.com/questions/63476497/docker-build-with-ubuntu-18-04-image-hangs-after-prompting-for-country-of-origin>

Then, let's try and run a system update to validate we can successfully install packages. 

### Dockerfile
```
FROM nvidia/cuda:11.1.1-devel-ubuntu20.04

USER root

# disable interactive configuration mode
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update \
    && \
    apt-get install -y \
    && \
    rm -rf /var/lib/apt/lists/* \
    && \
    update-locale LANG=en_US.UTF-8 LC_MESSAGES=POSIX
```

Save the file. We then need to try and build the image for the first time. Using the ```build``` command, the ```-t``` argument is used to tag the image. This tag takes the username first followed by the image name. The ```-f``` argument is used to use the point the build process to the file we want that includes the build instructions, in this case the ```Dockerfile.base``` file just created.

```console
holmech@linux-desktop:~$ docker build -t holmec/noetic-base -f Dockerfile.base .
```

The command may take some time to complete. A successful build should have the following terminal output.

