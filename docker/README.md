# Docker - Getting Started

### Docker Commands
The first docker command we will run return docker version and verifies it is working.
```sh
$ docker version
```

Next we see what see all commands we can run in docker
```
$ docker
```
It is a long list of commands, bus as part of our demo we will be using only a few of them.


Now, lets run a container. For this step lets first pull a docker image using below command.  
```
$ docker pull busybox
```
The pull command fetches the busybox image from the Docker registry and saves it to our system. To know more about [Busybox](https://en.wikipedia.org/wiki/BusyBox). To see the list of docker image on the system, run
```
$ docker image ls
```
Now, lets run a container based on this image, to do so we will run
```
# $ docker run busybox <some command>
$ docker run busybox ls
```
Just like we listed all the images we have on the system, we can also list all the running and exited containes. One thing to note is that everytime you run a container as soon as the process inside of it exits, the container is essentially dead. 
```
# To list all the running containers
$ docker container ls
# To list all the running and exited containers
$ docker container ls -a
```
Till now we have run a single command on the busybox container, what if you want to run multiple commands and intract with the container shell itself? Running the run command with the `-it` flags attaches us to an interactive tty in the container.
```
$ docker run -it busybox sh
```
Now we can do all sorts of cool stuff, and my favorite one is
```
# rm -rf /
# ls
# exit
```
Oh no, what have we done? Is everything okay? 
...Yes, everytime you request a container you get a NEW clean system. Remember containers are transient, they are meant to run the process and be done with it. So if I run below command again, I will get a new container:
```
$ docker run -it busybox sh
$ ls
```
This is where you get to see the immutable property of the `image`, whatever happens to the container, it does not affect the image.

If we want a container to stick around, we need it to run in daemon mode using `-d` arg. In this mode it will create the container and treat it as a long running process or a service. We also provide a name using `--name` for easy reference.
```
$ docker run --name busyforlife -it -d busybox
```

Now instead of using `docker run` command I can use `docker exec` command, that will run new commands inside the **existing** container.
```
$ docker exec -it busyforlife sh
# ls
# touch newfile.txt
# ls
# exit
```
Now, if you go `exec` into the container again, the `newfile.txt` will still be there. And if you want to save this change as a new image, then we can use `docker commit` command. In this case it would look like:
```
# docker commit busyforlife <your dockerhub id>/busybox:new
$ docker commit busyforlife containerdemo/busybox:new
```

And, now if I create a new container based of `busybox:new` image we will see always have the `newfile.txt` file.
```
$ docker run containerdemo/busybox:new ls
```
Similar to github commit and push, we can also push this new image to the dockerhub.
```
$ docker push containerdemo/busybox:new
```
Now, to stop a running container we will do
```
$ docker container stop busyforlife
```


### Creating Docker Image from scratch
To create a new image from scratch we will create a `Dockerfile`. One thing I would like to point is that docker images can be built incrementally, by that I mean that you can take other layers from other images and stack them together and build on top of them.
```
FROM ubuntu:18.04

RUN apt-get -y update

# update packages and install
RUN apt-get install -y openjdk-11-jre-headless wget curl unzip

RUN apt-get -y install git
RUN apt-get -y install maven

ENV JAVA_HOME /usr/lib/jvm/java-11-openjdk-amd64
```
To build the docker image we need to use `docker build` command and tag it as containerdemo/myjavaimage. 
**NOTE:** Don't forget the `.` at the end of the command, it means look for Dockrfile in the current directory.

```
$ docker image build -t containerdemo/myjavaimage .
```
To verify it worked, we can list it using
```
$ docker images
```
Now, finally lets run the `mvn` command to ensure proper environment setup.
```
$ docker run ontainerdemo/myjavaimage mvn --version
```


