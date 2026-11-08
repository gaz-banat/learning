# Overview of the Boot Stages

The Talos Linux boot process follows these major stages:

Firmware initialization (UEFI or BIOS)
Boot loader execution (GRUB or systemd-boot)
Kernel loading and early hardware initialization
Initramfs and machined startup
Disk discovery and mounting
Machine configuration application
Network configuration
Kubernetes component startup
Cluster readiness

Each stage builds on the previous one, and a failure at any point will prevent later stages from completing.

## Stage 1: Firmware Initialization
When you power on a machine, the firmware (UEFI or legacy BIOS) initializes hardware and looks for a bootable device. For UEFI systems, the firmware reads the EFI System Partition (ESP) and looks for a boot loader. For legacy BIOS, it reads the Master Boot Record (MBR).

Talos Linux supports both UEFI and legacy BIOS boot, but UEFI is recommended for modern hardware. UEFI provides several advantages:

Secure Boot support (with proper key enrollment)
Faster boot times
Support for GPT partition tables (allowing disks larger than 2TB)
Standardized boot manager interface


## Stage 2: Boot Loader
Talos Linux uses one of two boot loaders:

GRUB: The traditional GNU GRand Unified Bootloader. It is used for legacy BIOS systems on x86_64 and may also be present on UEFI systems that were upgraded from older Talos versions. GRUB reads its configuration from a file and loads the kernel and initramfs.

systemd-boot: A simpler boot loader that is part of the systemd project. It is the default boot loader for new UEFI installations starting with Talos 1.10. systemd-boot is faster and simpler than GRUB but only works with UEFI.

The boot loader's job is to:

Present a boot menu (if configured)
Load the Talos Linux kernel and initramfs into memory, either separately with GRUB or as a Unified Kernel Image (UKI) with systemd-boot
Pass kernel command-line parameters (embedded in the UKI when using systemd-boot)
Transfer control to the kernel

```shell
# You can see the kernel command line on a running system
talosctl read /proc/cmdline
```

Typical kernel parameters for Talos Linux include:

```shell
talos.platform=metal console=ttyS0 console=tty0 init_on_alloc=1 slab_nomerge pti=on
```

## Stage 3: Kernel Initialization
Once the boot loader hands off to the kernel, Linux performs hardware initialization:

CPU detection and configuration
Memory management setup
Hardware driver loading
PCI device enumeration
Storage controller initialization
Network interface discovery
The kernel extracts the initramfs into a temporary root filesystem in RAM. This initramfs contains the Talos Linux init process and the minimum tools needed to continue booting.


```shell
# View kernel boot messages
talosctl dmesg
# The dmesg output shows every step of kernel initialization, which is invaluable for diagnosing hardware compatibility issues.
```

## Stage 4: Machined and Init
After the kernel mounts the initramfs, it starts the Talos Linux init process. This is not systemd or any traditional init system. Talos Linux uses its own init process called machined.

Machined is responsible for:

Setting up the initial process environment
Mounting essential filesystems (proc, sys, dev, cgroup)
Starting the machine configuration service
Managing all subsequent system services

```shell
# Check the status of Talos services
talosctl service
# This is where Talos Linux diverges significantly from other Linux distributions. There is no systemd, no init scripts, and no runlevels. Machined manages a fixed set of services that are all directly related to running Kubernetes.
```

## Stage 5: Disk Discovery and Mounting
Machined discovers and mounts the necessary disk partitions:

EFI System Partition (ESP): Contains the boot loader and, on systemd-boot systems, Talos UKIs
BOOT partition: Contains GRUB configuration and Talos boot assets on GRUB-based installations
META partition: Stores machine metadata and configuration
STATE partition: Holds system state, including machine configuration
EPHEMERAL partition: Working space for container data, logs, and Kubernetes data such as etcd on control plane nodes

```shell
# View disk and partition information
talosctl get disks
talosctl get discoveredvolumes

# Check mounted partitions
talosctl get mountstatus
```

The STATE partition is particularly important because it stores sensitive node data such as machine configuration, secrets, and certificates. Talos supports LUKS2 encryption for STATE and EPHEMERAL, but disk encryption must be configured; it is not automatically enabled on every installation.

## Stage 6: Machine Configuration
Once the STATE partition is mounted, machined reads the machine configuration. This configuration determines everything about how the node behaves:

Network settings
Cluster membership
Kubernetes component configuration
System extensions and custom settings
If this is the first boot and no configuration exists on the STATE partition, Talos enters maintenance mode and waits for a configuration to be applied via the Talos API.


```shell
# Check the current machine configuration
talosctl get machineconfig

# View the configuration status
talosctl get machinestatus
```

## Stage 7: Network Configuration
With the machine configuration loaded, Talos sets up networking:

Network interfaces are configured (DHCP or static)
DNS resolvers are set
NTP time synchronization starts
The Talos API endpoint becomes available

```shell
# Check network interface status
talosctl get addresses
talosctl get links
talosctl get routes
Network configuration happens before Kubernetes starts because the Kubernetes components need network connectivity to function. If you have ever seen a Talos node stuck in boot, network issues are often the cause.
```

## Stage 8: Kubernetes Components
Once networking is up, machined starts the Kubernetes components. The order depends on whether the node is a control plane or worker:

### Control Plane Nodes:

etcd starts and initializes (or joins an existing cluster)
The kubelet service starts
The Kubernetes API server, controller manager, and scheduler start as static pods via the kubelet
The kubelet obtains its client certificate and registers the node
kube-proxy and the configured CNI start from the Kubernetes bootstrap manifests, if enabled

### Worker Nodes:

The kubelet starts and connects to the API server
kube-proxy and the configured CNI start from Kubernetes manifests, if enabled
The node registers with the cluster

```shell
# Monitor service startup
talosctl service

# Watch etcd status (control plane only)
talosctl service etcd

# Check kubelet status
talosctl service kubelet
```

## Stage 9: Cluster Readiness
The final stage is cluster readiness. This means:

etcd has a healthy quorum (for HA clusters)
The API server is accepting connections
All control plane components are running
The node is registered in Kubernetes
CNI networking is functional
CoreDNS is running

```shell
# Check overall cluster health
talosctl health

# This checks:
# - etcd cluster health
# - API server availability
# - All nodes are ready
# - DNS is functional
```

# The Boot Process when upgrade is done

When you upgrade Talos Linux, the boot process has an additional twist. Talos uses an A/B image scheme:

The new Talos version is written as the inactive boot image
The boot loader is updated to point to the new image
The machine reboots into the new version
If the boot fails, the machine can fall back to the previous version

```shell
# Initiate an upgrade
talosctl upgrade --image ghcr.io/siderolabs/installer:v1.10.0

# The upgrade process:
# 1. Downloads the new installer image
# 2. Writes the new boot image
# 3. Updates boot loader config
# 4. Reboots
```

# Debugging Boot Issues
When something goes wrong during boot, use these tools:

```shell
# View all boot messages
talosctl dmesg

# Check service status
talosctl service

# View logs for a specific service
talosctl logs machined
talosctl logs etcd
talosctl logs kubelet

# Check the machine configuration is valid
talosctl get machineconfig -o yaml
```

For pre-boot issues (firmware or boot loader problems), you need physical console access or an IPMI/BMC connection to see what is happening.

