
---
# BOOTING AND INSTALLING AND CONFIGURING AND UPGRADING

You have to boot the node in some way
Then you have to install the Talos OS onto the node with a configuration
You also have to someday upgrade the Talos OS to a newer version 


Think about a Talos OS node in 2 parts
1. What makes the Talos OS                      - kernel + initramfs + binaries + libraries (this applies to an update to the OS)
2. What makes the Talos configuration           - v1alpha1 + other config documents (this applies to apply-config)


# PRE-REQ - BOOT ASSETS

## First understand what is a boot asset

a boot asset could be 
- an image (iso file, a raw disk image, a qcow2 disk image, a container image)
- a kernel + initramfs
- an iPXE script
More at https://www.talos.dev/v1.10/talos-guides/install/boot-assets/


### about images
An image can be used for:
booting                         -       you could be booting a metal system, a vm in a cloud provider, a vm on a hypervisor, a vm on a local hypervisor
or 
installing                      -       specified by .machine.install.image in v1alpha1 configuration (generally an installer image) after Talos has booted
or
upgrading                       -       specified by the talosctl upgrade command to a running Talos system


An image can be got from:
installer image                 -       the base installer image available from ghcr.io/siderolabs/installer:<version> 
                                        look at the release page - Images, this is an OCI container image
OR
image factory service           -       https://factory.talos.dev                   
                                        only for official Talos Linux Releases, official Talos Linux system extensions, official Talos Overlays (booting and installing images can be obtained)
OR
imager container                -       ghcr.io/siderolabs/imager:<version>           
                                        to build for your local changes with custom system extensions


### about kernel + initramfs
this is used just for booting, very handy for kvm virtual machines


### about iPXE script



## Second understand what is a platform

Support matrix for a version lists the platforms it is supported on
https://www.talos.dev/<version>/introduction/support-matrix/, e.g. https://www.talos.dev/v1.11/introduction/support-matrix/


Platforms are in these categories

cloud                               -       aws, gcp, azure, openstack, akamai, digital ocean, hetzner, nocloud, exoscale, etc.
metal (bare metal)                  -       x86 - BIOS, UEFI, SecureBoot
                                            arm64 - UEFI, SecureBoot
virtualized                         -       vmware, hyper-v, kvm, proxmox, vagrant+libvirt, xen, opennebula 
sbc                                 -       raspberry pi, banana pi, turing rk1, pine64, etc.
local                               -       qemu, docker, virtualbox

## Third understand what is a schematic

A schematic is a hash
The hash is unique for a combination of 2 things (COULD BE MORE, THIS IS INCOMPLETE)
1. System Extensions
2. Command Line arguments to a kernel


## Generic Boot Assets

If you are not customizing anything then you can obtain a boot asset directly from the Github release page for a particular version of Talos

https://github.com/siderolabs/talos/releases/tag/<version> and look at Assets section
e.g. https://github.com/siderolabs/talos/releases/tag/v1.11.3 

For example you could get the base installer image for a version of Talos
The base installer image    -   kernel + initramfs with Talos init + installer binary


## Talos Image factory service

Factory service is used to build an image based on a schematic and then return that image

### Considerations for building an image using Factory Service

Hardware Types

    1. Bare Metal           -       suitable for x86-64 and arm64 bare metal AS WELL AS generic VM's.
    2. Cloud Server         -       AWS, GCP, Azure, VMWare, Equinix Metal, Proxmox, OpenStack, Digital Ocean, Hetzner and many others
    3. Single Board         -       Raspberry Pi, Pine64, Jetson Nano and similar

Talos Version

Machine architecture
    amd64
    arm64

SecureBoot image (the image is signed with a key and the hardware is pre-configured)

System Extensions needed

Customization
    kernel command line arguments, e.g. 'console=tty0,115200'                           # https://docs.siderolabs.com/talos/v1.12/reference/kernel/
    any embedded machine configuration that needs to be added, e.g. HostNameConfig
    Bootloader type, e.g. BIOS, UEFI, Dual Boot, Auto


### HOW

1. Use the GUI at https://factory.talos.dev

Follow the prompts


2. Use an api call

