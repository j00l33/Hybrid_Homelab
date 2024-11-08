Proxmox acts as a hypervisor which is used to create and host the virtual machine and virtual network infrastructure.

There are five VLANS within the network, containing six VMs

##VLANS

Active Directory Server VLAN:

	Contains the Active Directory Server VM

Active Directory Worker VLAN:

	Contains the Active Directory Workstation VM

Network Services VLAN:

	Contains the DHCP/DNS Server and the Logging and Monitoring Server VM

Local Management VLAN:

	Contains the VM used to configure and troubleshoot network services VM

Cloud Backup VLAN:

	Contains the VM used to collect backup and then push it to the cloud.

##Bridges

lanBridge:

Serves as a switch within the interior of the virtual network.
Connection to OPNsense VM through a network interface acting as a trunk port.
Works with the router to perform inter-VLAN routing.
Five network interfaces connect to each VLAN.

wanBridge:

Serves as an external switch to connect the router to the Proxmox node.
It acts as strictly a 1:1 connection between the two interfaces.
Proxmox node preforms NAT for outbound traffic.

