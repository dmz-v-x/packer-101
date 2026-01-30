## Packer Provisioners

### 1. What Provisioners Are

Provisioners are plugins that **run inside the temporary machine** created by a builder.

Their job is to **configure that machine before it becomes the final image**.

Builders handle image creation.  
Provisioners handle customization inside the machine.

Typical tasks done by provisioners:

- Install operating system packages
- Install application runtimes
- Copy files into the machine
- Run setup scripts
- Configure users, permissions, and services

Provisioners never create images on their own.  
They always run **after the builder starts the machine and before the image is finalized**.

---

### 2. Where Provisioners Run

Provisioners run in one of two places, depending on the type:

- Inside the temporary machine created by the builder
- On your local machine running Packer

Most provisioners run **inside the temporary machine**.

This distinction is very important for understanding what they can and cannot do.

---

### 3. Built-in Provisioners in Packer

According to the latest official Packer documentation, these provisioners are built in and supported.

breakpoint  
Pauses the build until you manually confirm. Used mainly for debugging and inspection.

file  
Uploads files or directories from your local system into the temporary machine.

shell  
Runs shell commands or scripts inside Linux or Unix-based images.

shell-local  
Runs commands on the local machine where Packer is executed.

powershell  
Runs PowerShell commands or scripts inside Windows images.

windows-shell  
Runs Windows CMD commands inside Windows images.

windows-restart  
Triggers a reboot during Windows image provisioning.

hcp-sbom  
Uploads a software bill of materials to the HCP Packer registry.

All of these provisioners run **after the machine is booted** and **before the image is created**.

---

### 4. How Provisioners Fit Into a Template

Provisioners are always defined inside a `build` block.

Basic structure:

    build {
      sources = ["source.amazon-ebs.example"]

      provisioner "shell" {
        inline = ["echo Hello World"]
      }
    }

The flow is always:

- Builder creates the machine
- Provisioners configure the machine
- Builder captures the final image

Provisioners never exist outside a build block.

---

### 5. The shell Provisioner

The shell provisioner is the most commonly used one.

It runs shell commands directly inside the temporary machine.

Example:

    provisioner "shell" {
      inline = [
        "sudo apt update",
        "sudo apt install -y nginx"
      ]
    }

Typical uses:

- Installing packages
- Enabling services
- Running setup scripts
- Cleaning up temporary files

If a shell command fails, the entire build fails.

---

### 6. The file Provisioner

The file provisioner uploads files or directories from your local system into the temporary machine.

Example:

    provisioner "file" {
      source      = "app.tar.gz"
      destination = "/tmp/app.tar.gz"
    }

Key points:

- It only copies files
- It does not execute them
- It is often paired with a shell provisioner

A common pattern is upload first, then execute.

---

### 7. The shell-local Provisioner

The shell-local provisioner runs commands **on your local machine**, not inside the image.

Example:

    provisioner "shell-local" {
      inline = ["echo Build started at $(date)"]
    }

Common uses:

- Logging build progress
- Generating files before uploading
- Running validation scripts
- Sending notifications

This provisioner never touches the image directly.

---

### 8. Windows Provisioners

When building Windows images, Linux shell provisioners are not usable.

Instead, Packer provides Windows-specific provisioners.

powershell  
Runs PowerShell scripts inside the Windows machine.

windows-shell  
Runs Windows CMD commands.

windows-restart  
Reboots the machine during provisioning when required.

These provisioners are only used when the builder targets Windows-based images.

---

### 9. Provisioner Execution Order

Provisioners run **in the exact order they are declared** inside the build block.

Example:

    provisioner "file" { ... }
    provisioner "shell" { ... }
    provisioner "shell-local" { ... }

Execution order:

1. file provisioner runs
2. shell provisioner runs
3. shell-local provisioner runs

If any provisioner fails, the build stops immediately.

---

### 10. Using Variables and Locals in Provisioners

Provisioners can reference input variables and locals.

Example:

    provisioner "shell" {
      inline = [
        "echo Installing app version ${var.app_version}"
      ]
    }

This allows:

- Environment-specific configuration
- Versioned installs
- Reusable templates

Provisioners fully support Packer’s variable system.

---

### 11. When Provisioners Are Used

Provisioners are typically used when:

- You need to install system packages
- You need to configure services
- You must copy application code
- You need custom setup logic
- You are preparing a machine image for production

In Packer, provisioners are the **primary mechanism** for preparing images.

---

### 12. Important Behavioral Rules

Key things to always remember:

- Provisioners run inside the temporary machine unless explicitly local
- They run after the builder boots the machine
- They run before the image is captured
- They modify what goes into the final image
- They are executed sequentially

Provisioners do not deploy applications.  
They prepare images.

---

### 13. Example Using Multiple Provisioners

    build {
      sources = ["source.amazon-ebs.example"]

      provisioner "file" {
        source      = "setup.sh"
        destination = "/tmp/setup.sh"
      }

      provisioner "shell" {
        inline = [
          "chmod +x /tmp/setup.sh",
          "sudo /tmp/setup.sh"
        ]
      }

      provisioner "shell-local" {
        inline = ["echo Provisioning completed"]
      }
    }

What happens here:

- A script is uploaded into the machine
- The script is executed inside the machine
- A message is logged locally

This is a very common real-world pattern.

---

### 14. Final Recap

Provisioner type and purpose:

- shell: configure software inside the image
- file: upload files into the image
- shell-local: run commands on the build machine
- Windows provisioners: configure Windows images
- breakpoint: pause the build for debugging

Provisioners are how you **shape the inside of your image**.

Builders create the image.  
Provisioners define what lives inside it.
