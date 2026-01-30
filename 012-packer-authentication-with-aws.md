## How Packer Authenticates with AWS

### 1. Where Packer Finds AWS Credentials

Packer checks credentials in the following places, in order.

---

### Environment Variables

If these environment variables are set, Packer will use them:

	AWS_ACCESS_KEY_ID
	AWS_SECRET_ACCESS_KEY
	AWS_SESSION_TOKEN   (only for temporary credentials)

This method is simple and works well for local testing or CI systems.

---

### AWS Shared Credentials File

Packer can read credentials from the standard AWS credentials file.

Location:

- Linux / macOS  
  ~/.aws/credentials

- Windows  
  %USERPROFILE%\.aws\credentials

Example credentials file entry:

	[packer-profile]
	aws_access_key_id = AKIA...
	aws_secret_access_key = SECRET...

---

### AWS CLI Profiles

You can tell Packer which profile to use by setting:

	AWS_PROFILE=packer-profile

Then run Packer normally:

	packer build .

This is safer than environment variables because credentials stay in one managed place.

---

### IAM Role (When Running Inside AWS)

If Packer runs on:

- An EC2 instance
- AWS CodeBuild
- Any AWS service with an attached IAM role

AWS automatically provides credentials via the instance metadata service.

In this case:

- No access keys are stored
- No environment variables are needed
- Credentials are short-lived and rotated automatically

This is the **recommended production approach**.

---

### 2. Best Practices for Credential Usage

---

### Using Environment Variables (Local or CI)

	export AWS_ACCESS_KEY_ID="AKIA..."
	export AWS_SECRET_ACCESS_KEY="SECRET..."
	export AWS_DEFAULT_REGION="us-east-1"

This approach:

- Works everywhere
- Is easy to configure
- Should be limited to short-lived credentials

Avoid using long-lived keys here in production.

---

### Using AWS Shared Credentials File

Create a dedicated profile for Packer:

	[packer-profile]
	aws_access_key_id = AKIA...
	aws_secret_access_key = SECRET...

Then select it:

	export AWS_PROFILE="packer-profile"
	packer build .

This separates Packer access from your default AWS user.

---

### Using IAM Roles (Best for Production)

Attach an IAM role to:

- EC2 build host
- CodeBuild project
- CI runner inside AWS

AWS automatically injects credentials.

Benefits:

- No secrets stored anywhere
- Automatic rotation
- Fine-grained permissions
- Lowest risk of credential leaks

This is the **industry-standard approach**.

---

### 3. IAM Permissions Packer Requires

To use the amazon-ebs builder, Packer must interact with EC2 and related services.

At minimum, the IAM user or role needs permission to perform the following actions.

---

### Core Required Permissions

- ec2:RunInstances
- ec2:TerminateInstances
- ec2:CreateImage
- ec2:CreateSnapshot
- ec2:Describe*
- ec2:CreateSecurityGroup
- ec2:AuthorizeSecurityGroupIngress
- ec2:CreateKeyPair
- ec2:DeleteKeyPair
- ec2:DeregisterImage
- ec2:ModifyImageAttribute
- iam:PassRole   (only if instance profiles are used)

These permissions allow Packer to:

- Launch the temporary build instance
- Configure networking and access
- Snapshot the disk
- Register and manage AMIs
- Clean up everything afterward

---

### Least-Privilege Considerations

Many example policies use:

	"Resource": "*"

This works, but is not ideal for production.

In real environments, you should restrict:

- EC2 actions to specific regions or VPCs
- Snapshot and AMI ownership to your account
- KeyPair creation to Packer-only resources

You can further tighten access using IAM conditions and resource ARNs.

---

### 4. Using IAM Instance Profiles with amazon-ebs

Sometimes the **temporary EC2 instance itself** needs AWS access during provisioning.

For example:

- Downloading files from S3
- Accessing Systems Manager
- Pulling secrets from a secure store

In this case, you assign an IAM role to the temporary instance.

Example in the source block:

	source "amazon-ebs" "example" {
		iam_instance_profile = "PackerInstanceProfile"
		...
	}

This allows the instance to assume the role securely during provisioning.

No credentials are embedded in scripts or templates.

---

### 5. Real-World Secure Credential Workflow

A common production setup looks like this.

---

### Step 1: Create IAM Role for the Build Host

- Grant minimal permissions required for Packer
- Set trust policy for EC2 or CodeBuild

---

### Step 2: Run Packer on That Host

- No access keys needed
- AWS injects temporary credentials automatically

---

### Step 3: Assign Instance Profile to Builder Instance (Optional)

- Use iam_instance_profile in the source block
- Allow provisioning scripts to access AWS services securely

---

### 6. Final Summary

Best practices at a glance:

- Local development  
  Use AWS shared credentials with named profiles

- CI/CD pipelines  
  Use environment variables or short-lived credentials

- Production builds  
  Use IAM roles attached to build hosts

- Permissions  
  Always follow least-privilege principles
