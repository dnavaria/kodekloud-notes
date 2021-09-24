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
- `docker run -v <host_directory:container_directory> <image_name>`