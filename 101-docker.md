# Docker 101

## Some other usefull commands
```bash 
#Stop all containers 
~]$ docker stop $(docker ps -a -q)

#Delete all containers
~]$ docker rm $(docker ps -a -q)
```

## Working with images
```bash 
# List al images
~]$ docker images

# Search image in a registry 
~]$ docker search nginx

# Pull an specific image from a registry
~]$ docker pull nginx

# Search with filter
~]$ docker search --filter=stars=20 nginx

# check image history
~]$ docker history nginx

# Get low-leve info from image
~]$ docker inspect nginx

# Delete image 
~]$ docker rmi nginx
~]$ docker rmi -f nginx # force
~]$ docker rmi --help   # show extra info from the previous command
```

## Run containers 
```bash 
# Run a container based on the nginx image, It wil assign a random name
~]$ docker run -dti nginx

# Run a container with an specific name
~]$ docker run -dti --name web2 httpd

# Run a one run container. it will autically remove the container after it exits
~]$ docker run -dti --rm --name web1 httpd

# Get low-lever info from a container
~]$ docker inspect web1
```

## List running containers
```bash
# List all containers
~]$ docker ps -a

# list last deployed container
~]$ docker ps -l
```

## Interact with your containers
```bash
# Start a container
~]$ docker start container-name

# Stop a container
~]$ docker stop container-name

# Remove a container
~]$ docker rm container-name
~]$ docker rm -f container-name # force

# Open a bash terminal in a specific container that is already running
~]$ docker exec -ti web1 /bin/bash

# Check running processes in a container
~]$ docker top <container_name>
~]$ docker container top <container_name>

# Puse/unpause container
~]$ docker pause container-name
~]$ docker unpause container-name

# Inspect docker element
~]$ docker inpect web1
``` 


## Show stats and hot change resources
```bash
# Display a live stream of container(s) resource usage statistics
~]$ docker stats

# Update container resources
~]$ docker update -m 200M --memory-swap -1 web5
```
