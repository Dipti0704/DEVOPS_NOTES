#  CI/CD, Testing, and Deployment in DevOps

##  What is CI/CD?

**CI/CD** stands for:
- **CI – Continuous Integration**
- **CD – Continuous Delivery** or **Continuous Deployment**

These are **DevOps practices** that help automate the process of **building, testing, and deploying** software, allowing faster and more reliable releases.

---

##  Continuous Integration (CI)

### **Definition**
Continuous Integration is the practice of **frequently merging all developers’ code changes** into a shared repository (like GitHub or GitLab), followed by **automated builds and tests**.

### **Purpose**
- Detect bugs early in the development process  
- Ensure all code works together properly  
- Automate repetitive build and test steps  
- Maintain consistent code quality

### **Typical CI Steps**
1. **Code Commit:** Developers push their code changes to Git.  
2. **Code Download & Dependency Installation:** The CI server (e.g., Jenkins, GitLab CI, GitHub Actions) fetches the code and installs dependencies.  
3. **Build:** Source code is compiled and packaged.  
4. **Static Analysis & Linting:** Check code style and vulnerabilities.  
5. **Unit Testing:** Validate individual functions or modules.  
6. **SAST (Static Application Security Testing):** Scan the source code for security flaws.  
7. **SCA (Software Composition Analysis):** Check dependencies for known vulnerabilities.  
8. **Dockerization:** Package the app into a Docker image.  
9. **Push to Artifact Repository:** Store the image or build artifact (e.g., in JFrog Artifactory, Nexus, or AWS ECR).

>  **Goal of CI:** Detect and fix issues early, before code reaches staging or production.

---

##  Continuous Delivery (CD)

### **Definition**
Continuous Delivery ensures that the **application can be released to production at any time** — all deployments to staging environments are **automated** and verified.

>  “Continuous Delivery” stops before *automatic* deployment to production — final approval is usually manual.

### **Steps in CD**
1. **Fetch Artifact:** Pull the build from the artifact repository.  
2. **Deploy to SIT (System Integration Testing) environment:**  
   - All modules are integrated and tested together.  
   - Validates that different parts of the system communicate correctly.  
3. **Performance Testing:**  
   - Check speed, scalability, and stability of the application.  
4. **Security Testing:**  
   - Run **DAST (Dynamic Application Security Testing)** on the deployed environment.  
   - Validate the running application for vulnerabilities.  
5. **Approval:**  
   - After validation, the release can be manually approved for production deployment.

---

##  Continuous Deployment

**Continuous Deployment** takes **Continuous Delivery** one step further —  
every code change that passes all stages of testing is **automatically deployed to production**, without manual approval.

### **Advantages**
- Ultra-fast release cycle  
- Reduced human errors  
- Real-time feedback and rollback  
- Fully automated pipeline

>  Tools commonly used: Jenkins, ArgoCD, Spinnaker, GitHub Actions, GitLab CI/CD, CircleCI

---

##  CI/CD Pipeline Overview

Below is a simplified view of a typical CI/CD process:

      ┌──────────────────────────────────────────────────────┐
      │                    Source Code                       │
      │          (GitHub / GitLab / Bitbucket)                │
      └──────────────────────────────────────────────────────┘
                            │
                            ▼
               ┌──────────────────────────┐
               │ Continuous Integration   │
               │  • Build                 │
               │  • Unit Testing          │
               │  • Linting & SAST        │
               │  • Docker Image Build    │
               │  • Push to Artifact Repo │
               └──────────────────────────┘
                            │
                            ▼
               ┌──────────────────────────┐
               │ Continuous Delivery      │
               │  • Deploy to SIT         │
               │  • Integration Testing   │
               │  • Performance Testing   │
               │  • Security (DAST, SCA)  │
               └──────────────────────────┘
                            │
                            ▼
               ┌──────────────────────────┐
               │ Continuous Deployment    │
               │  • Automated Production  │
               │    Release (Blue-Green)  │
               │  • Monitoring & Feedback │
               └──────────────────────────┘


---

##  Types of Testing in CI/CD

| **Test Type** | **Purpose** | **Tools / Examples** |
|----------------|-------------|-----------------------|
| **Unit Testing** | Tests individual functions or methods. Uses mocks to isolate modules. | JUnit, pytest, Mockito |
| **Integration Testing (SIT)** | Validates if different modules communicate correctly. | Postman, REST Assured |
| **System Testing** | Tests the entire system end-to-end. | Selenium, Cypress |
| **Performance Testing** | Ensures application can handle load. | JMeter, Locust |
| **Security Testing** | Detect vulnerabilities in code and runtime. | OWASP ZAP, Burp Suite |
| **SAST (Static App Security Testing)** | Scans source code for vulnerabilities before execution. | SonarQube, Checkmarx |
| **DAST (Dynamic App Security Testing)** | Tests a running application for vulnerabilities. | OWASP ZAP |
| **SCA (Software Composition Analysis)** | Checks for vulnerabilities in open-source libraries. | JFrog Xray, Black Duck |

---

##  Unit Testing and Mocking

### **What is Unit Testing?**
Unit testing is the process of testing **individual pieces of code (functions, classes, or modules)** in isolation.

### **Mocking**
When a function depends on other modules or APIs, we can **mock** those dependencies so that we test only the current function.

Example:
- Function `F2M2()` depends on another function `F3M2()`.  
  While testing `F2M2`, we assume `F3M2` works correctly.  
  This is **mocking**.

### **Tools for Mocking**
- **Mockito (Java)**  
- **unittest.mock (Python)**  
- **Sinon.js (JavaScript)**  

>  Mocking helps isolate the unit under test and ensures faster, more reliable unit testing.

---

#  Introduction to Linux

##  History
| Year | OS / Event | Notes |
|------|-------------|-------|
| **1969–1990** | UNIX | Developed at Bell Labs; foundation of modern OS design. |
| **1985–1995** | Microsoft Windows | GUI-based OS; became dominant in desktop systems. |
| **1991** | Linux | Created by Linus Torvalds; open-source UNIX-like OS. |

>  Today, over **90% of production servers** run on Linux.

---

##  What is Linux?
Linux is an **open-source, UNIX-like operating system kernel**.  
It acts as an **interface between hardware and software**, managing system resources.

### **Responsibilities of the Kernel**
- Manage **CPU** scheduling  
- Manage **Memory and Storage**  
- Manage **Input/Output devices**  
- Handle **Security and Authentication**

### **Types of Linux Architectures**
- **Monolithic Kernel:** All system services run in kernel space.  
- **Microkernel:** Minimal kernel with system services running in user space.

---

##  Common Linux Components
| Component | Description |
|------------|-------------|
| **Shell** | Command-line interface (e.g., Bash, Zsh). |
| **File System** | Organizes files hierarchically under `/`. |
| **Process Management** | Handles running applications and daemons. |
| **User Management** | Controls users, groups, and permissions. |

---

#  AWS IP Address Types

AWS supports **three main types of IP addresses** for EC2 instances:

| Type | Description |
|------|--------------|
| **Private IP** | Used within the VPC; remains attached to the instance. |
| **Public IP** | Changes each time the instance restarts unless it's elastic. |
| **Elastic IP (EIP)** | Static public IP that can be reattached to any instance. |

>  Private IPs allow internal communication between EC2s inside a VPC, while public and elastic IPs are used for external access.

---
