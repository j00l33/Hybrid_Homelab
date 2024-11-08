A backup solution is implemented in order to regularly schedule backups of key configuration and log data to the cloud.

##Workflow

	A local VM is used to gather logs and configuration data from the other
	 machines within the local virtual network. ->

	The backup VM uses a lambda function to create an ephemeral Jump Box. ->

	The Jump Box is securely tunneled to by the local Backup VM and
	 the data is uploaded. ->

	The Jump Box pushes the data to S3 through a private S3 endpoint. ->

	A separate lambda function is used to destroy the Jump Box.

##Backup Solution Rationale

AWS S3 is used for storage because of its flexibility, security, cost, and rich documentation.

In order to connect to S3 from the local VM, I identified three primary options:

	 Directly accessing S3 with AWS credentials from the local VM.

	 Using a Site-to-Site VPN to access an S3 endpoint.

	 Using an SSH tunnel through a Jump Box in order to access the S3 Endpoint.


The direct access was the least secure option, it would've required the VM to have credentials that allowed for direct S3 access.

The Site-to-Site option was more secure, but was costly and this is intended to be a zero-cost project.

Therefore, the Jump Box based solution was selected for it's security and cost benefits. In addition, my Jump Box solution involves creating and running the Jump Box on an as-needed basis, 
which mitigates cost concerns and enhances security.


With this method, the local VM has fewer privileges and cannot directly access S3. The local VM only has the ability to request that the Jump Box be made available to it for a time, in order to 
upload the Backup. Only the Jump Box has the ability to upload to S3, not the VM itself.



This provides multiple benefits such as:

	Limiting access to S3 if the local VMs credentials were to be compromised.

	Creating a time-limited exposure mechanism to S3.

	Ability to SSH tunnel over the public internet.

	Enables security check mechanisms at the Jump Box layer which allows for granular checks of the data to be uploaded.

	Creates a richer audit trail by forcing all S3 access via this backup mechanism.


In order to make the Jump Box secure, we don't allow the local VM to define its configuration.

Instead, the Jump Box is defined and provisioned by a lambda function, this function is not mutable by the local VM, it can only be invoked by the VM.