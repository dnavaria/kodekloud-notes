# Docker Networks
- There are three types of networks available in Docker:
	- bridge
		- Default network container gets attached to.
		- It is a private inter nal network created by docker on the host.
		- All containers attach to htis network by default and they get an internal IP address, usually in the range 172.17 series
		- The containers can access each other using this internal IP if required.
		- To access any of these containers from the outside world, map ports of these containers to ports on the docker host as we have seen before.
	- null
		- The containers are not attached to any network and doesn't have any access to the external network or other containers.
	- host
		- This takes out any netwok isolation between the Docker ho st and the Docker containers
		- Meaning if you were to run a web server on port 5000 in a web app container attach to host network, it is automatically accessible on the same port externally with out requiring to publish the port using the -p option as the web container uses the host network.
		- This would also mean that unlike before, you will now not be able to run multiple web containers on the same host on the same port as the ports are now common to all containers in the host network.

## Bridge Network
- Say we have multiple docker hosts running containers, each docker host has its own internal private bridge network in the 172.17 series allowing the containers running on each host to communicate with each other.
- However, containers across the host has no way of communicating with each other unless you publish the ports on those containers and setup some kind of routing yourself.
- This is where overlay network comes into play.
- With Docker swarm you could create a new network of type overlay, which will create an internal private network, that spans across all the nodes participarting in the swarm cluster.
- We could then attach the containers or services to this network using the network option while creating a service.
- So we can get them to communicate with each other thorugh the overlay network.

```
docker network create --driver overlay --subnet 10.0.9.0/24 my-overlay-network
docker service create --replicas 2 --network my-overlay-network nginx
```

## Ingress Network
- Say we have web service running on port 5000, for an external user to access to web service , we must map the port to a port on the Docker host.
- In this case map port 5000 on the container to port 80 on the docker host.
- Once we do that, a user will be able to access the  web server using the URL with port 80.
- This work just fine when running a single container.
- When you create a docker swarm it automatically creates an ingress network
- THe ingress network has built in load balancer that redirects traffic from the published port to all the mapped ports.
- Since the ingress network is created automatically, there is no configuration that you have to do.
- You simply have to create the service you need by running the service create command
- Ingress network is infact is a type of overlay network, meaning its a single network that spans across all the nodes in the cluster
- The way the load balancer works is it receives requests from any node in the cluster and forwards that requests to the respective instances on any other nodes, essentially creating a routing mesh.
- The routing mesh helps in routing user traffic, that is received on a node that is not even running and instance of webservice to other nodes where the instace is running.
- The is the default behaviour of docker swarm and we don't need to do any other configuration.

## Embedded DNS
- All container in docker host can resolve each other using container name.
- Docker has a built in DNS server that helps the container to resolve each other, using the container name.
- The built in DNS server always runs at 127.0.0.11