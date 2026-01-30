## Packer HCL Template

### 1. What Is a Packer Template

A Packer template is a configuration written in HCL that tells Packer exactly how to build a machine image.

A template answers four core questions:

- What image should be created
- Where the image should be built
- How the image should be configured
- Which values should be reusable or configurable

Packer reads all files ending with:

.pkr.hcl

When you run a build, Packer loads every `.pkr.hcl` file in the directory and treats them as one combined template.

---

### 2. How Packer Templates Are Structured

Packer templates are made of **HCL blocks**.

Each block has a single responsibility and clearly tells Packer what to do at each stage of the build.

The main blocks you will use are:

- packer
- variable
- locals
- source
- build
- provisioner
- post-processor

Each block plays a role in the full image build lifecycle.

---

### 3. packer Block — Meta Configuration

The `packer` block is used for template-level configuration.

Its most important job is declaring **required plugins**.

Modern Packer uses a plugin-based architecture, so builders and provisioners are loaded as plugins.

Example:

    packer {
      required_plugins {
        amazon = {
          version = ">= 1.1.1"
          source  = "github.com/hashicorp/amazon"
        }
      }
    }

What this block does:

- Tells Packer which plugins are required
- Locks plugin versions for reproducibility
- Allows `packer init` to download everything needed

Without this block, Packer would not know how to build images for AWS or other platforms.

---

### 4. variable Block — Input Variables

Variables make templates reusable and configurable.

Instead of hardcoding values, you define them as inputs.

Example:

    variable "region" {
      type    = string
      default = "us-east-1"
    }

Variables can be overridden using:

- Command line flags
- Variable files
- Environment variables

Example override:

    packer build -var="region=us-west-2"

Why variables matter:

- Same template works for multiple environments
- No need to edit files for small changes
- Safer and cleaner configuration

---

### 5. locals Block — Internal Computed Values

The `locals` block defines values calculated inside the template.

These values:

- Are computed using expressions or functions
- Cannot be overridden externally
- Are used to avoid repetition

Example:

    locals {
      timestamp = regex_replace(timestamp(), "[- TZ:]", "")
    }

Common uses of locals:

- Creating unique image names
- Combining variables
- Centralizing repeated logic

Think of locals as helper values only visible inside the template.

---

### 6. source Block — Define the Builder

The `source` block defines **where and how** Packer builds the image.

Every image build starts from a source.

Example for AWS:

    source "amazon-ebs" "example" {
      ami_name      = "packer-example-${local.timestamp}"
      instance_type = "t2.micro"
      region        = var.region
      source_ami    = "ami-12345678"
      ssh_username  = "ec2-user"
    }

What this block does:

- Launches a temporary machine
- Uses a base image
- Defines region, instance type, and access details

Important concepts:

- The machine created here is temporary
- It exists only during the build
- It is later converted into the final image

---

### 7. build Block — Tie Everything Together

The `build` block connects sources with configuration steps.

It defines:

- Which source blocks to use
- Which provisioners to run
- Optional post-processing steps

Example:

    build {
      sources = ["source.amazon-ebs.example"]

      provisioner "shell" {
        inline = ["echo Hello World"]
      }
    }

Without a build block:

- Sources exist but are never executed
- Provisioners never run

The build block is the execution plan of the template.

---

### 8. provisioner Blocks — Configure the Image

Provisioners define how the temporary machine is configured.

They run **inside** the temporary instance created by the source block.

Example:

    provisioner "shell" {
      inline = [
        "sudo apt update",
        "sudo apt install -y nginx"
      ]
    }

Provisioners can:

- Install software
- Copy files
- Create users
- Configure services

Key rules:

- Provisioners must be inside a build block
- They run in order
- If one fails, the build stops

Whatever state the machine is in after provisioning becomes the final image.

---

### 9. post-processor Blocks — After the Image Is Built

Post-processors run after the image is created.

They do not modify the machine.

They work on the **final artifact**.

Example:

    post-processor "checksum" {
      checksum_types = ["md5", "sha256"]
    }

Common uses:

- Tagging images
- Generating manifests
- Exporting metadata
- Uploading artifacts

Post-processors are optional but useful in CI and production workflows.

---

### 10. How Packer Loads Multiple Template Files

Packer automatically loads all `.pkr.hcl` files in a directory.

Example structure:

    .
    ├── variables.pkr.hcl
    ├── sources.pkr.hcl
    ├── build.pkr.hcl

When you run:

    packer init .
    packer build .

Packer merges all files into a single logical template.

This allows clean separation of concerns without extra configuration.

---

### 11. Complete Simple Template Example

    variable "region" {
      type    = string
      default = "us-east-1"
    }

    locals {
      timestamp = regex_replace(timestamp(), "[- TZ:]", "")
    }

    source "amazon-ebs" "app" {
      ami_name      = "app-${local.timestamp}"
      instance_type = "t2.micro"
      region        = var.region
      source_ami    = "ami-0abcdef1234567890"
      ssh_username  = "ec2-user"
    }

    build {
      sources = ["source.amazon-ebs.app"]

      provisioner "shell" {
        inline = ["echo Hello from Packer"]
      }
    }

This template:

- Declares a variable
- Computes a timestamp
- Launches a temporary EC2 instance
- Configures it
- Produces an AMI

---

### 12. Final Recap — Template Blocks

Block name and purpose:

- packer: declare required plugins
- variable: define configurable inputs
- locals: define internal computed values
- source: define where and how the image is built
- build: connect sources and execution steps
- provisioner: configure the machine
- post-processor: act on the final image

