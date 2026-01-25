
# Terraform Notes – Part 2

## 1. Terraform State

Terraform uses a **state file** to keep track of the infrastructure it manages.  
The state file acts as a **mapping between Terraform configuration and real-world resources**.

By default, Terraform stores state in a file named:

```text
terraform.tfstate
````

The state file contains:

* Resource IDs
* Resource attributes
* Metadata
* Dependency information

---

## 2. Why Terraform State is Important

Terraform **cannot function properly without state**.

State is required to:

* Track which resources Terraform manages
* Determine what changes are needed
* Improve performance
* Handle dependencies between resources

State works like a **blueprint of the real infrastructure**.

---

## 3. Terraform State Lifecycle

* `terraform init`
  Initializes backend and installs providers

* `terraform plan`
  Compares desired state with current state

* `terraform apply`
  Updates infrastructure and state file

Even if infrastructure already exists, Terraform still relies on state to understand resource ownership.

---

## 4. Terraform State Locking

State locking prevents multiple users from modifying the same state file at the same time.

* Locking happens automatically
* If locking fails, Terraform stops execution
* Locking ensures consistency and safety

### Manual Unlock

```bash
terraform force-unlock LOCK_ID
```

State locking should not be disabled in production environments.

---

## 5. Sensitive Data in Terraform State

Terraform state may store sensitive information such as:

* Database passwords
* API keys
* Secrets

### Marking Output as Sensitive

```hcl
output "db_password" {
  value     = aws_db_instance.db.password
  sensitive = true
}
```

Marking an output as sensitive hides it from CLI output, but **the value still exists in the state file**.

---

## 6. Terraform Backends

A backend defines:

* Where state is stored
* How state is loaded
* How operations are executed

Terraform must initialize the backend before use.

---

## 7. Local Backend

The **local backend** is the default backend.

### Characteristics

* Stores state on local filesystem
* Uses OS-level locking
* Suitable for individual use

### Example

```hcl
terraform {
  backend "local" {
    path = "terraform.tfstate"
  }
}
```

Local backend is not recommended for team environments.

---

## 8. Remote Backend

A **remote backend** stores Terraform state in a shared location.

### Benefits

* Team collaboration
* Centralized state
* State locking
* Better security

Common remote backends:

* AWS S3
* Terraform Cloud
* Consul

Terraform allows **partial backend configuration** to avoid storing secrets in code.

---

## 9. Terraform with AWS – Prerequisites

Before using Terraform with AWS:

* AWS CLI installed
* IAM user created
* Access key and secret key available

### Export Credentials

```bash
export AWS_ACCESS_KEY_ID=<access_key>
export AWS_SECRET_ACCESS_KEY=<secret_key>
```

Terraform uses these credentials to authenticate with AWS APIs.

---

## 10. AWS Provider Configuration

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }
  required_version = ">= 1.2.0"
}

provider "aws" {
  region = "us-east-1"
}
```

---

## 11. Provisioning an EC2 Instance

```hcl
resource "aws_instance" "ec2_test" {
  ami           = "ami-08c40ec9ead489470"
  instance_type = "t2.micro"

  tags = {
    Name = "TerraformTestInstance"
  }
}
```

### Output Public IP

```hcl
output "instance_public_ip" {
  value = aws_instance.ec2_test.public_ip
}
```

---

## 12. Terraform State Commands

```bash
terraform state list
terraform state show <resource>
terraform state mv
terraform state rm
terraform state pull
terraform state push
```

These commands help manage Terraform state without modifying infrastructure.

---

## 13. Terraform Meta-Arguments

Meta-arguments modify resource behavior.

### count

```hcl
resource "aws_instance" "server" {
  count = 3
  ami   = var.ami
  instance_type = "t2.micro"
}
```

### for_each

```hcl
resource "aws_instance" "server" {
  for_each = var.ami_map
  ami      = each.value
  instance_type = "t2.micro"
}
```

### depends_on

```hcl
depends_on = [aws_s3_bucket.bucket]
```

---

## 14. Terraform Modules

A module is a **reusable collection of Terraform files**.

### Module Structure

```text
module/
 ├── main.tf
 ├── variables.tf
 └── outputs.tf
```

### Calling a Module

```hcl
module "ec2" {
  source   = "./ec2-module"
  image_id = "ami-xxxx"
}
```

Modules improve:

* Code reuse
* Maintainability
* Scalability

---

## 15. Module Inputs and Outputs

### Input Variable

```hcl
variable "image_id" {
  type = string
}
```

### Output Value

```hcl
output "private_ip" {
  value = aws_instance.server.private_ip
}
```

---

## 16. Terraform Functions

Terraform provides built-in functions for data manipulation.

Examples:

```hcl
max(5, 10, 15)
element(["a", "b", "c"], 1)
lookup({a="one", b="two"}, "c", "default")
```

Terraform does not support user-defined functions.

---

## 17. Provisioners in Terraform

Provisioners are used to execute scripts or commands on resources.

Provisioners should be used **only as a last resort**.

---

## 18. Types of Provisioners

### local-exec

```hcl
provisioner "local-exec" {
  command = "echo Hello Terraform"
}
```

### file

```hcl
provisioner "file" {
  source      = "app.conf"
  destination = "/etc/app.conf"
}
```

### remote-exec

```hcl
provisioner "remote-exec" {
  inline = [
    "sudo apt update",
    "sudo apt install -y nginx"
  ]
}
```

---

## 19. Provisioner Execution Behavior

* Provisioners run during resource creation
* If a provisioner fails, the resource is marked as **tainted**
* Tainted resources are destroyed and recreated on next apply

### Destroy-Time Provisioner

```hcl
provisioner "local-exec" {
  when    = destroy
  command = "echo Resource destroyed"
}
```

---

## 20. Debugging Terraform

Terraform provides detailed logs for debugging.

### Enable Debug Logs

```bash
export TF_LOG=TRACE
export TF_LOG_PATH=terraform.log
```

Log levels:

* TRACE
* DEBUG
* INFO
* WARN
* ERROR

---


