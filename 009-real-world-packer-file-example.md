## Real-World Packer Example — Building a Production-Ready AWS AMI

### 1. Project Directory Structure

A real Packer project is usually split into multiple files.

	packer-webserver/
	├── variables.pkr.hcl
	├── webserver.pkr.hcl
	├── install_httpd.sh

Each file has a clear responsibility.

---

### 2. variables.pkr.hcl — Input Variables

This file defines values that can change between environments without touching logic.

	variable "region" {
	  type    = string
	  default = "us-east-1"
	}

	variable "instance_type" {
	  type    = string
	  default = "t2.micro"
	}

	variable "ami_prefix" {
	  type    = string
	  default = "packer-webserver"
	}

What this does:

- region controls where the AMI is built
- instance_type controls the temporary EC2 size
- ami_prefix controls naming consistency

This allows the same template to work for dev, staging, and prod.

---

### 3. install_httpd.sh — Provisioning Script

This script runs **inside the temporary EC2 instance**.

	#!/bin/bash

	# Update the OS
	sudo yum update -y

	# Install Apache web server
	sudo yum install -y httpd

	# Enable and start Apache
	sudo systemctl enable httpd
	sudo systemctl start httpd

	# Create a simple homepage
	echo "<h1>Web Server baked using Packer</h1>" | sudo tee /var/www/html/index.html

Why scripts like this are used:

- Easy to version control
- Easy to reuse
- Easy to debug independently

---

### 4. webserver.pkr.hcl — Main Packer Template

This file ties everything together.

	locals {
	  timestamp = regex_replace(timestamp(), "[- TZ:]", "")
	}

	packer {
	  required_plugins {
	    amazon = {
	      version = ">= 1.2.8"
	      source  = "github.com/hashicorp/amazon"
	    }
	  }
	}

	source "amazon-ebs" "webserver" {
	  ami_name      = "${var.ami_prefix}-${local.timestamp}"
	  instance_type = var.instance_type
	  region        = var.region

	  source_ami_filter {
	    filters = {
	      name                = "al2023-ami-2023.*-x86_64"
	      root-device-type    = "ebs"
	      virtualization-type = "hvm"
	    }
	    most_recent = true
	    owners      = ["amazon"]
	  }

	  ssh_username = "ec2-user"
	}

	build {
	  name    = "ami-webserver"
	  sources = ["source.amazon-ebs.webserver"]

	  provisioner "file" {
	    source      = "install_httpd.sh"
	    destination = "/tmp/install_httpd.sh"
	  }

	  provisioner "shell" {
	    inline = [
	      "chmod +x /tmp/install_httpd.sh",
	      "sudo /tmp/install_httpd.sh"
	    ]
	  }

	  post-processor "manifest" {
	    output = "webserver-manifest.json"
	  }
	}

---

### 5. Step-by-Step What Actually Happens

#### Step 1: packer init

	Packer reads the packer block
	Downloads the amazon builder plugin
	Locks plugin versions

#### Step 2: packer validate

	Packer checks:
	- variables exist
	- types are correct
	- syntax is valid

#### Step 3: packer build

	Packer launches a temporary EC2 instance
	Uses latest Amazon Linux 2023 AMI
	Connects via SSH
	Uploads install_httpd.sh
	Runs the script inside the instance
	Stops the instance
	Creates a new AMI
	Writes webserver-manifest.json

---

### 6. What the Builder Does Here

The amazon-ebs builder:

- Launches a temporary EC2 instance
- Uses EBS-backed storage
- Snapshots the root volume
- Registers a new AMI in your AWS account

The temporary instance is deleted after the build.

---

### 7. What the Provisioners Do Here

file provisioner:

- Copies install_httpd.sh into the instance

shell provisioner:

- Executes the script
- Installs Apache
- Enables the service
- Creates a default webpage

These changes are baked permanently into the AMI.

---

### 8. What the Post-Processor Does

The manifest post-processor:

- Creates webserver-manifest.json
- Stores AMI ID
- Stores build name
- Stores source metadata

This file is extremely useful in CI/CD pipelines.

---

### 9. Final Output After Build

After a successful build:

- A new AMI exists in AWS
- The AMI already has Apache installed
- You can launch EC2 instances instantly
- No manual setup is required
- A manifest file tracks what was built

---

### 10. How This Is Used in Real Companies

Typical real-world flow:

	Packer builds AMI
	CI stores AMI ID
	Terraform launches EC2 using AMI
	Auto Scaling uses the same AMI
	All servers are identical

This is **immutable infrastructure in action**.

---

### 11. Key Takeaway

This is the **exact pattern used in production systems**:

- One template
- Repeatable builds
- Zero manual configuration
- Fully automated image lifecycle
