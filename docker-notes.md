# Introduction
- Docker utilizes LXC containers.
- We are not running linux container on windows. Windows run a linux container on a linux virtual machine under the hood.
- A container is lives as long as the process inside it is alive.
# Basic Docker Commands
- `docker run <image_name>` => Runs the image with the name provided. If the image is not provided it then pull the image and then run that image. If the container is stopped and then started again then the same image is reused.
- `docker pull <image_name>:tagid` => Pull the image from the docker hub or private repository if specified with latest tag if no tag is specified or pulls the image with the given tag id.
- `docker ps` => list the metadata of the running containers
- `docker ps --all/-a` => lists all the containers both stopped and running
- `docker stop <container_name/container_id>` => stops the container
- `docker rm <container_name>` => removes the container permanently
- `docker images` => lists all the images present on the system
- `docker rmi <image_name>` => removes the image from the system and one should delete all dependent conainers to remove image.
- `docker exec <container_name> <command>` => executes a command inside a container
- `docker run -d <image_name/conainer_name>` => runs a container in deattached mode
- `docker attach <container_id>` => attach the container back to the console
- `docker run -it <image_name/container_name> bash` => start container bash console
- `docker run -it <image_name>` => run container in `i` interactive mode and `t` for terminal mode
- `docker run -p <host_port:container_port> <image_name>` => mapping ports 
- `docker run -v <host_directory:container_directory> <image_name>` => mapping directories
- `docker history <image_name>` => show the history of the container build
- `docker build .` => used to build custom images using the command defined in the dockerfile
- `dockere push <image_name>` => used to push images to docker hub or private repository
- `docker run -e <env variable name>=<value> <image_name/container_name>` => set env variables in docker container
- `docker inspect <container_name:container_id>` => check the env varible and other metadata
- `docker run --entrypoint <entrypoint which is usually an executable>  <image_name> <parameter>` => setting entry point in docker run command
# Docker Images 
```
FROM Ubuntu
RUN apt-get update && apt-get -y install python
RUN pip install flask flask-mysql
COPY . /opt/source-code
ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```
- When Docker builds the images it builds these in a layered architecture.
- Each line of instruction creates a new layer in the docker images with just the changes from the previous layer.
- Since each layer only stores the changes from the pervious layer it is reflected in the size as well.
- All the layers build are cached so the layered architecture helps you restart docker build form that particular step in case it fails or if you were to add new steps in the build process you wouldn't have to start all over again.
- In dockerfile you can specify commands by using `CMD [<commands>,<paramenters>]` or if you want to give a default parameter in dockerfile use `ENTRYPOINT [<command>]` and specify the parameter in the `CMD` list.
# Docker Compose
## Docker run
- Linking different containers using `--link`.
- Link is a command line option which can be used to llink two containers together.
- `docker run -d --name=vote -p 5000:80 --link redis:redis voting-app`
- By defining a container with a name we can use that name to link that container with other containers.
	- `--link` command creates an entry into the `/etc/hosts` directory with the hostname provided after `--link` command with an internal IP of the container.
- Note that using links this way is deprecated and the support may be removed in future in docker.
- This is because as we will see in some time advanced and newer concpets in docker swarm and network supports better ways of achieving what we just did here with links.
- In case if you want to build the docker image using docker compose file replace `image: <image_name>` with `build: <path to directory which contains docker file and code>` 
## Docker Compose Versions
- We might see docker compose files in different formats at diferent places.
- Docker compose evolved over time and supports a lot more options that it did in the beginning.
- The original version of Docker compose file known as `version 1`.
- This had a number of limitations for example if you wanted to deploy conatiners on a different network other that the default bridged network, there was not way specifying that in this portion of the file.
- Also say you have a dependency or start or startup order of some kind for example your database container must come up first and only then should the voting application be started, there was no way you could specify that in the version of the docker compose file.
- Support for these came in `version 2`, with `version 2` and up the format of the file also changed a little bit.
- You no longer specify your stack information directly as you did before.
- It is all encapsulated in the services section so create a property called `services` in the root of the file and then move all the services underneath that you will still use the same docker-compose up command to bring up your application stack but how does docker-compose know what version of the file you are using.
- We are free to use the` v1, v2 or v3` depending on our needs.
- For version 2 and up you must specify the version of docker-compose file you're intending to use by specifying the version at the top of the file.
- Another difference is the networking, all the containers it runs using `v1` to the default bridged network and then use links to enable communication.
- With `version 2 `docker compose automatically creates a dedicated bridged network for this application and then attaches all containers to that new network.
- All containers are then able to communicated to each other using each others service name.
- We basically do not need to use links in `version 2` of docker compose.
- You can simply get rid of all the links you mentioned in `version 1` and convert the file to `version 2.`
- Finally `version 2` also introduces a depends on feature if you wish to specify a startup order, for this we can add it depends on property to the voting application and indicate that it is dependent on some other container.
- `Version 3` is similar to `Version 2`, in the structure meaning it has a version specification at the top and a services section under which you put all your services just like in `version 2` and make sure to specify the version number as 3 at the top.
- `Version 3 `comes with support for `docker swarm`.
- There are some options that were removed and added to see details on those you could refer to.
- To create a new property called networks at the roots level adjacent to the services in the docker compose file and add a map of networks we are planning to use.
- Then under each service create a networks property and provide a list of networks that service must be attached to.
```
version: 2
services:
     redis:
	      image: redis
		  networks:
		      - back-end
	  db:
	      image: postgres:9.4
		  networks:
		      - back-end
	  vote:
	      image: voting-app
		  networks:
		      - front-end
			  - back-end
	   result:
	       image: result
		   networks:
		       - front-end
			   - back-end
networks:
     front-end:
	 back-end:
		   
```

