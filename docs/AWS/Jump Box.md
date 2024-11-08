The Jump Box is the primary piece of ephemeral infrastructure within the backup solution.

##Overview

It serves as an intermediary between the local backup VM and S3.

Creation and Destruction managed by terraform, via lambda functions.

User Data Script used to configure the Jump Box.

Terraform provisions an EC2 instance in which acts as the Jump Box.

Terraform provides the AMI, instance type, public IP, and associated subnet and security group.

The EC2 is given a Custom IAM role which enables it to connect to S3 as well as trigger the cleanup lambda function.

The User Data Script is also provided as part of the terraform configuration.

The User Data Script is used to configure the ec2 to give it the functionality of a Jump Box.

###User Data Script

	Creates backup user.

	Configures SSH with newly generated public key.

	Creates staging directory for backup data.

	Creates script to push uploaded data to S3, through the endpoint.

	Script triggers cleanup lambda function after a time.

	Script given executable permissions for the backup user.

Backup VM will trigger the build function and receive the private SSH key, SSH to the Jump Box, upload the data via the backup user role, and trigger the upload script on the Jump Box. 

The data is uploaded to S3.

The cleanup function is then triggered by either the Jump Box itself or the local backup VM.
