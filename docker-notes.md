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
- `docker network ls` => to list all networks
- `docker network create --driver <host/bridge/none> --subnet <subnet/mask> <network name> ` => create a custom network
- `docker system df -v` => to view the actual disk space used by the images and containers.

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
	- Now if we were to create a container which is basically like a child sytem within the current system, the chlid system needs to think that it is an independent system on its own and it has its own set of processes originating from the root process with a `process id of 1`.
	- But we know there is no hard isolation between the underlying container and the host, so the process running inside the container are in fact processes running on the underlying host and two process cannot have `process id 1`.
	- This is where namespaces comes into play.
	- With process id namespaces each process can have multiple process ids associated with it.
	- For example when the process start in the container its actually just another set of processes on the base linux system and it gets the next available process id, however they also get another process id starting with `pid 1`in the container namespace which is only visible inside the container so the container thinks that it has its own root process tree and so it is an independent system.
	- Let's say we ran an nginx server as a container, we know that the nginx container runs a nginx service.
	- If we list all services inside the docker container we see that the nginx service running with a `process id 1` , if we list the services on the docker host we will see the same service but with a different process ID.
	- That indicates that all process are infact running on the same host but separated into their own containers using namespace.
	
	## How much of the resources are dedicated to the host and the containers and how does docker manage and share the resources between the containers by default?
	- Docker host and containers shares the same system resources such as CPU and memory.
	- By default there is no restriction as to how much of a resouce a container can use and hence a container may end up utilizing all of the resources on the underlying host.
	- Docker uses cgroups or control groups to restrict the amount of hardware resources allocated to each container.
	- But there is a way to restrict the amount of CPU or memory a container can use Docker uses three groups or control groups to restrict the amount of hardware resources allocated to each container.
	- This can be done by providing the `--cpu` option to the docker command, providing a value of `.5` will ensure that the container does not take up more than 50 percent of the host at any given time.
	- The same goes with memory `--memory` limits the amount of memory the container can utilize.
# Docker Storage [[docker-storage]]
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
## Storage Drivers [[docker-storage-drivers]]
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

# Docker Networking
- When we install docker it creates three networks automatically :
	- **bridge**
		- It is the default network a container gets attached to.
		- If you would like to associate the container with any other network you specify  the network information using the network command line paramenter `docker run ubuntu`
		- This network is a private intrernal network created by docker on the host.
		- All containers attached to this network by default and they get an internal IP address usually in the range `172.17` series.
		- The containers can access each other using this internal ip if required.
		- To access any of these containers from the outside world map the ports of these containersto ports on the docker host.
	- **none**
		- `docker run ubuntu --network=none`
		- The containers are not attached to any network and doesn't have any access to the external network or other containers.
		- They run in an isolated network.
	- **host**
		- `docker run ubuntu --network=host`
		- Another way to access the container externally is to associate the container to the host network.
		- This takes out any network isolation between the docker hsot and the docker container.
		- Meaning if we are going to run a web server on port 5000 in a web container it is automatically accessible on the same port externally without requiring any port mapping as the web container uses the hosts network.
		- This would also mean that unline betfore you will now not be able to run multiple web containers on the same host on the same port as the ports are  now common to all containers in the host network.
- By default docker only creates one internal bridge network.
- We could create our own internal network using the command:

```
docker network create \
	--driver bridge \
	--subnet 182.18.0.0/16
	custom-isolated-network
``` 
- Running `docker inspect <container name>` will show you metadata and in `NetworkSettings` section, we will see the type of driver used adn the ip and mac address.
- Containers can reach each other using their names.
- Docker has a built in DNS server that helps the containers to resolve each otherg the containers name.
- Not that the built in DNS server always runs at address `127.0.0.11`.
## How are the containers isolated within the host?
- Docker uses network namespaces that creates a separate namespace for each container.
- It then uses virtual ethernet pairs to connect containers together.

# Docker Swarm [[docker-swarm]]
- With Docker swarm we can now combine multiple docker machines toghether into a single cluster
- Docker swarm will take care of distributing our services or our application instances into separate hosts for HA and for load balancing across different systems and hardware to setup a docker swarm.
- To setup docker swarm we must have hosts or multiple hosts with docker installed on them.
- Then we have to designate one host to be the swarm manager and others as workers.
- Once we are done with that we will run `docker swarm init --advertise-addr <ip>`  command on the swarm manager 
- This will initialize the swarm manager and the output will also provide the command to be run on the workers 
- Join the worker nodes using 
	- `docker swarm join --token <token output is given when you initialized the swarm manager > <swarm manager ip>`

# Kubernetes
- With docker we were able to run a single instance of an application using the docker cli by running the docker run command.
- With kubernetes using the kubernetes cli known as kubectl, we can run a thousand instance of the same application with a single command.
- Kubernetes can scale it up to two thousand with another command.
- Kubernetes can be even configured to do this automatically so that instances and the infrastructure itself can scale up and down based on user load.
- Kubernetes can upgrade these two thousand instances of the application in a rolling fashion one at a time with a single command.
- If something goes wrong it can help you roll back these images with a single command.
- Kubernetes can help you test new features of your application by only upgrading a percentage of these instances through ab testing methods.
- The kubernetes open architecture provides support for many different network and storage renders
- Any network or storage brand that you can think of has a plugin for kubernetes.
- Kubernetes provide a variety of authentication and authorization mechanisms all major cloud service providers have native support for kubernetes.

## Kubernetes Architecture
- It consists of a set of nodes generally a physical or virtual machine on which kubernetes is installed.
- Node is a worker machine where the docker containers can be launched by kubernetes.
- A cluster is a set of nodes grouped together this way even if one node fails, we have our application still accessible from the other nodes.
- The master is a node with the kubernetes control plane components installed.
- The master watches over the nodes in the cluster and is responsible for the actual orchestration of containers on the worker nodes.
- When you install kubernetes on a system, you are actually installing the following components:
	- **An API server**
		- Acts as the front end for kubernetes 
		- The users, management devices, cli interfaces all talk to api server to interact with component cluster
	- **Etcd server**
		- It is a distributed, reliable key-value store used by kubernetes to store all data used to manage the cluster
		- When you have multiple nodes and multiple masters in your clusters etcd stores all that information in the cluster in a distributed manner 
		- It is responsible for implementing logs within the cluster to ensure there are no conflicts between the masters
	- **kubelet**
		- It is the agent that runs on each node in the cluster.
		- The agent is responsible for making sure that the containers are running on the nodes as expected.
	- **Container runtime**
		- The container runtime is the underlying software that is used to run containers in our case it happens to be Docker
	- **Controller**
		- The controllers are the brain behind the orchestration
		- They are responsible for noticing and responding when nodes, containers or endpoints close down.
		- The controllers makes decisions to bring up new containers in such cases
    - **Scheduler**
		- It is responsible for distributing work or containers across multiple nodes
		- It looks for newly created containers and assigns them to nodes
- **kubectl**
	- It is the kubernetes cli which is used to deploy and manage applications on a kubernetes cluster to get cluster related information, status of the nodes in the cluster and other things.
	- The `kubectl run hello-minikube`  command is used to deploy an application on the cluster
	- The `kubectl cluster-info`  is used to view information about the cluster and
	-  The `kubectl get nodes` command is used to list all the nodes part of the cluster.