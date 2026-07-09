


CONTAINER ORCHESTRATORS

Things that run containers on several nodes by talking to a container engine on a node

Kubernetes
Apache Mesos
Docker Swarm



CONTAINER HIGH LEVEL RUNTIMES (container engines)

Thing that runs containers on a single node and provide for container isolation, image management and container lifecycle management

Docker Engine					-	image build, volume management, networking, image distribution, uses containerd
Podman					
Buildah						-	this container engine is used to build containers
Mirantis Container Runtime


CONTAINER MID LEVEL RUNTIMES

Things that are front ends to manage containers - responsible to create, destroy, start and stop containers

CRI-O						-	specifically for kubernetes
containerd						-	provides an api over a low lever container runtime


CONTAINER LOW LEVEL RUNTIMES

Things that configure different parts of the Linux kernel and then run a container  (NOTE - OCI is a good standard) - container execution

rkt
runc							-	OCI compliant, go programming language
crun							-	OCI compliant, C programmling language
runv							-	OCI compliant
runhcs
kata
gVisor
LXC
LXCD


TERMS

namespace

cgroups
cgroups are resource constraints
The basic idea of cgroups is preventing one group of processes from dominating certain system resources in such a way that another group of processes can’t make progress on the system.


seccomp
seccomp is a Linux kernel mechanism that limits the number of syscalls to a group of processes on the system. 
You can remove potentially dangerous syscalls from being called by the proceses





IMAGE

an image is ‘a diectory tree + a json file that describes the rootfs + a json file called manifest that links multiple images together to support different architectures’

Image Formats:

OCI				-		The manifest and layer tarballs are located in the local directory as individual files.
OCI Tar (archive)

Docker (dir)			-		a container image compliant with Docker image layout. Very similar to OCI format but it is non standardized
docker-daemon		-		an image stored in the Docker daemon’s internal storage
Docker Tar (archive)		-		a container image in Docker image layout, which is packed into a tar archive
		


OCI container runtime specification JSON file