the process is very simple 
- submit a file (aka schematic) with a customization to the factory service /schematics endpoint 
- and it will return an id that matches the customization.
The id can be used to obtain the boot assets from the image factory service.

```shell
# Create a schematic YAML and get the schematic ID
cat > schematic.yaml << 'EOF'
customization:
  systemExtensions:
    officialExtensions:
      - siderolabs/nvidia-container-toolkit
      - siderolabs/iscsi-tools
      - siderolabs/qemu-guest-agent
EOF

SCHEMATIC_ID=$(curl -s -X POST --data-binary @schematic.yaml https://factory.talos.dev/schematics | jq -r '.id')

# Declare the version
VERSION=v1.12.10

# THE EXAMPLES BELOW ARE FOR HARDWARE TYPE - BARE METAL (good for bare metal and generic VM's)

# Get assets for booting
https://factory.talos.dev/image/${SCHEMATIC_ID}/${VERSION}/metal-amd64.iso
# OR
https://factory.talos.dev/image/${SCHEMATIC_ID}/${VERSION}/metal-amd64.raw.zst
# OR
https://factory.talos.dev/image/${SCHEMATIC_ID}/${VERSION}/metal-amd64.qcow2
# OR
https://pxe.factory.talos.dev/pxe/${SCHEMATIC_ID}$/${VERSION}$/metal-amd64
# OR
https://factory.talos.dev/image/${SCHEMATIC_ID}/${VERSION}/kernel-amd64
https://factory.talos.dev/image/${SCHEMATIC_ID}/${VERSION}/initramfs-amd64.xz


# Get asset for Installing
https://factory.talos.dev/metal-installer/${SCHEMATIC_ID}:${VERSION}


# Use the schematic directly in upgrading
talosctl upgrade --image factory.talos.dev/metal-installer/${SCHEMATIC_ID}:${VERSION} -n <node>
```

Look at the document to know all the possible end points for factory image service


## Imager container

Obtain the imager container itself from - ghcr.io/siderolabs/imager:<version> e.g. ghcr.io/siderolabs/imager:v1.10.7

"base profile"/"image kind" options to the imager container:

NOTE: A BASE PROFILE (aka Image Kind) OPTION BUILDS FOR A SPECIFIC PLATFORM AND ITS TYPE - eg. 'aws' profile option builds for cloud/aws

    iso                                             -   iso                 builds a Talos ISO image (see ISO)
    secureboot-iso                                  -   iso                 builds a Talos ISO image with SecureBoot (see SecureBoot)
    
    metal                                           -   disk image          builds a generic disk image for bare-metal machines
    secureboot-metal                                -   disk image          builds a generic disk image for bare-metal machines with SecureBoot
    aws                                             -   disk image          builds a disk image for the specific cloud platform
    gcp                                             -   disk image          builds a disk image for the specific cloud platform
    azure                                           -   disk image          builds a disk image for the specific cloud platform
    openstack                                       -   disk image          builds a disk image for the specific cloud platform
    nocloud                                         -   disk image          builds a disk image for the specific cloud platform

    secureboot-installer                            -   container image     builds an installer container image with SecureBoot (see SecureBoot)
    image                                           -   i saw this in the documentation at https://www.talos.dev/v1.11/talos-guides/install/bare-metal-platforms/network-config/

The base profile can be customized with the additional flags to the imager:
    --arch                                          -   specifies the architecture of the image to be generated (default: host architecture)
    --meta                                          -   allows to set initial META values (metal network configuration)
    --extra-kernel-arg                              -   allows to customize the kernel command line arguments.
                                                        Default kernel arg can be removed by prefixing the argument with a -. 
                                                        For example -console removes all console=<value> arguments, whereas -console=tty0 removes the console=tty0 default argument.
    --system-extension-image                        -   allows to install a system extension into the image
    --image-cache                                   -   allows to use a local image cache

```shell
docker run --rm -t -v /tmp/out:/out \
  ghcr.io/siderolabs/imager:v1.12.10 \
  iso \
  --arch=amd64 \
  --extra-kernel-arg=-console \
  --extra-kernel-arg=console=tty0 \
  --system-extension-image ghcr.io/siderolabs/nvidia-container-toolkit:535.54.03-v1.13.5 \
  --system-extension-image ghcr.io/siderolabs/iscsi-tools:v0.1.4 \
  --system-extension-image ghcr.io/siderolabs/qemu-guest-agent:8.2.2
```




