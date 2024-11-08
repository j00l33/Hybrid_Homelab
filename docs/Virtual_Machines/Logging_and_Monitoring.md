The logging and monitoring service will both run on a single VM. Similar to the DNS/DHCP server, we are using a version of alpine for virtualization. The VM enables two seperate servies, a logging service and a monitoring service.

##Setup

	-Configured a VM on Proxmox with Alpine

	-Setup Alpine and configured a static IP

	-Installed Rsyslog, Monit, logrotate, and jq

	-Configured Rsyslog, Monit, and logrotate

	-Used OpenRC to configure automatic use of Rsyslog and Monit

	-Verified cron job was created for logrotate

	-Enabled SSH for remote management and backup access 

##Rsyslog configuration

UDP and TCP log traffic is permitted.

Logs are sorted into files by hostname and application.

##logrotate configuration
	
Logging, Windows Monitoring, and Monit data are rotated Daily.

Keeps a weeks worth of logs at a time, meaning that seven, single-day logs exist at a time.

Log files are compressed after being rotated.

Automatically create a new log file after each rotation.

##Monit configuration

There are two separate Monit solution approaches used, one for Windows machines and one for Linux machines. Both approaches eventually converge into a single pane monitoring solution.

###Windows Approach:

The windows machines are configured with custom PowerShell scripts to collect system data and regularly send the data to the Monit server. 

To receive this data, the Rsyslog system is used and its configuration has been modified. Whenever data is received, it is filtered for the application. 

If the application matches the Monit related keyword, the 'logs', which are really just Rsyslog formatted messages encapsulating a JSON payload, are sent to a separate directory.

This data then has to be extracted, filtered, and checks are made to determine what to output to status files. 

These checks include CPU usage, available memory, free disk space, DNS and Directory Services, and spikes in failed login attempts.

A script takes care of this process, and is automatically ran via cron more frequently than the rate at which new data comes in. 

###Linux Approach

The linux machines are configured to run Monit processes locally.

The local Monit processes watch over local system services and resources, and attempt service restarts when needed.

The local Monit processes create a web interface that is accessible with a username/password.

The centralized monitoring VM uses a script to authenticate to the web interfaces, scrape the system status data, and write the results to individual system status files. This script is ran every 60 seconds via cron. 

###Converged Solution

Monit is configured to check each of these status files, for both the windows and linux machines. If there's a problem, then another script is executed which sends an alert message to a webhook. The statuses are all displayed via the centralized Monit process which hosts a webpage acting as a single pane monitoring window.
