
# **Docker Basics on AWS EC2 — Detailed Notes**


## **1. Connecting to AWS EC2 Instance**

After creating an EC2 instance, we need to SSH into it.

### **Steps:**

1. Open the AWS dashboard → go to your instance → click on **Connect**.
2. Choose **SSH Client**.
3. Copy the SSH command provided (it includes the public DNS + your key).
4. On your local system, go to the directory where your `.pem` file is stored:

   ```bash
   cd /path/to/your/pem-file
   ```
5. Run the SSH command:

   ```bash
   ssh -i your-key.pem ubuntu@ec2-xx-xx-xx-xx.compute.amazonaws.com
   ```
6. Now your PowerShell terminal is connected to the Ubuntu machine on EC2.

---

 ## **2. Creating a Docker Container (hello-world)**

To create a container:

```bash
docker run hello-world
```

If Docker is not installed it will show an error.
After installation, running this command downloads the image and creates the container.

Example output:

```
Unable to find image 'hello-world:latest' locally
...
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

This output explains how Docker works internally.

---

 ## **3. Understanding What This Output Means**

### **3.1 Docker Run Always Creates a New Container**

Whenever we execute:

```bash
docker run <image-name>
```

Docker creates a **new container** from that image.

---

 ## **4. Docker Images**

* Containers are **always created from images**.
* Images = templates or blueprints.
* Analogy:
  **Image = Class**, **Container = Object instance**.
* Images can be:

  * **Local** (already downloaded)
  * **Remote** (Docker Hub)

If image is missing locally:

```
Unable to find image locally...
Pulling from library...
```

This means Docker pulled the image from Docker Hub.

---

## **5. What Are Containers?**

* Containers behave like **lightweight virtual machines**.
* Containers always come from images.
* Each container is an isolated environment with its own filesystem & processes.
* They share the host OS kernel (which makes them lightweight).

---

## **6. Docker Client & Docker Daemon**

When you execute a Docker command:

### **Docker Client**

* The CLI tool (`docker`) that you interact with.
* Sends commands to Docker Daemon.

#### **Docker Daemon**

* Runs in the background as a service.
* Responsible for:

  * Pulling images
  * Creating containers
  * Running containers
  * Managing container lifecycle

Output example:

```
The Docker client contacted the Docker daemon.
The Docker daemon pulled the image...
The Docker daemon created a container...
```

---

## **7. Daemon and Background Processes in Linux**

Linux background services = **daemons**.

To view background services:

```bash
systemctl status
```

To see Docker-specific daemon:

```bash
systemctl status docker
```

---

## **8. Docker Installation Results in 2 Components**

After installation Docker provides:

1. **Docker client** → CLI tool
2. **Docker daemon** → background engine

Check versions:

```bash
docker version
```

---

## **9. Getting Inside a Docker Container**

We can start a container and open a terminal inside it:

```bash
docker run -it ubuntu bash
```

### Flags:

* `-i` → interactive
* `-t` → terminal access (tty)
* `ubuntu` → image name
* `bash` → run bash shell inside container

### Example:

Before container:

```
root@ip-172-31-32-238:/home/ubuntu
```

After entering container:

```
root@f21106bca534:/#
```

The hostname changed → this is the **container ID**.

---

## **10. Processes Inside a Container**

Inside the container, run:

```bash
ps -ef
```

This will show **only the processes inside that container** because containers have isolated process namespaces.

---

## **Summary Flow of How Docker Works**

1. You run:

   ```bash
   docker run hello-world
   ```
2. **Client** sends request to **Daemon**.
3. Daemon checks if image exists locally.
4. If not found → pulls from Docker Hub.
5. Daemon creates container from that image.
6. Daemon runs the container.
7. Output is sent back to Client → terminal.

---

## **Most Important Concepts Learned**

* Images = blueprint
* Containers = instances
* docker run = create new container
* Docker client vs Docker daemon
* Daemons = background Linux services
* Enter container using `docker run -it ubuntu bash`
* Containers have isolated processes (`ps -ef`)