### Considerations for Bare Metal platform

Bootloader - Talos uses either
    1. GRUB Bootloader (bios based systems on x86_64)

    uses vmlinuz + initramfs + kernel arguments stored in configuration file ( derived from .machine.install.extraKernelArgs)

    Partition layout for GRUB
        MBR         -       initial boot code
        BIOS        -       grub bootloader
        EFI         -       grub bootloader for UEFI systems (only for upgrades from earlier Talos versions)
        BOOT        -       grub configuration file (contains kernel arguments) and Talos boot assets (vmlinux, initramfs)


    2. systemd-boot (UEFI based systems on x86_64 and arm64)


Boot Asset

    ISO

    PXE

    Disk Image


Network Configuration
    This is a way of submitting initial network configuration for a bare metal server prior to installation and configuration
    ```talosctl get meta 0x0a -n <node>```


Secureboot with UEFI based systems


Equinix based bare metal


Matchbox based bare metal



# PRE-REQ - TALOS CONFIGURATION DOCUMENTS

START HERE - https://www.talos.dev/<version>/reference/configuration/

A machineconfig is a yaml document, in the .spec field are many sub documents that make up various configuration aspects

NOTE: when applying config (apply-config) you apply the sub documents directly

v1alpha1                                -       The talos machine and cluster configuration
                                                VERSION=v1.13 # only major and minor version, not patch version
                                                https://www.talos.dev/${VERSION}$/reference/configuration/v1alpha1/config/

NetworkRuleConfig                       -       for a network firewall rule on a talos node
                                                https://www.talos.dev/${VERSION}/reference/configuration/network/networkruleconfig/

TrustedRootsConfig                      -       for additional trusted CA roots for talos
                                                https://www.talos.dev/${VERSION}/reference/configuration/security/trustedrootsconfig/

NetworkDefaultActionConfig              -   


VolumeConfig                            -       for a volume on a talos node (Ephemeral, State, Meta, etc.)
                                                https://www.talos.dev/${VERSION}/reference/configuration/block/volumeconfig/

UserVolumeConfig                        -       for configuring an additional volume/partition on disk


HostNameConfig

ExtensionServiceConfig


and so on, follow the URLs



a node configuration file is always made up of Talos Configuration manifests

v1alpha1
v1alpha1/ExtensionServiceConfig
v1alpha1/TrustedRootsConfig
v1alpha1/NetworkDefaultActionConfig
v1alpha1/NetworkRuleConfig
and so on



# INSTALLING TALOS

## Boot your Talos node first
look above for booting

## Installing with talosctl

### the files
Control plane nodes configuration
Worker nodes configuration
Secrets


### method
talosctl gen secrets  	----->  secrets.yaml		            # cluster id/secret, bootstraptoken, trustdinfo token, etcd cert/key, k8s crt/key, k8sserviceaccount cert/key, os crt/key
talosctl gen config		-----> 	control plane
talosctl gen config		----->	worker

talosctl apply-config


## Installing with talhelper

### the files
talconfig.yaml													# the config for the cluster with controlplane and worker nodes, you create this file				CAN GO IN GIT
manifests/<manifest>											# a talos api manifest (generally other than a machineconfig), e.g. v1alpha1/TrustedRootsConfig		CAN GO IN GIT
patches/<patch>													# either a patch of a talos machine config file (careful, not of talconfig!) 						CAN GO IN GIT
                                                                # OR
																# a complete k8s resource via cluster.inlineManifests that will be applied by talos into k8s 	CAN GO IN GIT

