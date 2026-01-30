## Packer Builders

### 1. What Builders Actually Are

Builders are the components in Packer that **create machine images for a specific platform**.

A builder is a plugin that knows how to:

- Launch a temporary machine such as an EC2 instance
- Run provisioning steps on that machine
- Capture the disk state
- Produce a final artifact such as an AMI, VM image, or Docker image

In simple terms:

Builders are responsible for **creating the image itself**, from start to finish.

They take your template plus provisioners and turn them into something you can actually use in a real environment.

---

### 2. What Builders Are Responsible For

A builder handles the entire lifecycle of image creation.

Step by step, a builder:

1. Creates or launches a temporary machine
2. Makes that machine accessible to Packer
3. Allows provisioners to configure it
4. Stops the machine when configuration is complete
5. Captures the machine’s disk as an image
6. Registers or outputs the final artifact

Provisioners do not create images.  
Builders do.

Provisioners only modify what is **inside** the machine that the builder creates.

---

### 3. How Builders Fit Into a Packer Template

In modern Packer HCL templates, builders are defined using a **source block**.

General structure:

    source "builder-type" "name" {
      # builder-specific configuration
    }

This block defines **how and where** the image will be built.

Later, the source is referenced from a build block:

    build {
      sources = ["source.builder-type.name"]

      provisioner "shell" {
        # configure the image
      }
    }

Important separation of responsibilities:

- Source block defines **how the image is created**
- Provisioners define **what goes inside the image**

---

### 4. Builders Are Plugin-Based

Modern Packer does not hard-code builders.

Instead:

- Builders are implemented as plugins
- Plugins are declared in the packer block
- Plugins are installed using `packer init`

This makes Packer:

- Modular
- Extensible
- Consistent across machines and CI systems

Without plugins, Packer cannot build images.

---

### 5. Types of Builders Supported by Packer

Packer supports many builder types through plugins.

Common categories include:

- Cloud builders
- Local virtualization builders
- Container builders
- Utility builders

Each builder exists to target a specific platform or use case.

---

### 6. Commonly Used Builder Types

Some widely used builder types include:

amazon-ebs  
Builds AWS EC2 AMIs from EBS-backed base images

file  
Creates artifacts directly from files without launching machines

null  
Connects to an existing machine over SSH and runs provisioners without creating an image

Custom builders  
Builders written by teams for internal platforms

Community-supported builders  
Builders maintained by the community for various platforms

Among these, the most commonly used in production environments is **amazon-ebs**.

---

### 7. The amazon-ebs Builder Explained

The amazon-ebs builder is the primary builder used for AWS.

It is maintained by HashiCorp and is designed to create EC2 AMIs.

What amazon-ebs does internally:

1. Launches a temporary EC2 instance from a base AMI
2. Waits for SSH access
3. Runs provisioners to configure the instance
4. Stops the instance
5. Creates an EBS snapshot
6. Registers a new AMI in your AWS account

This process is exactly how teams bake production-ready AMIs.

---

### 8. Real-World Use Cases for Builders

Builders are used for many real-world scenarios.

Common examples:

Create AWS AMIs  
Builder: amazon-ebs

Build Docker images  
Builder: Docker plugin

Prepare VM images for testing  
Builder: virtualbox-iso

Run provisioning on existing servers  
Builder: null

Create custom volume-based images  
Builder: amazon-ebsvolume

Each builder solves a different image creation problem.

---

### 9. Example amazon-ebs Source Block

Below is an example of a builder defined using a source block.

This is only the builder section, not a full template.

    source "amazon-ebs" "example" {
      ami_name      = "packer-latest-image-${local.timestamp}"
      instance_type = "t2.micro"
      region        = var.region

      source_ami_filter {
        filters = {
          name = "amzn2-ami-hvm-*-x86_64-gp2"
        }
        most_recent = true
        owners      = ["amazon"]
      }

      ssh_username = "ec2-user"
    }

Explanation of key parts:

- amazon-ebs tells Packer to build an AWS EC2 AMI
- example is a reference name used later in the build block
- source_ami_filter selects a base AMI automatically
- ssh_username tells Packer how to connect to the instance

Everything inside this block is specific to how the image is created.

---

### 10. How Packer Uses Builders During a Build

When you run:

    packer build .

Packer performs the following steps:

1. Reads all source and build blocks
2. Ensures required builder plugins are installed
3. For each build:
   - Starts the builder
   - Launches the temporary machine
   - Runs provisioners
   - Creates the image artifact
4. Outputs the final image identifier

If the builder fails, no image is produced.

---

### 11. Builders Versus Provisioners

It is critical not to confuse these two.

Builders:

- Create machines
- Control image creation
- Produce artifacts

Provisioners:

- Run inside the machine
- Install software
- Configure the system

Builders create the canvas.  
Provisioners paint on it.

---

### 12. Summary: Builders in Packer

Key takeaways:

- Builders are responsible for creating image artifacts
- They are defined using source blocks
- They are implemented as plugins
- They launch temporary machines and snapshot them
- Most production workflows rely on amazon-ebs for AWS

