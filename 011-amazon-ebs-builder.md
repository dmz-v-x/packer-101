## `amazon-ebs` Builder

### 1. What amazon-ebs Builder Does

The amazon-ebs builder is the official Packer plugin used to create **AWS AMIs**.

This builder automates the full AMI creation lifecycle.

Step by step, what happens internally:

- Packer launches a **temporary EC2 instance** from a base AMI
- Packer connects to the instance, usually via SSH
- Provisioners run inside the instance to configure it
- Packer stops the instance
- Packer creates an EBS snapshot of the root disk
- AWS registers that snapshot as a new AMI
- Packer terminates the temporary instance

The final result is a **fully baked AMI** that can be reused for:

- Production servers
- QA environments
- Auto Scaling Groups
- CI/CD deployments

---

### 2. Prerequisites for Building a Real AWS AMI

Before running a real amazon-ebs build, several real-world requirements must be met.

---

### AWS Credentials

Packer must be able to authenticate with AWS.

The credentials used must have permissions to:

- Launch and terminate EC2 instances
- Create and delete AMIs
- Create and delete EBS snapshots
- Manage security groups
- Use VPCs and subnets
- Create or use key pairs

Credentials can be provided using:

- Environment variables
- AWS credentials file
- IAM role (if running inside AWS)

Example using environment variables on Linux or macOS:

	export AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY"
	export AWS_SECRET_ACCESS_KEY="YOUR_SECRET_KEY"
	export AWS_DEFAULT_REGION="us-east-1"

Packer automatically picks these up.

---

### Base AMI Requirement

amazon-ebs always needs a **starting AMI**.

You can provide this in two ways:

- Hardcode a specific AMI ID
- Use source_ami_filter to dynamically find the latest image

In real-world setups, source_ami_filter is preferred to avoid outdated base images.

---

### 3. Real AWS AMI Template — Complete Example

Below is a **production-style Packer template** written in HCL.

This template builds an Amazon Linux 2023 AMI with Apache installed.

---

### Main Packer Template

	locals {
	  timestamp = regex_replace(timestamp(), "[- TZ:]", "")
	}

	variable "aws_region" {
	  type    = string
	  default = "us-east-1"
	}

	packer {
	  required_plugins {
	    amazon = {
	      version = ">= 1.2.8"
	      source  = "github.com/hashicorp/amazon"
	    }
	  }
	}

	source "amazon-ebs" "al2023" {
	  ami_name      = "custom-al2023-${local.timestamp}"
	  region        = var.aws_region
	  instance_type = "t3.micro"

	  source_ami_filter {
	    filters = {
	      name                = "al2023-ami-*"
	      root-device-type    = "ebs"
	      virtualization-type = "hvm"
	    }
	    owners      = ["amazon"]
	    most_recent = true
	  }

	  ssh_username = "ec2-user"
	}

	build {
	  sources = ["source.amazon-ebs.al2023"]

	  provisioner "shell" {
	    scripts = ["install_webserver.sh"]
	  }

	  post-processor "manifest" {
	    output     = "aws_ami_manifest.json"
	    strip_path = true
	  }
	}

---

### 4. install_webserver.sh — Provisioning Script

Create this file in the same directory as the template.

This script runs **inside the temporary EC2 instance**.

	#!/bin/bash

	sudo yum update -y
	sudo yum install -y httpd
	sudo systemctl enable httpd
	sudo systemctl start httpd

What this script does:

- Updates the OS
- Installs Apache
- Enables Apache to start on boot
- Starts Apache immediately

All of this is baked permanently into the AMI.

---

### 5. Template Breakdown — What Each Section Does

locals  
Generates a timestamp to ensure every AMI name is unique

variable  
Defines the AWS region and allows overrides

packer.required_plugins  
Ensures the amazon-ebs plugin is installed before building

source "amazon-ebs"  
Defines how the temporary EC2 instance is created

source_ami_filter  
Automatically selects the latest official Amazon Linux 2023 AMI

ssh_username  
Defines how Packer connects for provisioning

build.sources  
Connects the source to the build process

provisioner "shell"  
Runs configuration scripts inside the instance

post-processor "manifest"  
Writes build metadata to a JSON file

---

### 6. Running the AMI Build

From the directory containing your files, run:

	packer init .
	packer fmt .
	packer validate .
	packer build .

What each command does:

packer init  
Downloads required plugins

packer fmt  
Formats HCL files consistently

packer validate  
Checks the template for errors

packer build  
Launches the EC2 instance and creates the AMI

---

### 7. What Happens During the Build

Behind the scenes, Packer performs the following:

- Creates a temporary key pair
- Creates a temporary security group
- Launches an EC2 instance
- Waits for SSH to be available
- Runs the provisioning script
- Stops the instance
- Takes an EBS snapshot
- Registers the snapshot as an AMI
- Cleans up temporary resources

All of this happens automatically.

---

### 8. Final Outcome

After a successful build:

- A new AMI appears in your AWS account
- The AMI already has Apache installed
- Any EC2 instance launched from it is immediately ready
- A manifest file records the AMI ID and build details