### method
```shell
# Construct a talconfig.yaml file
vi talconfig.yaml

# Generate secrets for a Talos cluster using talhelper
talhelper gensecret > talsecret.sops.yaml		

# Configure sops - if using vault then make sure VAULT_TOKEN is exported in the environment
vi .sops.yaml

# Encrypt talsecret.sops.yaml with sops in place
sops -e -i talsecret.sops.yaml		# CAN GO IN GIT

# Generate config for cluster based on talconfig.yaml and talsecrets.sops.yaml (encrypted file) 
talhelper genconfig

# you will get a clusterconfig/ directory with a machineconfig for each node and the equivalent talosconfig
ls clusterconfig/<nodeconfig>
ls clusterconfig/talosconfig

# DO NOT PUT ./clusterconfig IN GIT

# EITHER apply config to a VM booted with a talos image or kernel/initramfs
talosctl apply-config -n <node_ip> --insecure --file ./clusterconfig/<nodeconfig>		
# OR use talhelper gencommand
talhelper gencommand apply --extra-flags '--insecure'

 # to merge the clusters config into your existing config file for talos - mostly ~/.talos/config
talosctl config merge ./clusterconfig/talosconfig				               
```


## Secrets in the talsecrets (talsecret.sops.yaml) file [the stuff we are trying to protect]

cluster id
cluster secret

bootstraptoken secret
secretboxencryption secret

trustdinfo token

etcd crt and key
k8s crt and key
k8saggregator crt and key
k8sserviceaccount key                   -       signs jwt tokens for the service accounts
os crt and key




# KUBERNETES CONTROL PLANE COMPONENTS IN TALOS

## apiserver, controller-manager and scheduler components run as static pods

```shell
# Note: these are run directly by kubelet bypassing the kube api-server

talosctl -n nsw-etca-opsk8stest-ctrl1 get staticpods

NODE                        NAMESPACE   TYPE        ID                        VERSION
nsw-etca-opsk8stest-ctrl1   k8s         StaticPod   kube-apiserver            1
nsw-etca-opsk8stest-ctrl1   k8s         StaticPod   kube-controller-manager   1
nsw-etca-opsk8stest-ctrl1   k8s         StaticPod   kube-scheduler            1
```

## etcd and kubelet components runs as a service

```shell
talosctl -n nsw-etca-opsk8stest-ctrl1 get service etcd

NODE                        NAMESPACE   TYPE      ID     VERSION   RUNNING   HEALTHY   HEALTH UNKNOWN
nsw-etca-opsk8stest-ctrl1   runtime     Service   etcd   2         true      true      false


$ talosctl -n nsw-etca-opsk8stest-ctrl1 get service kubelet  
NODE                        NAMESPACE   TYPE      ID        VERSION   RUNNING   HEALTHY   HEALTH UNKNOWN
nsw-etca-opsk8stest-ctrl1   runtime     Service   kubelet   2         true      true      false
```

# SERVICES ON ANY TALOS NODE

## The following run as services on a node (some are related to Talos OS and others to K8s)

```shell
talosctl -n nsw-etca-opsk8stest-ctrl1 get services

NODE                        NAMESPACE   TYPE      ID                     VERSION   RUNNING   HEALTHY   HEALTH UNKNOWN
nsw-etca-opsk8stest-ctrl1   runtime     Service   apid                   2         true      true      false
nsw-etca-opsk8stest-ctrl1   runtime     Service   auditd                 2         true      true      false
nsw-etca-opsk8stest-ctrl1   runtime     Service   containerd             2         true      true      false
nsw-etca-opsk8stest-ctrl1   runtime     Service   cri                    2         true      true      false
nsw-etca-opsk8stest-ctrl1   runtime     Service   dashboard              1         true      false     true
nsw-etca-opsk8stest-ctrl1   runtime     Service   etcd                   2         true      true      false
nsw-etca-opsk8stest-ctrl1   runtime     Service   ext-qemu-guest-agent   1         true      false     true
nsw-etca-opsk8stest-ctrl1   runtime     Service   kubelet                2         true      true      false
nsw-etca-opsk8stest-ctrl1   runtime     Service   machined               2         true      true      false
nsw-etca-opsk8stest-ctrl1   runtime     Service   syslogd                2         true      true      false
nsw-etca-opsk8stest-ctrl1   runtime     Service   trustd                 2         true      true      false
nsw-etca-opsk8stest-ctrl1   runtime     Service   udevd                  2         true      true      false


## Some of the above services could be running in containers (dont forget difference between a container and a pod)

talosctl -n nsw-etca-opsk8stest-ctrl1 containers

NODE                        NAMESPACE   ID                     IMAGE        PID    STATUS
nsw-etca-opsk8stest-ctrl1   system      apid                                2369   RUNNING
nsw-etca-opsk8stest-ctrl1   system      ext-qemu-guest-agent                2287   RUNNING
nsw-etca-opsk8stest-ctrl1   system      trustd                              2565   RUNNING


## Get more information about a service

talosctl -n nsw-etca-opsk8stest-ctrl1 service ext-qemu-guest-agent

NODE     nsw-etca-opsk8stest-ctr1
ID       ext-qemu-guest-agent
STATE    Running
HEALTH   ?
EVENTS   [Running]: Started task ext-qemu-guest-agent (PID 2290) for container ext-qemu-guest-agent (22h21m0s ago)
         [Preparing]: Creating service runner (22h21m0s ago)
         [Preparing]: Running pre state (22h21m0s ago)
         [Waiting]: Waiting for service "containerd" to be "up", file "/system/run/machined/machine.sock" to exist, file "/dev/virtio-ports/org.qemu.guest_agent.0" to exist (22h21m1s ago)
         [Starting]: Starting service (22h21m1s ago)


## Get logs for a service

talosctl logs <service> -n <node>
```

