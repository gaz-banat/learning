

IT COMPONENTS


Applications

Data

Runtime

Middleware

OS

Virtualization

Servers

Storage

Networking


COMPUTING ARCHITECTURES


CISC used in personal computers

x86_64 or x64 or amd64 64 bit Intel and AMD

IA-64 Intel Itanium


RISC



ARM (Advanced RISC Machine) used in smartphones, tablets, and other
portable/iot devices


POWERPC

IBM Power 8




IBM Z


CPU



MEMORY



DISK




PROCESS

- has access to memory
- is assigned a number of file handles


THREAD

- is a lightweight process
- has access to the memory of its parent process



LINE ENDINGS


In Unix line endings are just LF (aka new line) - \\n, hex 0a

In Windows line endings are CR LF - \\r\\n, hex 0d0a
In windows files end with ctrl+z



To look at line endings in a file in Linux use hexdump
hexdump -C \<file\>




ENCODINGS


ASCII UTF-8

bits per character 7
total characters 128
printable characters 95



IMAGE FILE FORMATS

.raw represents a plain, unstructured disk image with no compression or
optimization applied
.raw.xz or raw.gz plain unstructured disk which is compressed
.vhd
.tar.gz compressed archive
.iso iso image (iso9660)
.qcow2 QEMU Copy-On-Write version 2 supports compression, snapshots and
thin provisioning.
.ova



BOOTING




##BIOS BASED (think MBR)



Sector 0 \| Sector 1 - 62 \| Sector 63

\| \|
512 Bytes \| 62 x 512 bytes = 31744 bytes \|

MBR \| \| First Partition

boot record \| \|

bootstrap code (stage 1 code) \| stage 1.5 code \| /boot filesystem
(generally here but not necessarrily)

(boot.img \~ 446 bytes) \| (core.img \~ 25,389 bytes) \| /boot/grub2
(the stage2 code)




##UEFI BASED (think GPT)


Secure Boot - UEFI Secure Boot requires that the operating system kernel
is signed with a recognized private key, which the system's firmware
verifies using the corresponding public key


.efi Extensible Firmware Interface,
A file with the EFI [[file
extension](https://www.lifewire.com/what-is-a-file-extension-2625879)]{.underline} is
an Extensible Firmware Interface file.
They are boot loader executables, exist on UEFI (Unified Extensible
Firmware Interface) based computer systems, and contain data on how
the [[boot
process](https://www.lifewire.com/what-does-booting-mean-2625799)]{.underline} should
proceed


BootOrder Variable in the UEFI firmware, this variable points to a .efi
file that is the bootloader for your operating system
if empty the motherboards boot manager looks in predefined places like
disks, optical drives, network (pxe)
motherboards boot manager tries to locate the .efi file


Efi boot files

WIndows \<driver\>:\\efi\\boot\\bootx64.efi the drive for the
installation media
Mac /System/Library/Coreservices/boot.efi the disk
Linux /EFI/BOOT/BOOTX64.efi RHEL9, on the installation media, other efi
files are also available



GPT - a way of partitioning disks like MBR but different to MBR in many
ways, more partitions, many copies of the partition table across the
disk, support for larger disks than 2TB, etc.



\### UEFI/BIOS BASED BOOT PROCESS - OPERATING SYSTEM IS NOT INSTALLED

Ask yourself where is the installation media?
On a CDROM
On a USB device
On an NFS share
On an HTTP/HTTPS server


###UEFI/BIOS BASED BOOT PROCESS - OPERATING SYSTEM IS INSTALLED

The UEFI/BIOS is the first stage of booting a computer and is not part
of the operating system.
It's stored in nonvolatile memory, like ROM or flash memory on the
computer's motherboard.
It ensures that all the core hardware connected to the computer is in
working condition so the computer can successfully boot.
Once this check is done,
EITHER
BIOS locates the boot sector on any attached bootable device
the first boot sector it finds that contains a valid boot record is
loaded into RAM
and control is transferred to the code that was loaded from the boot
sector (GRUB stage 1, very small bootstrap code, fits into the first
512-byte sector)
(loads the Master Boot Record or MBR from the primary hard drive, MBR is
in sector 0)
OR
UEFI loads the GPT (GUID Partition Table) equivalent

in either of these cases this is where GRUB2 is stored.

GRUB2 (GRand Unified Bootloader, version 2)
Stage 1 - was loaded by BIOS/UEFI as seen above, its job is to locate
stage 1.5 code
Stage 1.5 - the code in this stage has file system drivers to mount
/boot and find and load the stage 2 code
Stage 2 - Look at the Linux OS file

If there is more than one operating system installed, it will display
the GNU GRUB screen for a few seconds in case the user wants to select
one of the options, otherwise it loads the kernel of whichever operating
system was set as default.

Kernel
The kernel performs the following steps in read-only mode:
First, it loads the memory, then it mounts and loads the root file
system before switching to the actual filesystem.
Then, it loads the devices in the system, remounts the filesystem in
read-write mode, and mounts up.
Then it calls the relevant init program

Init (SystemD since RHEL 7)
In the final stage, your computer uses init files to execute a runlevel
program, which is responsible for starting and stopping services on the
machine and presenting the user login prompt.



\### PREBOOT EXECUTION ENVIRONMENT (PXE) (aka network boot) - OPERATING
SYSTEM IS NOT INSTALLED

Client must support PXE in its BIOS/UEFI or NIC firmware
DHCP server should be available on the network
TFTP server should be available on the network

Client broadcasts a DHCP request and a PXE request
DHCP server responds, it uses options 66 and 67 to advertize the PXE
boot server IP address (the TFTP server and the file name of the NBP)
Client contacts PXE boot server after reading DHCP options
Client downloads and boots the Network Bootstrap Program using TFTP
NBP is a kernel with some basic drivers and programs that can download
the remaining OS components
