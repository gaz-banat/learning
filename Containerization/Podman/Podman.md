
NEED TO UNDERSTAND

slirp4netns




PODMAN


Docker vs Podman:

API	-	Docker is a RESTful API daemon. Podman supports RESTful API via a systemd socket-activated service.

Python Packages		-		docker-py and podman-py  (are these python packages)

Unlike Docker, Podman includes such notable features as 
- user-namespace support,
- multiple transports, 
- customizable registries, 
- integration with systems, 
- the fork/exec model, 
- and out-of-the-box rootless mode.



Understanding Rootful and Rootless:

rootful										-	Running a container as root of the host (something like sudo podman run)

1. you are root, you run the first container process as root in the container		- 	the actual ID visible on the host for the container process is 0
e.g. # podman run -it ubi8:latest /bin/bash 

2. you are root, you run the first container process as non-root in the container	-	the actual ID visible on the host for the container process is the UID of the user running the process inside the container
e.g. # podman run -u sync -it ubi8:latest /bin/bash						so UID 26 inside the container (/etc/passwd of container) maps to UID 26 outside the container (which may or may not be in /etc/passwd of host)


rootless										-	Running a container with a standard unix account (non root) of the host

1. you are non-root, you run the first container process as root in the container	-	the actual ID visible on the host for the container process is your UID (meaning that the root user of the container maps to the UID of your unix account of the host machine)
e.g. $ podman run -it ubi8:latest /bin/bash 								so while the container process has root inside the container, outside the container on the host it only has what your privliges are
															this mapping is being accomplished by linux kernel feature user_namespace
															Each process after that in the container can map to a user within the user_namespace (100,000 range, this may be related to /etc/subuid and /etc/subgid)

2. you are non-root, you run the first container process as non-root in the container	-	the actual ID visible on the host for the container process is a non-root UID from /etc/subuid
e.g. $ podman run -u sync -it ubi8:latest /bin/bash						so the non-root user in the container is mapping to a non-root user in the host
															this mapping is being accomplished by linux kernel feature user_namespace
																								


NOTE: some containers are not capable of running under the increased restrictions of rootless. As an example, if a container creates new devices, loopback mount points, and performs other highly restricted operations, then they must be run as root


Show the mappings of user and group from container to host in the form
ID in container		ID on Host		Count of id’s

podman unshare cat /proc/self/uid_map
podman unshare cat /proc/self/gid_map



Transports:

Container storage locations are called ‘transports’. These are the 7 - 
Container registry (Docker)
oci
dir
docker-archive
oci-archive
docker-daemon
container-storage


Configuration:

Registries
~/.config/containers/registries.conf
/etc/containers/registries.conf			
/etc/containers/registries.conf.d/


Containers
  configuration
	/usr/share/containers/containers.conf
	/etc/containers/containers.conf
	/etc/containers/containers.conf.d/
	$HOME/.config/containers/containers.conf
   data
	~/.local/share/containers

Storage
~/.config/containers/storage.conf



Authentication file for podman login:

The auth file for podman login could be one of
  ${XDG_RUNTIME_DIR}/containers/auth.json		-		this will mostly resolve to /run/user/$UID/containers/auth.json - remember that /run is temporary filesystem destroyed on system reboot
  ${HOME}/.config/containers/auth.json
  /run/user/$(id -u)/containers/auth.json
  /run/containers/$(id -u)/auth.json
  ${HOME}/.docker/config.json

Which auth file was used?

podman login --verbose <registry> --get-login --log-level=debug 	 (look for ‘Found credentials for <registry>’)




subuid and subgid:

https://www.funtoo.org/LXD/What_are_subuids_and_subgids%3F#:~:text=%2Fetc%2Fsubuid%20and%20%2Fetc%2Fsubgid%20let%20you%20assign,username%3Astart%3Acount



Podman RESTful API Service:

The API service for podman can be activated by a socket.

Socket activation of the API service				
	defined in /usr/lib/systemd/user/podman.socket
	this is a unix socket
	start it with systemctl --user start podman.socket && ls $XDG_RUNTIME_DIR/podman/podman.sock

	Now you are ready to use the socket
	e.g. export DOCKER_HOST=${XDG_RUNTIME_DIR}/podman/podman.sock
	Now a program like docker-compose can come along and connect to the socket by looking at the value of DOCKER_HOST. Podman will respond to API requests via the socket.


Socket activation of containers



Quadlets:

Rootful unit search path
/etc/containers/systemd/
/usr/share/containers/systemd/

Rootless unit search path
$XDG_CONFIG_HOME/containers/systemd/ or ~/.config/containers/systemd/
/etc/containers/systemd/users/$(UID)
/etc/containers/systemd/users/




COMMANDS

Main
podman info
podman login

Container:
podman container prune


Images
podman [image] search httpd
podman pull
podman inspect
podman images
podman rmi 
podman tag
podman push
podman create
podman start
podman run
podman ps 
podman logs
podman cp
podman exec
podman commit <container_name> <image_name>
podman stop
podman kill
podman rm 


Images Advanced:
podman image tree 
podman image diff
podman image inspect
podman image mount
podman image umount


podman port -a			-		show all port mappings from host to containers

Podman system:
podman system connection
podman system connection default podman-machine-default-root	-	switch to a rootful connection to a remote system for running containers with root priv
podman system df
podman system service 	-	run the podman api servce



Network:
podman network create --gateway 172.17.1.1 --subnet 172.17.1.0/24 my_net
podman inspect my_net
podman run -d --name webapp -v /home/gbanat/webcontent:/var/www/html:Z -p 8082:8080 registry.access.redhat.com/ubi8/httpd-24



Podman macine:
podman machine list
podman machine start



Kuberentes:
podman generate kube
podman play kube




SOLUTIONS

For rootless containers I want to see the mapping of root user to user on the host
podman top -l user huser



QUADLET

1. Create a container unit file in
for root
	/usr/share/containers/systemd
	/etc/containers/systemd
rootless
	$HOME/.config/containers/systemd


2. Inform systemd about the new unit file
root
	systemctl daemon-reload
rootless
	systemctl --user daemon-reload

3. You now have a service file, get its status
systemctl [--user] status <name>.service

4. Enable and start
systemctl [--user] enable <name>.service
systemctl [--user] start <name>.service


The quadlet reads a container file and produces a service file
/usr/libexec/podman/quadlet
e.g. Do a dry run for the user systemd - /usr/libexec/podman/quadlet -dryrun -user


Quadlet supports the following file types
.container
.kube
.network
.volume





BUILDAH

specializes in building OCI images. 


Commands:

buildah unshare






SKOPEO

used for many of the tasks related to sharing and manipulating container images and image repositories 