# VOLUMES ON A TALOS NODE

## Volume (aka partitions) Labels 

EFI                 -   the partition used for storing the bootloader files for UEFI based systems
BIOS                -   the partition used for storing the bootloader files for legacy bios systems
BOOT                -   contains the kernel, initramfs, grub config and other boot related data (Talos OS) {I think only for GRUB bootloader based systems}
META                -   the volume used for storing Talos metadata
STATE               -   the volume used for storing system state including machine configuration and PKI data
EPHEMERAL           -   the volume for /var
                        largest writable data volume that holds runtime data for the node
                        used for storing container data, downloaded images, logs, metrics, etcd data for controlplanes
                        it is a catch all location for storing data
                        the volume is erased on reboot

# STAGES OF TALOS OS

Installing

Rebooting

Running             -   Talos booted up, found configuration in STATE, running with that configuration

Maintenance         -   Talos booted up, no configuration was found in STATE, waiting for configuration to be applied


## Resetting a Talos node

Removes the machine from kubernetes, removes etcd and clears the following data - EPHEMERAL, STATE, extra disks and reboots the node into Maintenance stage
Now the machine is waiting for a new configuration


### Full factory Reset 

```shell
talosctl -n $node reset
```


### Just resetting STATE and EPHEMERAL

```shell
talosctl reset --system-labels-to-wipe STATE --system-labels-to-wipe EPHEMERAL
```



# THE DISCOVERY SERVICE

A service to help kubernetes cluster nodes discover one another

## via kubernetes registry (etcd)

## via external registry
https://discovery.talos.dev


# THE KEY MANAGEMENT SERVER

A place to hold keys used for encryption, e.g. the key that encrypts the disk at rest


# TALOS LINUX SYSTEM EXTENSIONS

## What
Extensions allow for additional functionality on top of the default Talos Linux capabilities.
kernel modules
firmware
user-space tools
e.g. guest agents for talos vm running on qemu

The base installer image does not have system extensions (i think)

## Where are the extensions available
https://github.com/siderolabs/extensions


## Installing extensions
https://github.com/siderolabs/extensions#installing-extensions




# KUBESPAN

KubeSpan is a feature of Talos that automates the setup and maintenance of a full mesh WireGuard network for your cluster




# CONSOLES

Physical video console
    First virtual TTY       -       kernel logs
    Second virtual TTY      -       Interactive dashboard

Serial console


# ROLES

os:reader
os:operator                 -       os:reader role + additional privileges for managing etcd, such as performing backups and managing etcd alarms
os:admin                    -       This is the most powerful role, granting access to all API methods, which includes all etcd-related operations
os:etcd:backup              -       a specific role that grants permission to perform etcd backups by calling the /machine.MachineService/EtcdSnapshot method.



# TALOS KERNEL PARAMETERS

https://www.talos.dev/v1.11/reference/kernel/


# NAMESPACES IN TALOS 

Looks like configuration documents, e.g. MetaKey or MachineConfiguration, etc. live in namespaces similar to how
Kubernetes resources live in namespaces. 

here is a list of namespaces i have come across with examples of configuration items

