In order to provision the Jump Box on an as-needed basis, Lambda functions are used to build and destroy Jump Box infrastructure upon request from the local backup VM. 
The Lambda functions use terraform to build and destroy Jump Box infrastructure.
Python 3.11 is used as the runtime environment within the lambda functions.
Lambda Layers are used to import modules and binaries into the Lambda execution environment.

There are five total layers used in the solution, three self-created, and two open source layers.

##Layers

###Self-Created
####Terraform Binary Layer:
	
	Created manually by retrieving the terraform binary that
	 is compatible with the runtime.
	
	Essential to actually execute terraform commands
	 within the functions.

####Paramiko Module Layer:

	Created manually using the .whl file for the module.

	Used by the build function to generate SSH keys that are
	 created based on OpenSSH specifications.

####NACL Module Layer:

	Created manually using the .whl file for the module.

	Dependency for the Paramiko module

###Open Source

####Cryptography Module Layer:

	Dependency for Paramiko module.

	Uses a version that doesn't rely upon rust files, which do not run properly when imported through a Lambda Layer

####BCRYPT Module Layer:

	Dependency for Paramiko module.

####BOTO3 Module Layer:

	Module which enables interaction with various AWS services, in this case, S3.

##Lambda

Two Lambda Functions are used to create and clean the infrastructure

###Build Function

	Python 3.11 runtime.

	Custom IAM role giving specific permissions related to EC2, IAM, and S3. 

	Uses all but the boto3 layer.

	Triggered by the local backup VM with strictly limited user.

	Creates SSH keys via Paramiko module.

	Uses terraform to provision the ec2 which acts as the Jump Box.

	Applies custom Jump Box IAM role giving it S3 access and ability to trigger cleanup function.

	S3 bucket serves as terraform state file storage for use across functions.

	Injects public SSH key into Jump Box.

	User Data feature used to configure Jump Box itself.

	Returns private SSH key to backup VM.

###Clean Functions
	
	Python 3.11 runtime.
	
	Custom IAM role giving specific permissions related to EC2, IAM, and S3.
	
	Uses the terraform binary layer and the boto3 layer.
	
	Triggered by backup VM or by a timer on the Jump Box.
	
	Uses terraform to destroy Jump Box infrastructure.
	
	Retrieves terraform state from S3 bucket.
	
	Clears out S3 bucket upon completion with boto3.
