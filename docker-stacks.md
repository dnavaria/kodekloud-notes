# Docker Stacks
- A container as we know is a packaged form of an application that has its own dependencies and runs in its own environment.
- A service is one or more instances of the same type of container that runs on a  single node or across multiple nodes in a swarm cluster.
- A stack is a group of interrelated services that together forms an entire application.

## Docker stack compose file example
```
version: 3
services:
   redis:
      image: redis
	  deploy:
	     replicas: 1
	db:
	   image: postgres:9.4
	   deploy:
	      replicas: 1
	vote:
	   image: voting-app
	   deploy:
	      replicas: 2
	result:
	   image: result
	   deploy:
	      replicas: 1
	worker:
	   image: worker
	   deploy:
	      replicas: 1
	
```

## Docker stack compose - place a service on a specific docker host
```
version: 3
services:
   redis:
      image: redis
	  deploy:
	     replicas: 1
	db:
	   image: postgres:9.4
	   deploy:
	      placement:
		     contraints:
			    - node.hostname == <hostname of docker host>
				- node.role == manager
	      replicas: 1
```

## Docker stack compose - limit resources
```
version: 3
services:
   redis:
      image: redis
	  deploy:
	     replicas: 1
		 resources:
		    limits:
			   cpus: 0.01
			   memory: 50M

```