hardware            -       SystemInformations.hardware.talos.dev
runtime             -       MetaKeys.runtime.talos.dev, SystemDisks.block.talos.dev, MachineStatuses.runtime.talos.dev
                            SecurityStates.talos.dev
config              -       Machineconfigs.config.talos.dev/v1alpha1
k8s


i saw pods and their containers running in a k8s.io namespace on a talos node
```shell
talosctl container --kubernetes -n <node>
```

containers not part of kubernetes but part of talos os run in the 'system' namespace


# IMAGES FOR A RELEASE OF TALOS

Each version of Talos has a set of default images it will use either for installaing or for the kubernetes control plane components

## Registries

These are the 3 registries to know
ghcr.io
gcr.io
registry.k8s.io

```shell
$ talosctl images default

ghcr.io/siderolabs/installer:v1.10.7
ghcr.io/siderolabs/flannel:v0.26.7
ghcr.io/siderolabs/kubelet:v1.33.4

gcr.io/etcd-development/etcd:v3.5.21

registry.k8s.io/coredns/coredns:v1.12.1
registry.k8s.io/kube-apiserver:v1.33.4
registry.k8s.io/kube-controller-manager:v1.33.4
registry.k8s.io/kube-scheduler:v1.33.4
registry.k8s.io/kube-proxy:v1.33.4
registry.k8s.io/pause:3.10
```


# COMMANDS

## TALOS DISCOVERY

NOTE: as a general rule add -n <node>, otherwise every node will be queried

```shell
node=nsw-etca-opsk8stest-ctrl1

# get the machine status of a node, i.e. the talos view of a node
talosctl -n $node get machinestatus  -o yaml

# get systeminformation
talosctl -n $node get systeminformation
talosctl -n $node get systemdisk

talosctl -n $node get securitystate -o yaml


# get the machine configuration of a node
talosctl -n $node get machineconfig
talosctl -n $node get machineconfig [ID] -o yaml


# get the image that the node was installed with
talosctl -n nsw-etca-opsk8stest-ctr1l get machineconfig v1alpha1 -o yaml | grep 'image:'


# devices (meaing PCI bus)
talosctl -n $node get devices 


# disk information and volume information
talosctl -n $node get disks
talosctl -n $node get disks vda -o yaml
talosctl -n $node ls -l /dev/disk/by-id
talosctl -n $node ls -l /dev/disk/by-label
talosctl -n $node ls -l /dev/disk/by-uuid


talosctl -n $node get blockdevices              # disks and partitions

talosctl -n $node get discoveredvolumes         # a volume is on a blockdevice of some sort


# volumeconfiguration information (still dont understand what this means)
talosctl -n $node get volumeconfigs
talosctl -n $node get volumeconfigs STATE -o yaml


# mount information
talosctl -n $node mountstatus
talosctl -n $node mounts
## looking at the /var mount
talosctl -n $node mounts | awk 'NR == 1 { print; next; }  $2~/dev\//&&$2!~/rbd/{ print }'
## show mounts from kubelet into the pods
talosctl -n $node  mounts | awk 'NR == 1 { print; next; }  $7~/\/var\/lib\/kubelet\/pods/ { print }'



# get the running processes on a node (this stupid command does not use get)
talosctl -n $node processes

# usage information
talosctl -n $node usage <mount_point>, e.g. talosctl -n $node usage /var


# network information
talosctl -n $node get linkstatus                            # the status of a link, aliased to link, links, ls
talosctl -n $node get linkstatus <linkstatus> -o yaml
talosctl -n $node get addresses
talosctl -n $node get routes
talosctl -n $node get resolver
talosctl -n $node get hostname
talosctl -n $node get timeserver


# kernel configuration
## e.g. looking for libceph compilation
talosctl -n $node cat /proc/config.gz | zgrep -E "^CONFIG.*_(RBD|CEPH)"


# get meta information (this only applies to bare-metal nodes)
talosctl -n nsw-etca-opsk8s-osd1 get meta 0x0a -o yaml


## get extensions in Talos OS
talosctl -n nsw-etca-opsk8s-ctrl1 get extensions


# get manifests that have been applied by talos to the kubernetes cluster, e.g. to provision coredns in kubernetes for example 
NOTE: These are of type Manifests.kubernetes.talos.dev and NOT v1alpha1
talosctl -n nsw-artm-opsk8stest-ctrl1 get manifests
talosctl -n nsw-artm-opsk8stest-ctrl1 get manifests 11-core-dns -o yaml


# get the member nodes of the cluster
talosctl -n $node get member
talosctl -n $node get member nsw-artm-opsk8stest.ctrl1 -o yaml


# get health of cluster
talosctl -n $node health


# get containers of Talos 
talosctl -n $node containers                        # not kubernetes containers
talosctl -n $node containers --kubernetes           # kubernetes containers


# Talosctl cluster configuration information
talosctl config info


# Talos kubernetes components images for current version of Talos
talosctl images default


# etcd
talosctl -n nsw-etca-opsk8stest-ctrl1 service etcd

NODE     nsw-etca-opsk8stest-ctrl1
ID       etcd
STATE    Running
HEALTH   OK
EVENTS   [Running]: Health check successful (148h8m29s ago)
         [Running]: Started task etcd (PID 2665) for container etcd (148h8m34s ago)
         [Preparing]: Creating service runner (148h8m34s ago)
         [Preparing]: Running pre state (148h8m34s ago)
         [Waiting]: Waiting for service "cri" to be "up", time sync, network, etcd spec (148h8m35s ago)
         [Starting]: Starting service (148h8m35s ago)

## remove an etcd member
talosctl -n $node etcd members
talosctl -n $node etcd remove-member <member-id>


# Logs for a service
talosctl -n $node logs <service> # talosctl -n $node get services


# There about 161 resource definitions that you can retrieve for a node
talosctl -n $node get rd 

# within these categories
$ talosctl -n nsw-etca-ossk8sdev-swrk7 get rd  | sed '/NODE/d' | awk '{ print $4 }' | while read rd; do echo "${rd#*.}"; done | sort | uniq
block.talos.dev
cluster.talos.dev
config.talos.dev
cri.talos.dev
etcd.talos.dev
files.talos.dev
hardware.talos.dev
k8s.talos.dev
kubernetes.talos.dev
kubespan.talos.dev
meta.cosi.dev
net.talos.dev
perf.talos.dev
runtime.talos.dev
secrets.talos.dev
siderolink.talos.dev
talos.dev
v1alpha1.talos.dev

```



