The workstation models an individual PC in a local netowrk. While there is only a single workstation in the virtual netowrk. It has been configured such that it is capable of being automated replicated and configured.

##Setup

	-Downloaded the Windows 10 Evaluation ISO and uploaded it to Proxmox

	-Created a VM that included the Windows ISO and the VirtIO Drivers ISO

	-Installation of Windows onto the virtualized hard drive utilizing the
	 VirtIO driver

	-Created a default user and installed the VirtIO driver to connect
	 to the network

	-Workstation successfully utilized the DHCP server and was given an
	 IP and domain info

	-Renamed the PC and joined the domain using admin credentials

	-Created a user and moved the PC into the proper Organizational Unit
	 on the AD Server

	-Logged into PC using new user credentials

	-Verified connection to the Domain Controller and proper
	 application of Group Policies

##Logging

The logging solution is similar to that of the AD-Server. Like the monitoring solution on this machine, it is configured automatically via Group Policy.

Logging relies on NXLog to send logs to the logging server. Group policy is used to share files, install and configure NXLog, and to run it on startup.

An NXLog configuration file that mirrors the AD-Sever configuration, without the administrative queries, is placed in a shared folder. Additionally, the NXLog installation software is in 
the shared folder. The shared folder has limited access within the domain.

Next, a Group Policy Object is created which contains policies which enable the following.

The files are coped from the shared folder onto the local storage of the Workstation.

The installation software is assigned for deployment. 

A software service preference is added which starts up the NXLog program upon workstation start up. 

The group policy object is applied to the Workstations OU. 

Once the software is running, it will send logs to the server, and the same server configuration is used to store host specified logs for a time before eventual rotation and deletion.

This setup allows for a streamlined and fully automated approach for configuring logging on new workstations.

##Monitoring

The monitoring solution on the workstations is similar to that of the AD Server. The difference being that instead of manually configuring and scheduling the PowerShell scripts locally, it 
is enforced remotely via Group Policy.

The Script was first created on the AD Server. The format of the script is the same, with the exclusion of certain AD related queries. 

Next, a File Share was created on the AD Server. This share was restricted such that only a subset of domain joined computers had read access to the share.

A Group Policy Object was made which handles the transferring of the PowerShell script from the share onto the Workstation and also handles configuring and deploying a task to the 
Workstation to run the script on a schedule.

After this point, the rest of monitoring solution mirrors the AD Server solution.

The data is sent to the Monit server. The Rsyslog script is configured to output the data to a separate directory. A script extracts and analyzes the data and then preforms the math behind 
the checks and sends the results to status files. This script is ran more frequently via cron than the rate of the data coming in.

Finally, Monit is configured to check the status files and send alerts via a webhook script. 