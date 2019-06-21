Docker is a container manager that can create and execute containers which contain specific runtime environments for your software. 

It uses the resource isolation features of the Linux Kernel (cgroups - control groups and namespaces) to allow containers to run independently within the same Linux instance.

Docker has three important concepts : docker machines, docker images and docker containers.

* Docker machines are machines where docker is running.
* Docker images are result of execution of scripts containing a set of commands in DockerFile.
* Docker containers are executions of an specific runtime environment i.e. Docker images. 

Below are some of the frequently used commands :

See the list of images :
docker images

Fetch an image :
docker pull {imagename}
e.g. docker pull mongo

Create a container :
docker run {imagename}  --- if the image is not available locally, it will pull it first.
e.g. docker run mongo

See the list of containers :
docker ps -a

Stop a container :
docker stop #containerId

Remove a container :
docker rm #containerId

Remove a image :
docker rmi #imageId

Check docker version :
docker -v

Check if docker service is running :
service docker status

While starting a container, better give name to the container :
docker run -it ubuntu --name ubuntu_latest

Use -d to run docker container process in the background.

To connect to the container shell, use the following command :
docker exec -it tomcat_lab /bin/bash

For port forwarding :
docker run --name tomcat_lab -p 8080:8080 -d tomcat

Use rm to automatically clean up the container : 
docker run --name tomcat_lab --rm -p 8080:8080 tomcat

==========================================================================================================================================

Volumes

Container is a self-contained entity. All components including filesystem, network, processes, etc. are enclosed within the container.
Volumes enable container to read and write data to file systems that are OUTSIDE the container lifecycle. And the contents of the volume DO NOT increase the size of the container.

There are two ways to integrate volumes to a docker container :

1. Bind mounts : the file/directory on the host machine is hosted into the container
docker run --name tomcat_lab -p 8080:8080 --mount type=bind,source=/tmp/docker/webapps/sample,target=/usr/local/tomcat/webapps/sample --rm -d tomcat

2. Named Volumes : Docker manages the entire lifecycle of the volume. Docker recommends this over bind mounts.

    List of shared volumes : docker volume ls

    Create a volume : docker volume create web-vol

    Get info about the volume : docker volume inspect web-vol

    Remove a volume : docker volume rm web-vol

docker run --name tomcat_lab --rm -p 8080:8080 --mount source=web-vol,target=/usr/local/tomcat/webapps/sample  -d tomcat

=============================================================================================================================================

Networks

You can configure docker containers to talk to each other or to talk to the external world by configuring Docker networking.

When you start Docker daemon, some default networks are already created.

To list them, use : docker network ls

Inspect details of a specific network using : docker network inspect bridge

To create a user-defined network : docker network create --driver bridge ubuntu-bridge

Now run two containers with the following command :

Run two containers with the following commands : 

docker run --name ubuntunet3 --network ubuntu-bridge --rm -d -it ubuntu:latest
docker run --name ubuntunet4 --network ubuntu-bridge --rm -d -it ubuntu:latest

================================================================================================================================================

Dockerfiles

What is a Docker image?
An image is an executable binary package that contains everything needed to run an application. An image typically includes application code, base OS, software libraries and configuration files. 

A container is an execution of the image. 

DockerFile is the all-important configuration file that you create and define what goes to that image. You would then use "docker build" command to build the image.

Lets say you want an image with :
1. ubuntu 16.04
2. nginx web server

FROM ubuntu:16.04
RUN apt-get update && \
 apt-get install -y curl \
 nginx
EXPOSE 80
CMD ["/usr/sbin/nginx"]

The FROM instruction sets the base image on which your image is going to be built. Docker uses a layered approach to building image. Each instruction in a DockerFile gets applied to the image that resulted from the previous stage. The FROM instruction must be the first instruction in a docker file.

The RUN instruction executes the command specified on top of the current image and commit the results, creating an updated image. You can have multiple RUN instructions to apply layers of updates to the image being built.

COPY command copies files from host to container. They can be artifacts, config files, etc. COPY <src> <dest> is the command.

CMD is used to specify what needs to be run. There can be only one CMD per DockerFile.

Simply run the command :
docker build -t ubuntu_lab:1.0 <name_of_directory_with_DockerFile>  
-t helps in giving tags. if you dont use it, an image without a tag and only an image id will be created.

A docker image will be created. Check using 'docker images'

Run the image using docker run --name ubuntu_lab -d --rm ubuntu_lab:1.0
This will run and exit the container since nginx goes to the background and containers require an active process.
So we need to tell nginx to run in the foreground.
For that use -y daemon off i.e. modify the docker file's CMD line to CMD ["/usr/sbin/nginx", "-y", "daemon off;"]

Build the image and run using the same steps. 
Check http://localhost/dockerwins.html and it should show you the contents of the copied html.
