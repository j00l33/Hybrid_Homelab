OPNsense will act as a firewall and router. It will also work with the lanBridge to perform inter VLAN routing. The VM is connected both bridges in the network, lanBridge and wanBridge.


##Setup

	-Downloaded the latest OPNsense ISO to my local machine then
	 uploaded it to the local storage of the node via the web interface.

	-Configured a VM with this ISO and attached the lanBridge and wanBridge
	 virtual hardware to the VM.

	-Assigned the virtual interfaces to the wanBridge and lanBridge.

	-Assigned the LAN interface to a static IP 

	-Assigned the WAN interface a static IP range. The Proxmox nodes internal
	 IP sits on the other side of the interface, which is the default gateway for the OPNsense VM.

	-Set up Local Manager VM

	-Use Manager VM to access OPNsense web interface

	-Configured VLAN interfaces

	-Configured firewall rules to permit inter-VLAN routing, Network Service 
	traffic, and internet access for specified traffic on specific VMs

##Logging

The logging solution for OPNsense uses its built-in logging capabilities to send logs to the server.

Via the web portal, the log settings are configured.

Logs are sent via UDP.

All logs across Applications, Levels, and Facilities are selected.

The logs are sent to the Logging Server with appropriate IP and port number.

The classic BDS syslog format is used.

The Rsyslog on the server is configured to sort by programs in addition to hosts, which OPNsense logs support.

##Monitoring

The monitoring solution for OPNsense uses the built in Monit functionality to enable the web interface. The Monit server uses this web interface to scrape the statuses and add them to the 
centralized alert system and status aggregation page.

Monit is enabled on OPNsense via the web interface.

The Monit client is configured that such the httpd is enabled and made accessible with a username/password, additionally, only a subsection of the internal network IP addresses have access to the Monit web interface.

A script on the monitoring server is configured to monitor and collect the statuses of the web interface. The script writes the processed statuses to local status files which the Monit process on the server will watch. Then Monit will send alerts and update a centralized status page accordingly.	

##Backup

The backup solution for OPNsense involves exporting the current configuration on OPNsense.

SSH is enabled on the interface associated with the VLAN in which the Backup VM lies.

On the Backup Server, SSH keys are generated, and the public key is assigned to a OPNsense user with sufficient privileges.

The Backup Server uses the SCP utility to make a copy of the configuration file locally.