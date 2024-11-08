The cloud backup solution is comprised of both permanent and ephemeral infrastructure. 
All the infrastructure falls within free use limits on AWS. 
All permanent infrastructure is created manually, whereas all ephemeral infrastructure is managed via terraform within lambda functions.
The permanent infrastructure is as follows.

##VPC

The VPC is given a name, an IPv4 CIDR block, and no IPv6 CIDR block.

###Internet Gateway

The Internet Gateway acts as a virtual router to the internet for the VPC. 
Created independently of the VPC but then attached to it.

###Subnets

Subnets are subdivisions of the VPC network space and have their own range of IP addresses. Both of the subnets are placed in the same availability zones for this project.

####Public Subnet

	The public subnet is named and made accessible to the outside internet. It has an IPv4 subnet block of 10.0.1.0/24

	Associated with the public route table.

####Private Subnet

	The private subnet is named and is not accessible to the outside internet. It has an IPv4 subnet block of 10.0.2.0/24

	Associated with the private route table.

###Route Tables

The route tables contain routes for traffic within a subnet. There is a route table for each subnet.

####Public Route Table
	
	Associated with the public subnet. 
	
	Provides routing to the Internet Gateway. 
	
	Enables SSH access.

####Private Route Table

	Associated with the private subnet.

	No internet access.

	Routes to an S3 endpoint.

###S3 Endpoint

An S3 Gateway Endpoint is needed for S3 access to the backup bucket. 

Provides a private AWS network path to S3 from the Jump Box.

Associated with the private route table and subnet.

The endpoint policy is customized to limit its ability to only put and list from the backup bucket.

###Network ACL

Network ACLs operate at the subnet layer to enforce inbound and outbound rules.

Rules are deny by default.

An ACL is applied to the private subnet, which only permits traffic in and outbound from the Jump Box IP and the S3 endpoint.

No rule is needed to allow for the S3 endpoint to communicate with S3 as it is connecting 'behind the scenes' on the AWS internal network to S3.

##S3

S3 provides storage for the actual backup data. In addition it provides a space to put the Terraform state files during an active backup.

Therefore, we will have two S3 Buckets for storage.

Both buckets are in the same region, block all public access, do not have versioning, and use server side encryption.

Each bucket has its own policies which determine how they can be accessed.

###Backup Bucket
	
	Only the Jump Box, through its role, can put data to the bucket.

###State Bucket
	
	Only the Lambda Functions, through their role, can put, get, and delete data within the bucket 

##IAM

IAM allows for exact control over what resources can be used/accessed/modified by whom.

Two roles have been created to handle Lambda and the Jump Box.

A user is created for the local backup VM.

An Instance Profile acts as a wrapper for an IAM role that can be applied to an EC2, in this case, the Jump Box is given the instance profile associated with the Jump Box IAM role. 

###Lambda Role
	
	EC2 Permissions needed to create/view/delete the Jump Box and security groups.
	
	S3 Permissions needed to store/retrieve/clean/view bucket contents.

	IAM Permissions needed to assign Jump Box role to the Jump Box.

###Jump Box Role

	S3 Permissions needed to upload data to the backup bucket.

	Lambda Permissions to trigger cleanup function.

###Backup User

	permissions to call specific Lambda functions given to the user.

##Lambda

Lambda is used to create the ephemeral infrastructure, but the lambda functions themselves are permanent.

There are two Lambda functions, one to provision the Jump Box, and one to clean up the Jump Box.

There are several Lambda Layers, which provide the terraform binary and an assortment of python modules.

###Create Jump Box Function

	Triggered by Backup VM.

	Python runtime environment.

	Handles SSH Key generation and distribution.

	Uses terraform to create and configure Jump Box.

	Uses S3 bucket to store terraform state.

###Clean Up Function

	Triggered by Backup VM or by the Jump Box if a timer runs out.

	Uses state file to destroy ephemeral infrastructure.
