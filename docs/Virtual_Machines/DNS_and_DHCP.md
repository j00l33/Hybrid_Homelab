DNS and DHCP services are run on a single VM. The Virtual x86_64 version of Alpine Linux is used as its tailored for use in a virtual environment.

##Setup

	-Downloaded the ISO locally and uploaded it to the Proxmox local storage

	-Set up a new VM and connected it to the lanBridge

	-Set up Alpine and configured its network interface to a static IP

	-Configured the dnsmasq configuration file

	-Used OpenRC to run dnsmasq service on startup

	-Enabled SSH for remote management and backup access 

##dnsmasq configuration

	Forwards AD queries to the AD server.

	Uses google DNS servers.

	Allocates a range for DHCP use for VLAN 20 for Active Directory Workstations.

	Gives the proper gateway to DHCP clients.

	Gives the proper domain name to DHCP clients.

	Enables logging for DNS and DHCP.
	
##Logging
	
The logging solution for the DHCP/DNS server relies on Rsyslog for most of its functionality. 

Rsyslog is configured locally to gather logs regarding the dnsmasq DHCP and DNS processes, as well as host and network interface information.

These logs are collected and forwarded to the Logging Server. Rsyslog is ran on startup via OpenRC.

The Rsyslog client on the server receives the logs and sorts them by host and program accordingly. The logs are rotated, compressed, and eventually deleted.

##Monitoring:

The monitoring solution for the DHCP/DNS server relies on a Monit client running locally.

Monit runs locally and observes the DNS and DHCP processes, host system resources, and network connectivity. Should one the DNS and DHCP processes fail, restarts will be attempted. 

Furthermore, these observed elements are exposed via Monit's built in web server. This server is monitored by the monitoring server via a script. The script writes the observed statuses to 
local status files which are monitored by the Monit process on the server.

Monit on the server is configured to send status alerts via a webhook as well as provide a centralized web page 
for all of the monitored machines.