## KUBERNETES DISCOVERY

```shell
## what pods are running in the host network
kubectl get pods -A | grep <host_ip_prefix>
e.g. for opsk8stest - kubectl get pods -A | grep 10.253


# gets 3 certificates and their keys - apiServer, apiServerKubeletClient, frontProxy
talosctl get KubernetesDynamicCerts -o yaml -n nsw-artm-opsk8sdev-ctrl1


# kubernetes containers running on a Talos node
talosctl -n $node containers -k


# get the configuration of the apiserver
talosctl -n $node get apiserverconfig -o yaml
```



## TALOS TASKS

```shell

######## WHERE DID I GET THIS FROM?
## upgrade a talos cluster
## talosctl -n nsw-etca-opsk8stest-ctrl1 upgrade --image ghcr.io/talos-systems/installer:v0.14.3
########

## upgrade a talos node
talosctl upgrade --nodes nt-drwn-opsk8stest-swrk1 --image factory.talos.dev/metal-installer/83d2fae90cb60257aa55fdf2bf14bb33c988162e1a715dc66da554a864e1c2b1:v1.9.6

## get an image onto a node
talosctl image pull <image> -n <node>

## patch a machineconfig
talosctl patch mc --patch-file 40-machine-images-from-aplregistry.yaml -n nsw-etca-opsk8stest-iwrk1
```


## TALOS TROUBLESHOOTING

```shell
# get the health of a cluster
talosctl health -n $NODE

# the logs of kubelet on a node (remember that kubelet runs control-plane static pods bypassing the api-server)
talosctl logs kubelet -n $NODE
```


# THE TALOS CLI REFERENCE

The talos cli reference
https://www.talos.dev/v1.9/reference/cli/


# COMMANDS ON WIKI

https://aplwiki.aarnet.edu.au/display/SA/talosctl+cheatsheet