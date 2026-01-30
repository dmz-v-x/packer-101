## Plugins in Packer

### 1. What Plugins Are in Packer

Packer is **not a single monolithic application**.

Instead, Packer works as a **plugin host**.

The core Packer binary is small.  
Almost all real functionality lives inside **plugins**, which are **separate binaries** that Packer loads when needed.

Plugins are what actually do the work.

---

### 2. The Four Major Plugin Categories

Packer plugins fall into **four main categories**.

#### Builders

Builders create machine images.

Examples:

- amazon-ebs → creates AWS AMIs
- docker → builds Docker images
- virtualbox-iso → builds VirtualBox VM images

Builders are responsible for:

- Launching temporary machines
- Running provisioning
- Capturing the final image artifact

---

#### Provisioners

Provisioners configure the temporary machine **after it boots**.

Examples:

- shell
- file
- powershell
- windows-shell

Provisioners are responsible for:

- Installing packages
- Copying files
- Running setup scripts

They do not create images.  
They only modify what goes inside them.

---

#### Post-Processors

Post-processors run **after the image is created**.

Examples:

- manifest
- checksum
- shell-local

Post-processors are responsible for:

- Generating metadata
- Creating checksums
- Uploading artifacts
- Notifying other systems

They never modify the image itself.

---

#### Data Sources

Data sources fetch **external information** during a build.

Examples:

- Fetching latest base AMIs
- Looking up existing images
- Pulling metadata from APIs

They help make templates dynamic without hardcoding values.

---

### 3. How Packer Discovers Plugins

Packer discovers plugins in **two main ways**.

---

### 4. Automatic Plugin Installation with packer init

This is the **modern and recommended approach**.

When your template contains a `packer` block with `required_plugins`, running:

	packer init .

causes Packer to:

- Read the required_plugins block
- Resolve plugin sources
- Download the correct versions
- Store them locally
- Make them available for builds

Example declaration:

	packer {
	  required_plugins {
	    amazon = {
	      version = ">= 1.2.0"
	      source  = "github.com/hashicorp/amazon"
	    }
	  }
	}

After this, Packer can use the amazon builder.

---

### 5. Where Packer Stores Plugins

By default, Packer stores plugins in user-specific directories.

#### Linux and macOS

	$HOME/.config/packer/plugins

#### Windows

	%APPDATA%\packer.d\plugins

These directories contain plugin binaries and checksum files.

---

### 6. Customizing the Plugin Path

You can override the default plugin directory using an environment variable.

	PACKER_PLUGIN_PATH=/custom/path

This is useful in:

- CI systems
- Shared build environments
- Restricted systems

---

### 7. Manual Plugin Installation

In environments where automatic downloads are not allowed, plugins can be installed manually.

To install a plugin from a remote source:

	packer plugins install github.com/hashicorp/my-plugin 1.0.1

To install from a local binary:

	packer plugins install --path ./my-plugin github.com/hashicorp/my-plugin

Packer will:

- Place the binary in the plugin directory
- Generate required checksum files
- Make the plugin usable by templates

---

### 8. Viewing Installed Plugins

To list plugins currently installed:

	packer plugins installed

This shows all plugin binaries Packer can load.

To see which plugins a template requires:

	packer plugins required

This command inspects the configuration and reports missing plugins.

---

### 9. Using External or Third-Party Plugins

Packer supports plugins not maintained by HashiCorp.

These are often community or vendor plugins.

Example:

	packer {
	  required_plugins {
	    cnspec = {
	      version = ">= 12.0.0"
	      source  = "github.com/mondoohq/cnspec"
	    }
	  }
	}

Running `packer init` will install this plugin automatically if it follows Packer’s plugin conventions.

External plugins can add:

- New builders
- New provisioners
- New post-processors
- New data sources

---

### 10. Writing Custom Packer Plugins

When existing plugins are not enough, you can write your own.

#### Key Requirements

- Language: Go
- SDK: packer-plugin-sdk
- Output: standalone binary

Custom plugins communicate with Packer core over RPC.

---

### 11. What You Can Build as a Custom Plugin

Using the SDK, you can implement:

- A custom builder for a new platform
- A custom provisioner for internal tooling
- A custom post-processor
- A custom data source

This allows deep integration of company-specific workflows.

---

### 12. Plugin Lifecycle During a Build

A typical plugin lifecycle looks like this:

	packer build
	→ parse HCL templates
	→ packer init installs plugins
	→ Packer loads plugin binaries
	→ builders run
	→ provisioners run
	→ post-processors run
	→ build completes

Everything beyond parsing is handled by plugins.

---

### 13. Why Packer’s Plugin Model Matters

This architecture makes Packer:

- Extensible
- Lightweight
- Provider-agnostic
- Easy to update independently of core

You can upgrade plugins without upgrading Packer itself.

---

### 14. Final Summary

Plugin type and purpose:

- Builder plugins → create machine images
- Provisioner plugins → configure machines
- Post-processor plugins → act after image creation
- Data source plugins → fetch external data
- External plugins → extend Packer’s capabilities
- Custom plugins → built by you using Go
