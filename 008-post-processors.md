## Post Processors

### 1. What Post-Processors Are

Post-processors are plugins that run **after** a builder has finished creating an artifact and **after** all provisioners have completed.

At this stage:

- The image already exists
- The machine is already configured
- The builder’s job is done

Post-processors operate on the **artifact produced by the build**, such as:

- An AMI
- A Docker image
- A VM image file

They do not change what is inside the image.

They handle **everything that needs to happen after image creation**.

---

### 2. What Post-Processors Are Used For

Post-processors take the build artifact and perform follow-up actions, such as:

- Creating checksums
- Generating metadata files
- Uploading artifacts
- Converting image formats
- Pushing artifacts to registries or storage
- Running notifications or automation steps

In simple terms:

Post-processors automate **post-build tasks**.

They make Packer usable in real CI/CD and production workflows.

---

### 3. Where Post-Processors Run in the Build Flow

The complete Packer flow looks like this:

- Builder creates the temporary machine
- Provisioners configure the machine
- Builder produces the final image artifact
- Post-processors operate on that artifact

By the time a post-processor runs, the image is already finalized.

---

### 4. Declaring Post-Processors in HCL Templates

Post-processors are declared **inside a build block**.

You can declare them in two valid ways.

---

### 5. Declaring a Single Post-Processor

You can define a post-processor directly in the build block.

Example:

    build {
      post-processor "checksum" {
        checksum_types      = ["md5", "sha512"]
        keep_input_artifact = true
      }
    }

This runs one post-processor action after the build completes.

---

### 6. Declaring Multiple Post-Processors Using Groups

You can group post-processors using a `post-processors` block.

This is useful when you want to clearly separate multiple post-build actions.

Example:

    build {
      post-processors {
        post-processor "manifest" {}
      }

      post-processors {
        post-processor "shell-local" {
          inline = ["echo Build is done"]
        }
      }
    }

Each `post-processors` block can contain one or more post-processor definitions.

Packer runs all declared post-processors.

---

### 7. Core Concept: Input Artifact

The **input artifact** is the output produced by the builder.

Examples:

- An AMI ID
- A Docker image reference
- A VM disk image

Post-processors always act on this input artifact.

They never re-run provisioning or rebuild the image.

---

### 8. Core Concept: keep_input_artifact

By default, when a post-processor produces a new artifact, Packer **replaces** the original artifact.

If you want to keep both the original and the new artifact, you must explicitly tell Packer.

Example:

    keep_input_artifact = true

This is useful when:

- You want to keep the AMI
- And also generate metadata or checksums
- Or produce multiple outputs from the same build

---

### 9. Controlling When a Post-Processor Runs

You can control **which builds** a post-processor applies to.

This is done using `only` and `except`.

---

### 10. Using only

The `only` option restricts a post-processor to specific sources.

Example:

    post-processor "checksum" {
      only = ["source.amazon-ebs.example"]
    }

This runs the checksum post-processor **only** for the specified source.

---

### 11. Using except

The `except` option skips specific sources.

Example:

    post-processor "checksum" {
      except = ["source.amazon-ebs.test"]
    }

This runs the post-processor for all sources except the listed ones.

---

### 12. Common Built-In Post-Processors

Packer provides several built-in post-processors.

These are officially supported and commonly used.

---

### 13. The manifest Post-Processor

The `manifest` post-processor generates a JSON file containing metadata about the build.

This file includes:

- Artifact IDs
- Build names
- Source information
- Timestamps

Example:

    post-processor "manifest" {
      output     = "packer-manifest.json"
      strip_path = true
    }

This is heavily used in CI/CD pipelines to track what was built.

---

### 14. The shell-local Post-Processor

The `shell-local` post-processor runs commands on the **local machine**, not inside the image.

Example:

    post-processor "shell-local" {
      inline = [
        "echo Build done",
        "echo AMI created"
      ]
    }

Common uses:

- Logging build completion
- Uploading artifacts
- Triggering notifications
- Calling external scripts

Important behavior:

- shell-local does not create a new artifact
- Packer always keeps the input artifact for shell-local

---

### 15. Real-World Use Cases for Post-Processors

Typical automation scenarios include:

- Generating a manifest file for downstream pipelines
- Creating checksums for security verification
- Uploading artifacts to storage systems
- Sending build notifications
- Recording metadata for audits and rollbacks

Post-processors connect Packer builds to the rest of your infrastructure tooling.

---

### 16. What Post-Processors Do Not Do

Post-processors do not:

- Modify the image contents
- Install software
- Change system configuration
- Rerun provisioners

Their scope is strictly **after image creation**.

---

### 17. How Post-Processors Fit Into a Full Template

Typical order inside a Packer template:

- source block defines how to build the image
- build block ties everything together
- provisioners configure the machine
- post-processors handle post-build automation

Each step has a clearly separated responsibility.

---

### 18. Final Recap: Post-Processors at a Glance

Feature and purpose:

- post-processor: define a single post-build action
- post-processors: group multiple post-build actions
- keep_input_artifact: keep original artifacts
- only and except: control where post-processors run
- manifest: generate build metadata
- shell-local: run local automation after build

Post-processors are the final step that turns a built image into a **fully automated, production-ready artifact**.
