This project is created to mimic an IT environment that combines on-premises network services, with a cloud provider to enable backups.

For the on-premises network, virtual machines run software to support different network services. These include Active Directory, DNS, DHCP, routing, firewall, maintenance, logging, monitoring, and backup solutions. 

The virtual machines are interconnected through a local virtualized network. Both the virtual machines and the virtualized network are ran via a hypervisor on my PC. Proxmox is used as the hypervisor for its flexibility and performance.

For the cloud backups, AWS is used. S3 is used to store the backup data. To connect the local backup service to S3, a jump box is used. This jump box is provisioned on an as-needed basis. Additionally, the jump box creation and destruction is managed by lambda functions running terraform. Invoking these lambda functions and connecting to the jump box is all the local backup service has permission to do on AWS.

In order to build out all the network service functionalities, both linux and powershell scripts were relied upon to enable automation and customization, particularly for the logging, monitoring, and backup solutions.

Security considerations were made throughout the process. These include reliance on IAM roles and users on AWS, strict firewall rules for both inter-VLAN and internet traffic, authentication and authorization requirements for data being shared from one VM to another, active directory security related GPO's, and an implementation of a secure and ephemeral jump box for cloud backups.

To bring additional challenge to the project I implemented a zero-cost rule. This means no additional hardware could be purchased or use of AWS outside the bounds of the free tier. The consequences of this requirement pertained primarily towards the initial Proxmox installation and the architecture of the backup solution on AWS.

