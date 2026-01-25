# Terraform Notes – Part 1

## 1. Introduction to Terraform

Terraform is an **Infrastructure as Code (IaC)** tool developed by **HashiCorp**.  
It allows you to define, provision, and manage infrastructure using code instead of manually creating resources through graphical interfaces.

Terraform uses a **declarative approach**, meaning you describe *what* infrastructure you want, and Terraform figures out *how* to create or update it.

Terraform can manage infrastructure across:
- AWS
- Azure
- Google Cloud
- Docker
- Kubernetes
- Many other providers

---

## 2. What is Infrastructure as Code (IaC)?

Infrastructure as Code (IaC) is the practice of managing and provisioning infrastructure through machine-readable definition files rather than manual processes.

### Problems with Traditional Infrastructure Management
- Manual provisioning is slow
- High chance of human errors
- Difficult to reproduce environments
- No version control
- Poor scalability

### Benefits of IaC
- Infrastructure is version controlled
- Consistent and repeatable deployments
- Easy collaboration using Git
- Faster provisioning
- Reduced manual errors

---

## 3. Why Terraform?

Terraform provides several advantages:

- Works with multiple cloud providers
- Human-readable configuration language (HCL)
- Maintains infrastructure state
- Supports automation
- Safe collaboration using version control systems

Terraform manages the **complete lifecycle** of infrastructure:
- Creation
- Modification
- Destruction

---

## 4. Terraform Architecture Overview

Terraform works using the following core components:

### Providers
Providers are plugins that allow Terraform to interact with APIs of cloud platforms and services (AWS, Docker, Azure, etc.).

### Resources
Resources represent infrastructure components such as:
- EC2 instances
- S3 buckets
- Docker containers
- Load balancers

### State
State is a file that maps Terraform configuration to real-world infrastructure.

### Execution Plan
Terraform generates an execution plan before making changes so users can review what will happen.

---

## 5. Terraform Installation (Linux)

### Installation Steps

check the documentation of terraform and you will find all the installation steps

## 6. Terraform Configuration Language (HCL)

Terraform uses **HashiCorp Configuration Language (HCL)**.

HCL is designed to be:

* Easy to read
* Easy to write
* Machine friendly

Terraform configuration files usually have the `.tf` extension.

---

## 7. Blocks and Arguments

Terraform configuration is built using **blocks** and **arguments**.

### Argument

An argument assigns a value to a name.

```hcl
filename = "/home/ubuntu/demo.txt"
```

### Block

A block is a container for arguments and other blocks.

General syntax:

```hcl
block_type "label1" "label2" {
  argument = value
}
```

---

## 8. Resource Blocks

A resource block defines an infrastructure object.

### Syntax

```hcl
resource "<provider>_<resource_type>" "<resource_name>" {
  argument1 = value
  argument2 = value
}
```

### Example

```hcl
resource "local_file" "example" {
  filename = "/tmp/sample.txt"
  content  = "Hello Terraform"
}
```

* `local` → provider
* `file` → resource type
* `example` → resource name

Multiple resources can be defined in the same configuration.

---

## 9. Terraform Execution Workflow

Terraform follows a three-step workflow:

### Step 1: Initialize

```bash
terraform init
```

* Initializes working directory
* Downloads providers
* Sets up backend

### Step 2: Plan

```bash
terraform plan
```

* Creates an execution plan
* Shows what will be created, updated, or destroyed

### Step 3: Apply

```bash
terraform apply
```

* Executes the plan
* Provisions infrastructure
* Updates state file

---

## 10. Terraform with Docker (Example)

### Terraform Block

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 2.21.0"
    }
  }
}
```

### Provider Block

```hcl
provider "docker" {}
```

### Resource Definitions

```hcl
resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx_container" {
  image = docker_image.nginx.latest
  name  = "terraform-nginx"

  ports {
    internal = 80
    external = 80
  }
}
```

---

## 11. Common Terraform Commands

```bash
terraform fmt
terraform validate
terraform show
terraform state list
```

* `fmt` formats Terraform files
* `validate` checks syntax
* `show` displays state or plan
* `state list` shows managed resources

---

## 12. Terraform Variables

Variables allow dynamic configuration.

### Defining Variables (`variables.tf`)

```hcl
variable "filename" {
  default = "/tmp/terraform.txt"
}

variable "content" {
  default = "This content comes from a variable"
}
```

### Using Variables

```hcl
resource "local_file" "file" {
  filename = var.filename
  content  = var.content
}
```

---

## 13. Terraform Data Types

Terraform supports multiple data types:

### String

```hcl
type = string
```

### Number

```hcl
type = number
```

### List

```hcl
type = list(string)
```

### Map

```hcl
type = map(string)
```

### Object

```hcl
type = object({
  name  = string
  items = list(number)
})
```

---

## 14. Terraform Outputs

Outputs are used to display values after `terraform apply`.

### Example

```hcl
output "file_path" {
  value = local_file.file.filename
}
```

Outputs are useful for:

* Debugging
* Sharing values between modules
* Viewing important information like IP addresses

---
