To make use of the limited RAM on a single PC, utilizing a type 1 hypervisor was a priority. 
Proxmox was selected due to being open-source, utilization of the Linux networking stack, and flexible networking capabilities.

The hardware solution involved a Proxmox Virtual Environment installation on a portable external drive. This allowed for my PC to remain mostly untouched, and for the internal hard drive to remain unpartitioned. I could simply boot from the external drive rather than the internal drive.

The other options I considered, but chose not to employ, were to:
Partition my PC's SSD
	I did not want to partition my internal SSD as I wanted to mitigate risk of corrupting my ordinary environment.
Virtualize my existing host configuration and use that as my ordinary environment.
	I did not want to virtualize my existing environment as I did not want to worry about PCI passthrough for my GPU or risk losing my existing environment.

The setup process was as follows:

##Create a bootable Debian USB installer:

	-Downloaded the Debian ISO
	-Used Rufus to create bootable USB
	-Booted from USB
	-Partitioned the target external drive into 4 partitions via the Debian installer
	-Installed Debian on target drive

##Partitions:

EFI System Partition

-Very small sized partition that contains the bootloader and serves as point of initial contact from the UEFI firmware to the OS.

Boot Partition

-Small sized partition which stores the kernel and initrd files, providing separation from the root partition which keeps kernel and ram disk files accessible even if root partition is corrupted.

Root Partition

-Large sized partition which stores the bulk of the system files including the OS and is where Proxmox and most of its data reside.

Swap Partition

-Medium sized partition which serves as an emergency extension to ram to prevent out of memory situations.

##Bootloader and EUFI configuration:

GRUB is used to bootstrap the kernel in which Proxmox exists on. The PC uses UEFI on startup, which then finds and loads the bootloader. GRUB then bootstraps the Linux kernel, and startup can begin.

GRUB was installed to the USB drive, enabling it to be booted on UEFI systems. 

The grub.cfg had to be manually edited to identify and load the kernel by making use of UUIDs to specify drives. In addition, the removable parameter was added since this is a USB drive installation.

The UEFI boot entries were edited with efibootmgr.

This ensured the USB was recognized as a boot option by the UEFI process. 

##Proxmox Installation:

-Ensured system was updated

-Installed prerequisite software

-Installed Proxmox and open-iscsi for addition storage support

-Verified Proxmox installation

-Updated systemd to handle Proxmox daemon

-Configured network interfaces and static IP

-Configured Firewall to enable web interface accessibility