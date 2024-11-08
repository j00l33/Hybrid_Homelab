This VM gathers data to backup from the other VM's within the network and then preforms the backup operations by utilizing AWS Lambda functions, a Jump Box, and S3. Lubuntu is the OS of choice for a lightweight and straightforward desktop Linux experience.

##Setup

	The VM was created on Proxmox with the Lubuntu ISO.

	Lubuntu was installed and configured with the lanBridge in a specified VLAN.

	The network interface was configured and given a static IP.

##Backup Data Gathering

This VM has to collect data from various VMs within the network. This is achieved in two primary ways. One method is with a shared drive which is maintained by the VM using it. The other method is by setting up SSH on the target VM and extracting specified files from a given VM. 

###Active Directory Server:

A shared drive is set up on the AD Server. A user account is given read access to the folder with a username and password.

On the Backup VM, there is a credentials file, which is used alongside the cifs utility which is used for windows file sharing.

An addition to the fstab configuration that uses the cifs utility and the credentials file, allows for automatic mounting at boot.

A automated script on the AD Server maintains the the data within the shared folder. 

###OPNsense Server:

The backup solution for OPNsense involves exporting the current configuration on OPNsense.

SSH is enabled on the interface associated with the vlan in which the Backup VM lies.

On the Backup Server, SSH keys are generated, and the public key is then assigned to a user with sufficient privileges. 

The Backup Server uses the scp utility to make a copy of the configuration file locally.

###DHCP/DNS Server:

Similar to OPNsense, scp is used to grab specific configuration files.

On the Backup Server, SSH keys are generated and the public key is given to the DHCP/DNS Server.

A script on the Backup Server uses the local SSH Key to grab the configuration files.

The configuration files regarding dnsmasq, monit, and rsyslog are selected for.

###Logging and Monitoring Server:

Likewise to the DHCP/DNS Server, scp is used to grab configuration and log files.

On the Backup Server, SSH keys are generated and the public key is given to the Monitoring/Logging Server.

A script on the Backup Server uses the local SSH Key to grab the configuration and log files.

The configuration files regarding monit and rsyslog are selected for. The latest set of logs are selected for as well.

###Collection Script:

The final step in data collection is running these scripts, collecting the data, and compressing it for backup.

The script executes the individual scripts to get new data. The data from the AD Server related mount, alongside the new data, is collected into a single directory and then compressed 
and stored separately.

##Cloud Backup

A script is used to:

	Run the collection script.

	Trigger the Jump Box creation lambda function and receive SSH key.

	Push the backup file to the Jump Box with SCP.

	Run the backup script on the Jump Box with SSH.

	Trigger the Jump Box cleanup lambda function.

A temporary authentication key is used to authenticate with AWS for CLI use. The authentication key is associated with a limited backup role that allows for lambda functions to be triggered.