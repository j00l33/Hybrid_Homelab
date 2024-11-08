The Active Directory Server acts as a domain controller for the workstation and provides configuration and management to any windows workstations within the network.

Setup:

	-Downloaded the Windows Server and VirtIO driver ISOs locally and uploaded
	 them to Proxmox

	-Created a VM on Proxmox with an additional drive for the VirtIO driver

	-Installed Windows Server and initial installation of the driver

	-Installed the driver on the admin account to allow virtual
	 network device detection

	-Configured the static network settings

	-Checked for latest Windows update

	-Selected and installed roles and features via Server Manager

	-Used the Active Directory Domain Services Configuration Wizard to
	 promote the server to a domain controller

	-Verified system services were running as intended thus far

	-Created a OU for workers

	-Created Group Policy Object pertaining to automatic updates for workers

	-Created Group Policy Object pertaining to network discovery protocols and
	 mitigating incorrect network identification for workers

##Logging

NXLog is used as a tool to enable logging to the Logging Sever. 

To enable NXLog, I downloaded and installed NXLog software.

Next, I configured the nxlog.conf file.

According to the NXLog documentation, NXLog configuration files can be configured with these main sections: Constants, Global Directives, Extensions, Input, Output, and Route.

The nxlog.conf file is configured with these sections as follows:		

Constants: Used the default pathing constants

Global Directives: Used the default directives for logging

Extension: Used the default xm_syslog extension

Input:

	-Added an input module to collect Windows Event Logs. 
	-Utilized the im_msvistalog module to query the event logs.
	-Queried logs for security, system, application, directory service, and DNS
	-Explicitly set Hostname

Output: 

	-Added an output module to send the logs to the Logging Server
	-Sends logs via UDP
	-Specify Logging Server IP, Port #, and Log Format
	-Prepend Hostname to each log

Routing:

	-Added an route module to connect the input module to the output module

After testing, the Hostname was not being properly sent in the log traffic and so the Logging Server was improperly naming the log files, hence the hardcoding of the Hostname.

##Monitoring

The monitoring solution includes a PowerShell Script that gathers data, formats the data into a JSON string, and sends the data to the central Monit server. The script is ran every 5 minutes through Task Scheduler. Once received by the monitoring server, the info is analyzed by a script that outputs results to status files. These status files are then observed by the Monit client to monitor the AD Server.

The Rsyslog service is taken advantage of to send system data to the central Monit server. Because of this configuration, the JSON payload is wrapped in a Rsyslog message format.

The PowerShell Script consists of 5 functions. Three functions to collect data, one to create the JSON payload, and one more to format the message and send the it over UDP.

The functions are as follows:

Get-System-Resources:

	Collects CPU use, available memory, and remaining disk space.

	Uses the Get-WmiObject cmdlet to pull Windows Management Instrumentation objects,
	the Win32_Processor, Win32_OperatingSystem, and Win32_LogicalDisk classes are
	used for the CPU, memory, and disk space respectively.
	
	The three get call outputs are piped to additional processing commands.
	
	For the CPU, the Measure-Object cmdlet is to group the load percentages,
	on each core. The average property is then selected for, returning a relative
	CPU use percentage. 
	
	For the available memory and disk space, the Select-Object is used to create
	calculated properties for available memory and free space. An expression
	is used to pull the relevant variables from the current object, converts to GB,
	and rounds to two decimal places. 
	
	The three variables are then returned as structured data.

Get-Services-Status:

	Collects the statuses of NTDS and DNS, converts them to strings,
	and returns them as structured data.

Get-Failed-Logins:

	Calculates the amount of failed logins within the last 5 minutes.

	The time 5 minutes ago is calculated.

	The Get-WinEvent queries Windows Event Log.

	The specific failed attempted login data is filtered for, in addition
	the start time is filtered for using the calculated time.

	The command is chained into Measure-Object and Select-Object to
	determine the quantity of filtered entries.

	This single integer is returned.

Create-JSON-Payload:

	The three Get functions are called and the structured data is collected.

	The structured data is then restructured into the final format,
	with the addition of a hostname and timestamp.

	The data is then converted into the payload with the ConvertTo-Json built-in

Send-UDP-Payload:

	The payload is provided to the function as a parameter.

	A rsyslog message is formed with a timestamp, hostname,
	application name, and JSON payload.

	The application name will be used on the monit server to
	sort the data appropriately.

	The Function uses the UdpClient .NET class to create a client
	to send UDP packets.

	The payload is converted to a byte array.

	The client is used to send the packet to the Monit server IP and specified port.

	The client is closed.

##Backup:

The backup solution for the domain controller involves creating a shared folder and storing data in the folder to be pulled by the backup server.

A service account was created for the backup server.

A File Share was created in which the backup server can access.

A PowerShell script updates the data within the folder, and removes old data.

The script wipes the folder and inputs new AD Configuration data regarding users, groups, OUs, Domain info, and GPO's.

Task Scheduler is used to run the script automatically every hour.