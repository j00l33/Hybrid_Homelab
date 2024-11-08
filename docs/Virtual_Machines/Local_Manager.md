Lubuntu was selected to provide a lightweight desktop experience for managing the OPNsense VM and other network services. This allows me to use a web browser to manage OPNsense which provides a far better experience than the console.

##Setup

	-Downloaded the ISO on local PC and uploaded it to Proxmox local storage

	-Configured a new VM on Proxmox with the ISO

	-Linked the VM to the default lanBridge interface

	-Configured OPNsense VLAN interfaces

	-Changed interfaces to the VLAN specific interface 

	-Switched the IP settings to the Local Management VLAN and assigned
	 this VM a static IP

##Logging

The logging solution on Lubuntu mirrors that of the DHCP/DNS VM. The only difference being that Rsyslog is managed via systemctl on Lubuntu instead of OpenRC on alpine.

Rsyslog is configured locally and creates log data. Rsyslog is configured to run automatically.

The log data is then sent to the centralized logging server.

The log server uses Rsyslog to receive logs and sort them by host and program.

Log rotation is enabled on the server to ensure rotation, compression, and eventual deletion of log data.

##Monitoring:

The monitoring solution on Lubuntu also mirrors that of the DHCP/DNS VM. There are two primary differences. Here, the Monit client will not attempt to restart any system services, and it is 
also ran automatically via systemctl.

The local Monit client is configured to watch over the hosts system resources and network connectivity.

The statuses are exposed on the web interface and the central monitoring server can authenticate and access the statuses.

The central monitoring server will actively monitor the web interface.