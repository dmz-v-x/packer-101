## Introduction to Packer

### 1. The Core Problem Packer Solves

Imagine this situation.

You have:

- A server such as a VM or EC2
- An operating system installed like Ubuntu or Amazon Linux

Now you manually do the following:

- Install Docker
- Install Node.js
- Configure users
- Set permissions
- Tune configuration files

You repeat this:

- On another server
- On another environment
- On another day

Problems with this approach:

- Every server ends up slightly different
- Humans forget steps
- Bugs appear randomly
- You hear “works on my machine”
- Scaling becomes painful and slow

This approach is called **mutable infrastructure**.

Mutable infrastructure means you keep changing servers after they are created.

---

### 2. The Big Idea — Immutable Infrastructure

Packer exists to support **immutable infrastructure**.

Immutable means:

- You build everything once
- You bake it into an image
- You never change the server after it is launched

Instead of this flow:

- Create server → configure it

You follow this flow:

- Configure → create image → launch servers from that image

Think of it like a baking analogy.

- Mutable infrastructure is like cooking after the food is served
- Immutable infrastructure is like baking the cake fully and then serving identical slices

Every server is the same because it comes from the same image.

---

### 3. What Packer Actually Does

Packer is an **image automation tool**.

It creates machine images such as:

- AWS AMIs
- Docker images
- Azure images
- GCP images
- Local virtual machine images

It does this using:

- A single configuration file
- Repeatable steps
- Version controlled templates

Important clarification:

- Packer does not deploy servers
- Packer only creates images

Deployment is handled by tools like Terraform, which will be connected later.

---

### 4. Packer’s Position in Real Systems

In real companies, the workflow usually looks like this:

- Packer creates the image such as an AMI
- Terraform creates infrastructure using that image
- CI/CD automates the entire process

Packer is used:

- Before production
- Before scaling
- Before deployments

This is why Packer is critical in real world environments.

---

### 5. What Packer Needs to Do Its Job

To create an image, Packer needs a few things.

A base image, for example:

- Ubuntu
- Amazon Linux

Instructions, such as:

- Install packages
- Copy files
- Run scripts

A target platform, such as:

- Local virtual machines
- AWS EC2
- Docker

All of this information lives inside a **Packer template**.

---

### 6. Modern Packer

Modern Packer uses:

- HCL which is HashiCorp Configuration Language
- A plugin based architecture
- packer init which is mandatory
- Versioned builders

Old JSON templates are deprecated.

They should be avoided completely.

We will not use them.

We will only use files with the extension:

- .pkr.hcl

---
