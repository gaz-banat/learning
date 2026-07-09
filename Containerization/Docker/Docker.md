



DOCKER DAEMONS

Docker daemon 					-		The main docker daemon, e.g. pulls an image
containerd daemon				-		runs the container



NOTE: The Docker daemon is a communication platform that communicates reads and writes of stdin, stdout, and stderr from the initial process (PID1) created in the container.



DOCKER DAEMON SOCKET

/var/run/docker.sock




CONFIGURATION FILES

~/.docker/config.json		-	per user configuration file

/etc/docker/config.json		-	system wide configuration file



CONFIGURATION FOR DAEMON

/etc/docker/
	daemon.json
	key.json
	seccomp.json
	certs.d/



DATA DIRECTORY FOR DAEMON

/var/lib/docker by default



ENVIRONMENT VARIABLES

DOCKER_HOST		-	i think this identifies the socket where docker daemon listens



NETWORKING

none				-	Adds the container to a container-specific network stack with no connectivity.
host				-	Adds the container to the host machine’s network stack, with no isolation.
default bridge		-	The default networking mode. Each container can connect with one another by IP address.
custom bridge		-	User-defined bridge networks with additional flexibility, isolation, and convenience features.




STORAGE

Storage drivers
- overlay

  

VOLUMES



LOGGING


Logging Drivers
- journald




ENTRYPOINT VS COMMAND

ENTRYPOINT 	- 	the process that will run when the image starts
				/bin/sh -c if ENTRYPOINT is omitted from Image build
				can be overridden by docker run --entrypoint

CMD			-	arguments to the ENTRYPOINT process
			-	can be overridden by arguments to ‘docker run’ after specifying the image


e.g. docker run --rm --name engy --network web -e foo=bar --entrypoint /bin/sh nginx -c “echo hello”




SECCOMP PROFILE





REGISTRY AND REPOSITORY AND IMAGE AND TAG

To get an image use the following syntax
registry/repository/image:tag


Examples:
docker.io/library/kong:3.4			-	docker.io is the registry, library is the repository, kong is the image and 3.4 is the tag
registry.redhat.io/rhel8/python-36	-	registry.redhat.io is the registry, rhel8 is the repository and python-36 is the image, the tag is not specified so will become ‘latest’



Docker Hub:

Browser				-	https://hub.docker.com
Curl					-	https://hub.docker.com/
docker pull command		-	docker.io or index.docker.io



Docker official images
docker official images are available in the ‘library’ repository.
docker provides registry as ‘docker.io’ and repository as ‘library’ when they are not specified 
e.g. docker pull kong:2.0.0 is docker pull docker.io/library/kong:2.0.0




Questions:
1. How do i list tags for an image if i know the repository and image name
2. How do i list all images in a repository
3. Which is the correct format?
https://registry.hub.docker.com/v2/repositories/<repository>/<image>:<tag> 
OR 
https://registry.hub.docker.com/v2/repositories/<namespace>/<repository>:<tag>




PRIVILEGED CONTAINERS

docker run -ti --name hacker --privileged -v /:/host ubi8 chroot /host


--tty -(t)		—This allocates a pseudo -tty and attaches it to the standard input of the container.
--interactive (-i)	—This connects stdin to the primary process of the container

These options give you an interactive shell within the container.





COMMANDS

I want to get the PID of a running container
docker inspect --format ‘{{ .State.Pid }}’ <container-name-or-id>  

I want to know the first process in the container
cat /proc/1/cmdline

One cool way of starting a container and keep it running
docker run -d --rm ubi8:latest tail -f /dev/null

I want to enter the network namespace of a container 
PID=$(docker inspect --format ‘{{ .State.PId }}’  <container_id>)
nsenter -t ${PID} -n ip addr