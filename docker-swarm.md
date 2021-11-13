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
				- The `mnt` folder stores informaiton about the mount points
			
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

## Docker Managers
- A manager node, as we just saw is the master node where the swarm cluster is initiated.
- The manager node is responsible for maintaining the cluster state, managing the workers, adding and removing workers and creating, distributing and ensuring the state of containers and services across all workers.
- Having a single manager node is not recommended because if it fails, there will be no manager node to  manage the cluster.
- For fault tolerance you can have multiple manager nodes in a single cluster.
- However when you have multip
- le manager nodes, there arises a conflict of inerest.
- To prevent that, only a single manger node is allowed to make management decisions, that node is called the leader.
- The leader cannot always make a decision on its own.
- All decisions have to be mutually agreed upon by all the managers or the majority of the managers in the cluster.
- This is important because if the leader was to make as decision and then fail before informing the other managers, about the decision, the cluster would be in an inconsistent state.
- For example, if a new worker  was to be added to the cluster by the leader, without updating the other managers and the leader fails, the other managers would not be aware of the new worker and future cluster operations will ignore the new worker and service running on that worker resulting in an inconsistent application state.
- This in known as the problem of distributed consensus.

## How does docker ensures that all managers have the same information about the cluster at all times?
- Docker solves this by implementing the Raft consensus algorithm.
- The raft consensus algorithm decided who is going to be the leader among the three.
- If the leader has enough votes available to make as decision and that all decisions are in consent with the other manager.
- Raft algorithm uses random times for intiating requests.
- A random timer is kicked off on the three managers.
- The first one to finish the timer sends out a request to the other managers requesting permission to be the leader.
- The other managers on receiving the request, response with their vote and the node assumes the leader role.
- Now that it is elected, the leader sends out notification at regular intervals to other masters informing them that it is continuing to assume the role of the leader.
- In case the other nodes do not receive a notification from the leader at some point in time, which could either be due to leader going down or losing network connectivity.
- The nodes initiate a re-election process among themselves and a new leader is identified.
- Every manager has it own copy of raft database the stores information about the entire cluster.
- It is important that they are in sync.
- If the leader has to make a decision, for example adding a new worker node to cluster or creating a new service:
	- It has to notify the other two managers, get a response from at least one to meet quorum which we will discuss shortly.
	- Then commit changes to the databases on all master nodes.
- This enusres that any change in the environment is made with consent from the majority of the managers in the cluster.
- When we say with consent, it just means that the data is stored in multiple database in a consistent fashion.
- When deciding on the number of master ndoes, it is recommeded to select an odd number like `3, 5, 7`
- The reason for that is in case the network was to get segmented and the master nodes were to get places into two separate networks, each groups now has only three nodes.
- But since we originally had six manager nodes, the quorum for that cluster to stay alive was four.
- If you look at the groups after segmentation neither of the groups have four managers to meet the 	quorum, It results in a failed cluster.
- In case we had odd number of managers originally say seven then after the network segmentation, we have four on one segment network and three on the other.
- So our clusters still lives on the group with four manager nodes as it meets the quorum of four.
- There are better chances of your cluster staying alive 
- Ensure you always select odd nubmer of master nodes when you desing you environment.
- When all master node fails and we only have one manager node available.
- Then the only way to recover is to force create a new cluster .
- When we run the docker swarm init command with the `--force` new cluster option, a new cluster is created with the current node as the only manager and we get a healthy cluster with a single manager node. 
- Since this manager already has information about the service and tasks, the worker nodes are still part of the swarm and services continue to run.
-  We may later add more manager nodes or promote existing workers to become manager nodes.
-  To promote an existing worker to manager node run `docker node promote` command.

## Can manager nodes do tasks like worker nodes?
- Yes they can, when you create a service the instances are also spread across manager nodes.
- We can disable that and dedicate a node for management purpose alone.
- This can be done by command `docker node update --availability drain <Node>`.
- Docker recommends dedicating manager nodes for management tasks only in the production environments.

### Summary Raft
- Every decision has to be agreed upon by majority of the manager nodes.
- If a new worker is to be added to the cluster, it has to be agreed upon by the majority of the manager nodes, which in this case is two.
- Out of a total of three masters, if one node was to fail or was not responding at that moment and only two nodes were available, the decision to add the new node can still made with an agreement between the two available nodes.
- This is known as the quorum.
- Quorum is defined as the minimum number of members in an assembly that must be present at any of its meetings to make the preoceedings that meeting valid.
- If there were say a total of five managers, then the quorum would be three nodes. if there were seven managers the quorum would be four.
- Formula to calculate quorum of N = `ceil(( n + 1 ) / 2)`
- Docker recommends no more than 7 managers for the swarm.
- Adding more managers does not increase scalability or performance of the cluster.
- However there is no hard limit on the number of manager nodes you can add to a cluster, you can add as many nodes as you want and Docker will not stop you.
- Quorum is the minimum number of manager to keep the cluster alive.
- The reverse of that is the max number of nodes that can fail will give us the fault tolerence.

# Hands on notes
- `docker node ls` => prints all the nodes in the cluster
- `docker swarm leave` => make a node leave the cluster
- `docker node rm <node name>` => to remove the node from the cluster
- `docker swarm join-token manager` => to get a token, by using which we will be able to join cluster as manager
- `docker swarm join-token worker` => to get a token, by using which we will be able to join cluster as worker
- `docker node promote <node name>` => to promote a worker node to manager node 
- `docker swarm init --force-new-cluster --advertise-addr <ip address>`
-  
- The master node can be identified by leader under the manager status 
- The * after the node ID indicates the current system that you are on.

# Docker Service
- `docker service update --help` => to view the update command options
- They key componeny of swarm orchestration is docker service.
- Docker service are one or more instances of a single application of a single application or a service that runs across the swarm cluster
- `docker service create --replicas=<instance number> <image name>` => command to deploy docker container in docker swarm
- Remember the docker service command must be run on the manager node not on the worker node
- The docker service create command is similar to the docker run command in terms of the options passed such as the `-e environment variables`, `-p for publishing ports or port mapping` and the --network to attach the container to a particular network.

## Working
- When you run docker service create command to create `n` instances of an image, the orchestrator on the manager node decides how many tasks to create, and then the scheduler should use that many tasks on each worker node.
- The task on the worker node is a process that kicks off the actual container instances.
- The task has a one to one relationship with each container.
- The task is responsible for updating the status of the container to the manager node, so the manager node can keep track of the workers and instances running on them.
- In case the container was to fall, the task fails as well.
- If a task fails, the manager node becomes aware of it and it automatically reschedules a new task to run a new container instance to replace the failed container.

## Replicas vs Global
- There are two types of services replicated and global:
	- Replicas
		- The services which are created with the `--replica` option with a predefined number of replicas
	- Global
		- `--mode global` instead of `--replica` 
		- When you want one instance of a service placed on alll nodes in the cluster.
		- A good example would be a monitoring agent

## Container naming in docker swarm
- If we don't specify a name while creating a container or a service, docker automatically generates a random funny name but you could use the name option to specify the name option to specify the name.
- When we use this `docker run` command it created a service with the same name as we only had a single container.
- But now that we have multiple instances of the container running, docker swarm automatically appends a number to the name on each container to avoid having the same name for all containers.

## Update docker service
- `docker service update --replicas=<new instance number> <image name>` => to update docker service
- `docker node update --availability drain <swarm manager node name>`