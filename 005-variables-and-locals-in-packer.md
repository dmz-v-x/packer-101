## Variables & Locals in Packer

### 1. Two Kinds of Variables in Packer

Packer supports **two kinds of variables**, and understanding the difference is critical.

The two kinds are:

- Input variables
- Local variables

They serve different purposes and behave very differently during a build.

---

### 2. Input Variables

Input variables make your Packer templates **dynamic and reusable**.

You can think of them like function parameters.

They are declared inside the template, but their values can be provided from **outside** the template at build time.

---

### 3. Declaring an Input Variable

Input variables are declared using the `variable` block.

Example declaration:

    variable "region" {
      type        = string
      description = "The AWS region to use"
      default     = "us-east-1"
    }

What each field means:

- type  
  Helps Packer validate the value early and fail fast if the type is wrong.

- description  
  Documents what the variable is for. This is strongly recommended for clarity.

- default  
  Makes the variable optional. If no value is provided externally, this value is used.

If a variable does **not** have a default value, Packer will **require** a value at build time.

---

### 4. How Packer Assigns Values to Input Variables

Packer resolves variable values using a **strict precedence order**.

Higher precedence always overrides lower precedence.

Order from highest to lowest:

1. Command line flags using `-var`
2. Variable definition files passed using `-var-file`
3. Automatically loaded variable files ending with `.auto.pkrvars.hcl`
4. Environment variables prefixed with `PKR_VAR_`
5. Default value defined in the variable block

This order is fixed and defined by Packer.

---

### 5. Command Line Variables

You can override variables directly when running a build.

Example:

    packer build -var="region=us-west-2" .

This has the **highest precedence** and overrides all other sources.

Use this for quick overrides or testing.

---

### 6. Variable Definition Files

A variable definition file contains **only assignments**, not declarations.

Example file contents:

    region  = "ap-south-1"
    app_env = "prod"

You pass it explicitly like this:

    packer build -var-file="myvalues.pkrvars.hcl" .

Important rule:

- Every variable in a `.pkrvars.hcl` file **must already be declared** using a `variable` block
- Otherwise, Packer will fail the build

---

### 7. Auto-loaded Variable Files

If a variable file ends with:

    .auto.pkrvars.hcl

Packer loads it automatically without needing `-var-file`.

Example filename:

    values.auto.pkrvars.hcl

This is commonly used for environment-specific values like dev, staging, or prod.

---

### 8. Environment Variables

Packer can read variables from environment variables.

Format:

    PKR_VAR_<variable_name>

Example:

    export PKR_VAR_region="eu-west-1"
    packer build .

Important rules:

- The environment variable name must start with `PKR_VAR_`
- The variable name part must be uppercase
- Inside the template, you still reference it as `var.region`

This method is commonly used for secrets and CI/CD pipelines.

---

### 9. Local Variables

Local variables are defined using the `locals` block.

They are **internal to the template** and cannot be overridden externally.

Example:

    locals {
      timestamp = regex_replace(timestamp(), "[- TZ:]", "")
      ami_label = "${var.app_name}-${local.timestamp}"
    }

Key characteristics of locals:

- Computed inside the template
- Can reference input variables
- Cannot be set from CLI, files, or environment variables

Think of locals as internal constants or helper values.

---

### 10. Referencing Variables in the Template

Inside any block, you reference:

- Input variables as `var.<name>`
- Local variables as `local.<name>`

Example:

    source "amazon-ebs" "app" {
      region       = var.region
      ami_name     = "${var.app_name}-${local.timestamp}"
      ssh_username = var.ssh_user
    }

This is how templates become reusable across environments.

---

### 11. Variable Types and Validation

Variables can have types beyond strings.

Example list type:

    variable "subnets" {
      type = list(string)
    }

If the value does not match the declared type, Packer fails early with a clear error.

This prevents subtle runtime issues later in the build.

---

### 12. Important Validation Rule

You must **declare a variable before assigning it a value**.

This means:

- Declaring in a `variable` block is mandatory
- Assigning a value without a declaration causes a build failure

This applies to:

- CLI variables
- `.pkrvars.hcl` files
- Environment variables

---

### 13. How Variables Flow in Packer

Value resolution order:

- CLI `-var` flags
- `-var-file` files
- `.auto.pkrvars.hcl` files
- Environment variables using `PKR_VAR_`
- Defaults in the template

Higher levels override lower ones without modifying the template.

---

### 14. Simple End-to-End Example

variables.pkr.hcl

    variable "region" {
      type    = string
      default = "us-east-1"
    }

    variable "app_name" {
      type = string
    }

build.pkr.hcl

    source "amazon-ebs" "app" {
      region   = var.region
      ami_name = "${var.app_name}-${local.timestamp}"
    }

values.auto.pkrvars.hcl

    app_name = "myapp"

Build commands:

    packer init .
    packer validate .
    packer build .

Packer automatically loads `values.auto.pkrvars.hcl` and uses it to supply `app_name`.

---

### 15. Final Takeaway

- Input variables are configurable from outside
- Locals are internal and computed
- Packer uses a strict precedence order
- Templates remain clean, reusable, and environment-agnostic

