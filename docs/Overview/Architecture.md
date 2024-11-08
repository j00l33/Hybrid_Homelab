To design and layout the network at a high-level, I have organized different elements of the network into three overarching categories:

1. Virtualized Network includes the virtual machines and components of the virtual network.
2. Cloud Infrastructure includes the cloud services that are utilized.
3. Network Services includes relevant software and services that are used as a function of the network.

##Virtualized Network:

Proxmox used as a Hypervisor

A single internal bridge acting as a switch used with a virtual router to forward traffic between 5 VLANs

###VLANS

VLAN 10 - Active Directory Server

	Active Directory Server VM

VLAN 20 - Active Directory Worker

	End-user workstation VM

VLAN 30 - Network Services

	DHCP and DNS VM

	Logging and Monitoring VM 

VLAN 40 - Local Management

	OPNsense and local services management VM

VLAN 50 - Cloud Backup and Management

	Cloud management VM

###Firewall/Router

VM running OPNsense will act as both a firewall and router which provides a buffer between the internal network and the Proxmox node.

Handles inter-VLAN routing.

Enforces intranet and internet traffic rules.

Works with the Proxmox node to enable NAT.


##Cloud Infrastructure

AWS used as cloud provider.

###S3 Buckets

	Backup Bucket for network backup data.

	State Bucket for temporary storage of terraform state file.

###VPC

Subnets

	Public Subnet used to contain ephemeral Jump Box and security group.

	Private Subnet used to contain static S3 endpoint.

###Lambda

Create function uses terraform to build and configure the Jump Box and security group.

Clean function uses terraform to destroy ephemeral infrastructure.

###Jump Box

An ephemeral EC2 instance that serves as an intermediary between the local backup VM and S3 backup bucket via the endpoint.



##Network Services:

###Active Directory

Windows Server VM acts as the Active Directory Server.

Handles authentication and policy enforcement for workstations.

Complete automated and remote configuration of workstation logging and monitoring services.

###Local Management

Lubuntu VM provides a desktop management experience to OPNsense and other network services.

###DNS and DHCP Services

Alpine Linux VM running dnsmasq.

Functions as both DNS and DHCP server.

DNS server forwards AD-related queries to the AD Server.

Provides DHCP for workstation network configuration.

###Logging and Monitoring Services 

Alpine Linux VM running Rsyslog and Monit.

Functions as both logging and monitoring server.

###Backup Services

Lubuntu VM handles backup data collection and cloud backup.

Workflow:

	Cron triggers backup script ->

	Backup data collection scripts ran ->

	Lambda build function triggered->

	New SSH key used to connect to Jump Box ->

	Backup data pushed to Jump Box ->

	Upload script on Jump Box executed ->

	Lambda cleanup function triggered.