```
version: "3"
services:
	redis:
		image: redis
	db:
		image:  postgres: 9.4
		environment:
			POSTGRES_USER: postgres
			POSTGRES_PASSWORD: postgres
	vote:
		image: voting-app
		ports:
			- 5000:80
	worker:
		image: worker-app
	result:
		image: result-app
		ports:
			- 5001:80
```
# Docker Engine
- Docker Engine is simply referred to a host with Docker installed on it.
- When you are installing docker you are actually installing three different components.
	- `Docker daemon`
		- This is a background process the manages Docker objects such as the images, containers, volumes and networks.
	- The `REST API Server` 
		- This server is the API interface that programs can use to talk to the demon and provide instructions.
		- We can create our own tools using this API.
	- The `Docker CLI`
		- This is that command line interface that we generally use.
		- It uses the rest api to interact with the docker demon.
		- Docker CLI need not be on the same host. It can be on another system like a laptop and can still work with a remote Docker Engine.
			- Simply use the `-H` option on the dokcer command and specify the remote Docker engine address and the port `docker -H=remote-docker-engine:port`
## How exactly applications are containerized in Docker?
- Docker uses namespace to isolate workspace, process ids, networks, interprocess communication, mounts and Unix time sharing systems are created in their own namespace thereby providing isolation between containers.
- `Namespace - PID`
	- Whenever a linux system boots up tit start with just one process with a process id of `1`.
	- This is the root process and kicks off all the other process in the system by the time the system.
	- By the time the system boots up completely we have a handful of processes running this can be seen by running the PS command to list all the running processes.
	- The `process id` are uniqure and two processes cannot have the same `process id`.
	- Now if we weere to create a container which is basically like a child sytem within the current system, the chlid system needs to think that it is an independent system on its own and it has its own set of processes originating from the root process with a `process id of 1`.
	- But we know there is no hard isolation between the underlying container and the host, so the process running inside the container are in fact processes running on the underlying host and two process cannot have `process id 1`.
	- This is where namespaces comes into play.
	- With process id namespaces each process can have multiple process ids associated with it.
	- For example when the process start in the container its actually just another set of processes on the base linux system and it gets the next available process id, however they also get another process id starting with `pid 1`in the container namespace which is only visible inside the container so the container thinks that it has its own root process tree and so it is an independent system.
	- Let's say we ran an nginx server as a container, we know that the nginx container runs a nginx service.
	- If we list all services inside the docker container we see that the nginx service running with a `process id 1` , if we list the services on the docker host we will see the same service but with a different process ID.
	- That indicates that all process are infact running on the same host but separated into their own containers using namespace.
	
	## How much of the resources are dedicated to the host and the containers and how does docker manage and share the resources between the containers by default?
	- Docker host and containers shares the same system resources such as CPU and memory.
	- By default there is no restriction as to how much of a resouce a container can use and hence a container may end up utilizin all of the resources on the underlying host.
	- But there is a way to restrict the amount of CPU or memory a container can use Docker uses three groups or control groups to restrict the amount of hardware resources allocated to each container.
	- This can be done by providing the `--cpu` option to the docker command, providing a value of `.5` will ensure that the container does not take up more than 50 percent of the host at any given time.
	- The same goes with memory `--memory` limits the amount of memory the container can utilize.
# Docker Storage
## How docker stores data?
- It creates this folder structure at `/var/lib/docker` 
- we have multiple folders under it
	- `aufs`
	- `containers`
		- All files related to container are stored under the container directory
	- `image`
		- All files related to images are stored under the container directory
	- `volumes`
## Layered architecture
- When docker builds images it builds these in a layered architecture.
- Each line of instruction in the docker file creates a new layer in the docker image with just the changes from the previous layer.
- The layered architechture allows docker to reuse those layers and build new images faster.
- `COPY-ON-WRITE`
	- The image read being read only just means that the files in these layers will not be modified in the image itself so the image will remain the same all the time until you rebuild the image using the docker build command.
## What happens when we get rid of the container?
- All of the data that was stored in the container layer also gets deleted.
## Persist Data
- For example we are working with a database and we would like to preserve the data stored by database.
	- We could add a persistent volume to the container 
	- `docker volume <volume name>` => creating a volume
		- This command creates a folder with volumne name specified in the command under the `/var/lib/docker` directory
		- Then when we run the docker container using the docker run command we could mount this volume inside the docker container `read write layer` using the `-v <volume name or path to a directory on host>:<path to directory in container>` option
	- There are two types of mount:
		- volume mount
			- it mounts the volume from the volume directory
		- bind mount
			- It mounts a directory from any location on the docker host
	- `-v` is old style, we should use `--mount` as a prefered way as it is more verbose, so we have to specify each parameter in a key equals value format
		- `docker run --mount type=bind,source=/<path>,target=/<path> <container name>`
## Storage Drivers
- Docker uses storage drivers to enable layered architecture
	- Maintaining the layered architechture
	- Creating a writable layer 
	- Moving files across layers to enable copy and write
- Common storage drivers:
	- `AUFS`
	- `ZFS`
	- `BTRFS`
	- `Device Mapper`
	- `Overlay`
	- `Overlay2`
- The selection of the storage drivers depends on the underlying OS being used.
- The default storage driver is `AUFS` whereas thsi storage drivers is not available on other operating ssytems like fedora or cent OS.
- In that case device mapper may be a better option
- Docker will choose the best storage driver available based on the operating system.