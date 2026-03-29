

# KVM HYPERVISOR (Kernel Space)

Kernel based Virtual Machine 		
The virtualization technology similar to ESXi, HyperV, HyperKit, VirtualBox, Xen




# QEMU (User Space)

Quick Emulator (QEMU) is a machine (hardware) emulator


The PC hardware emulated by QEMU includes a motherboard, network controllers, SCSI, IDE and SATA controllers, serial ports (the complete list can be seen in the kvm(1) man page) all of them emulated in software. All these devices are the exact software equivalent of existing hardware devices, and if the OS running in the guest has the proper drivers it will use the devices as if it were running on real hardware. This allows QEMU to run unmodified operating systems.

QEMU can use KVM  and can run with the support of the virtualization processor extensions, via the Linux KVM module





# LIBVRT

is a virtualization/hypervisor management library which wraps QEMU and KVM to provide APIs for use by other programs, such as Vagrant, which is a tool for creating virtualized development environments.

So - vagrant uses libvirt and libvirt uses QEMU and KVM




# PARAVIRTUALIZED

This however has a performance cost, as running in software what was meant to run in hardware involves a lot of extra work for the host CPU. 
To mitigate this, QEMU can present to the guest operating system paravirtualized devices, where the guest OS recognizes it is running inside QEMU and cooperates with the hypervisor.




# COMMANDS


NOTE: 2 types of utilities from libvirt - virt-* and virsh

```shell


# I want to install a new VM

virt-install \
    --connect qemu:///system \
    --name ${VMNAME} \
    --virt-type kvm \
    --vcpus $VCPUS \
    --memory $MEMORY \
    --machine q35 \
    --osinfo name=linux2022,detect=false \
    --features acpi=on,apic=on,vmport.state=off \
    --clock offset=utc,rtc_tickpolicy=catchup,pit_tickpolicy=delay,hpet_present=no \
    --pm suspend_to_mem.enabled=off,suspend_to_disk.enabled=off \
    --disk path=/srv/libvirt/images/disks/${VMNAME}.${DOMAIN}.vda,size=${DISK_SIZE},format=qcow2,cache=writeback,target.bus=virtio \
    --network bridge=br-net0,model=virtio,driver.queues=${VCPUS} \
    --memballoon none \
    --rng /dev/urandom \
    --tpm default \
    --install kernel=/srv/libvirt/images/boot/vmlinuz-amd64,initrd=/srv/libvirt/images/boot/initramfs-amd64.xz \
    --extra-args "net.ifnames=0 ip=${IPADDR}::${IPGWY}:${NETMASK}:${VMNAME}:eth0:none:${DNS1}:${DNS2}:${NTP1} console=ttyS0,115200n8 talos.platform=metal slab_nomerge pti=on consoleblank=0" \
    --controller type=virtio-serial,driver.iommu=on \
    --graphics vnc \
    --console pty \
    --serial pty \
    --autostart
 
 
#    --cpu host-passthrough \
#    --network bridge=br-net0,model=virtio,driver.queues=${VCPUS},mtu.size=9000 \
 

domain=<domain_name>

# Get all running domains
virsh list —all


# Creating a domain

## EITHER
## Create a domain from an xml file (the long way)
virsh define $domain.xml
## OR
## Use virsh install

## Ensure a domain automatically starts on hypverisor boot
virsh autostart $domain

## Start a domain now
virsh start $domain


# Information for a domain

## Get running info about a domain
virsh dominfo $domain
virsh domstate $domain

## Get the xml info for a domain
virsh dumpxml $node >$node.xml


# Configuring a domain

## Configure the xml definition of a domain WHEN IT IS SWITCHED OFF
virsh edit $domain				# set vcpus to 12

## various configurations from command line
virsh setvcpus $domain 12 --config
virsh setmaxmem $domain 50331648 -—config  ## value in KiB
virsh setmem $node 50331648 -—config ## value in KiB


# Removing a domain

## Ensure a domain does not autostart
virsh autostart --disable $domain

## shutdown a domain
virsh shutdown $domain

## need to understand exactly what destroy does
virsh destroy $domain


```