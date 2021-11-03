# Docker Swarm
## Storage and Filesystem
- docker main directory is `/var/lib/docker`
	- This directory consists of the following subdirectory:
		- `aufs` - The aufs stores data in three different directories:
			- `diff`
				- This is the folder where the actual contents of each layer is stored
				- A new layer is created for each instruction in the docker file
				- A layer is just another folder in this case and it mapped by the storage driver
			- `layers`
				- This folder stores metadata about how image layers are stacked
			- `mnt`
				- The `mnt` filder stores informaiton about the mount points
			
		- `builder`
		- `containers`
		- `image`
		- `network`
		- `plugins`
		- `swarm`
		- `tmp`
		- `trust`
		- `volumes`
- To find the storage driver being used  run `docker info`
	- Default storage driver is `aufs`
- Each storage driver stores data differently
- `docker system df -v` => to view the actual disk space used by the images and containers.