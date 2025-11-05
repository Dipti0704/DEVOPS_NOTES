#  Introduction to DevOps

## What is DevOps?
DevOps is a **set of practices, cultural philosophies, and tools** that combine **software development (Dev)** and **IT operations (Ops)**.  
The main goal of DevOps is to **deliver applications and services faster, more efficiently, and more reliably**.

> **Definition:**  
> DevOps is a cultural and technical movement that emphasizes collaboration between development and operations teams to **build, test, and release software rapidly and reliably**.

## Why DevOps?
- Faster delivery of software and updates  
- Continuous improvement and feedback  
- Reduced downtime and better system reliability  
- Automation of manual processes  
- Improved collaboration between teams  
- Quick recovery from failures  

## Core Principles of DevOps
1. **Collaboration & Communication** – Dev and Ops teams work together.  
2. **Automation** – CI/CD pipelines automate testing, deployment, and monitoring.  
3. **Continuous Integration (CI)** – Developers merge code frequently and test automatically.  
4. **Continuous Delivery/Deployment (CD)** – Code changes are automatically deployed to production.  
5. **Monitoring & Feedback** – Real-time insights ensure system health and user satisfaction.  

---

#  Software Development Life Cycle (SDLC)

## SDLC Phases
1. **Requirement Analysis**  
   - Gather and analyze business needs.  
   - Understand the project scope.  

2. **Design**  
   - Create system architecture and design specifications.  

3. **Development (Coding)**  
   - Write code based on design documents.  
   - Follow standards and conventions.  

4. **Testing**  
   - Perform unit, integration, and system testing.  
   - Ensure the software is bug-free.  

5. **Deployment/Release**  
   - Deploy the tested application to production.  
   - Ensure zero downtime deployment (e.g., **Blue-Green** or **Canary Deployment**).  

6. **Monitoring & Maintenance**  
   - Monitor system health and performance.  
   - Handle issues and roll out updates.  

---

#  DevOps Lifecycle (Integration with SDLC)

| Phase | Description | Tools |
|-------|--------------|-------|
| **Plan** | Define goals, requirements, and project scope. | Jira, Trello |
| **Code** | Write the code, manage versions. | Git, GitHub, GitLab |
| **Build** | Compile and package the code. | Maven, Gradle, Jenkins |
| **Test** | Validate code functionality and performance. | JUnit, Selenium |
| **Release** | Deploy builds to environments. | Jenkins, Spinnaker |
| **Deploy** | Push to production with zero downtime. | Kubernetes, Docker |
| **Operate** | Ensure systems run smoothly. | Ansible, Chef, Puppet |
| **Monitor** | Track metrics, logs, and user feedback. | Prometheus, Grafana, ELK |

---

#  DevSecOps (Security in DevOps)

Security should be **integrated at every stage** of the DevOps lifecycle.  
This is known as **Shift-Left Security**, meaning we introduce security checks early in the development process to reduce cost and time.

## Security Tools and Concepts
- **SAST (Static Application Security Testing)**  
  - Scans source code for vulnerabilities before execution.  
  - Example: SonarQube, Checkmarx  

- **DAST (Dynamic Application Security Testing)**  
  - Tests the running application for vulnerabilities.  
  - Example: OWASP ZAP, Burp Suite  

- **SCA (Software Composition Analysis)**  
  - Detects open-source component vulnerabilities and license issues.  
  - Example: JFrog Xray, Black Duck  

- **Linting**  
  - Ensures code quality and consistency (e.g., ESLint, Pylint).  

- **Docker Image Validation**  
  - Checks container images for vulnerabilities before deployment.  

---

#  Infrastructure as Code (IaC)

In DevOps, infrastructure is managed through **code** — known as **Infrastructure as Code (IaC)**.  
It allows developers to **define, provision, and manage** infrastructure using configuration files.

## Popular IaC Tools
- **Terraform** – Manages multi-cloud infrastructure using a declarative configuration language.  
- **AWS CloudFormation** – AWS-specific IaC tool.  
- **Ansible / Chef / Puppet** – Configuration management and automation tools.  
- **Boto3** – AWS SDK for Python to interact with AWS services programmatically.

---

#  AWS VPC (Virtual Private Cloud)

## What is a VPC?
A **Virtual Private Cloud (VPC)** is an **isolated section of the AWS Cloud** where you can launch AWS resources in a **secure and controlled network environment**.

## VPC Components

| Component | Description |
|------------|--------------|
| **VPC** | Your private virtual network in AWS. |
| **Subnet** | A segment of your VPC. Each subnet can host EC2 instances. |
| **Public Subnet** | Has internet access via an **Internet Gateway (IGW)**. |
| **Private Subnet** | No direct internet access (used for databases, internal services). |
| **Internet Gateway (IGW)** | Allows resources in public subnets to communicate with the internet. |
| **NAT Gateway** | Enables private subnets to access the internet securely. |
| **Route Table** | Defines how traffic is directed within and outside the VPC. |
| **Security Groups** | Control inbound and outbound traffic at the **instance (VM)** level. |
| **Network ACLs (NACLs)** | Control traffic at the **subnet** level. |

>  **Security in Layers:**  
> - **Instance Layer:** Security Groups  
> - **Subnet Layer:** Network ACLs  
> - **VPC Layer:** Routing and isolation  

---

#  Example: Simple VPC Setup


**Notes:**
- Instances in the same VPC can communicate with each other by default.  
- Public subnets have routes to the internet via the IGW.  
- Private subnets connect through the NAT Gateway for outbound internet access.  

---

#  Artifacts and Deployment Strategies

## Artifact Repositories
- Store build outputs (e.g., JARs, Docker images).  
- **Example:** JFrog Artifactory, Nexus Repository.  

## Deployment Strategies
- **Blue-Green Deployment:** Two environments — one active (Blue), one idle (Green). Switch traffic after testing.  
- **Canary Deployment:** Gradual rollout to a small percentage of users before full release.  
- **Rolling Deployment:** Updates instances one at a time to ensure uptime.  

---

