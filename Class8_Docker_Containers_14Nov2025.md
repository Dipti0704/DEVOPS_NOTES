
# 1. What Exactly Is a Container?

A container is **not a VM**, **not a mini OS**, and **not an emulator**.

A container is simply:

**A Linux process + a temporary filesystem + namespaces + cgroups**

Internally, a container:

* Is just a process in the host OS
* But the process is isolated using Linux kernel features
* The filesystem is layered and temporary (ephemeral)
* It uses the host kernel, not its own kernel
* It has its own fake world (network, PID tree, hostname, filesystem)

---

# 2. Hypervisor vs Containers (Deep Explanation)

## 2.1 What is a Hypervisor?

A hypervisor is a category of software that allows us to create and run Virtual Machines.
Each VM has:

* Its own OS
* Its own kernel
* Its own libraries
* Its own root filesystem
* Its own init system, BIOS, MBR, bootloader

Thus, a VM behaves like a full computer.

---

# 2.2 Why Virtual Machines Are Heavy

Every VM has:

1. A complete OS
2. Full memory allocation
3. Independent kernel
4. Boot sequence (BIOS → MBR → Bootloader → Kernel → INIT → Processes)
5. Virtual hardware: CPU, RAM, NIC, Disk

This makes VMs slow to start and consume large resources.

---

# 2.3 VM Boot Process (Full Detailed Explanation)

**This happens for every VM independently:**

### 1. BIOS

Hardware checks + boot device selection.

### 2. MBR

Reads partition table, identifies OS to boot.

### 3. Bootloader

Loads the selected OS kernel.
Example: GRUB for Linux, Windows Boot Manager for Windows.

### 4. Kernel Loads

The kernel initializes:

* Memory manager
* Process scheduler
* System calls
* Drivers

### 5. INIT Process

`systemd` or `init` starts services.

### 6. Processes + Applications Launch

Only after all these steps does your program start.

---

# 2.4 Why Docker Does Not Do This

Containers do **not** have:

* BIOS
* MBR
* Bootloader
* Their own OS
* Their own kernel

A container simply starts its **main process directly**.

No OS boot.
No init system unless you add one.
Startup time is milliseconds.

---

# 3. What Happens Internally When You Create a Container?



When you run:

```
docker run nginx
```

Internally, Docker performs these steps:

---

## Step 1: docker → containerd → runc Pipeline


```
docker client
    ↓
containerd
    ↓
containerd-shim
    ↓
runc
    ↓
your process (nginx)
```

---

## Step 2: containerd creates the container

containerd:

* Sets up filesystem layers (OverlayFS)
* Creates namespaces (PID, NET, IPC, UTS, MNT, USER)
* Creates cgroups for resource limits
* Prepares root filesystem
* Starts containerd-shim

---

## Step 3: runc uses Linux syscalls

runc performs:

* clone
* unshare
* chroot/pivot_root
* mount namespaces
* cgroup assignments

Then it executes the main container command:

```
/usr/sbin/nginx
```

This is now **PID 1 inside the container**, but just a normal PID (ex: 9422) on the host.

---

## Step 4: containerd-shim manages lifecycle

Shim ensures:

* Container keeps running even if containerd restarts
* It becomes the parent of the container's process
* Reports process exit to containerd

---

## Step 5: Container runs until PID 1 exits

If the main process exits:

* containerd-shim detects exit
* Network namespace destroyed
* Writable layer deleted
* Cgroups cleaned
* Container disappears

Hence:

A container = a process + temporary filesystem
Not a VM.

---

# 4. Why Containers Are Ephemeral


A container uses:

* Read-only image layers
* A temporary writable layer

When a container stops:

* Writable layer is deleted
* Logs, temp files, internal state vanish

To persist data, use:

* Docker volumes
* Bind mounts
* External storage (DB, S3)

---

# 5. How Containers Create Their "Own World"

Containers feel like separate machines because of **namespaces**.

### Namespaces provide pseudo-isolation:


1. **PID Namespace**
   Process sees itself as PID 1.

2. **Network Namespace**
   Own IP, routing table, firewall rules.
   Host sees only a veth pair.

3. **Mount Namespace**
   Own filesystem via OverlayFS.

4. **UTS Namespace**
   Own hostname.

5. **IPC Namespace**
   Own shared memory.

6. **USER Namespace**
   Root inside container ≠ real root on host.

---

# 6. Difference Between Two Containers vs Two VMs


### VM Differences:

1. Different IP address
2. Fully independent network stack
3. Separate process space
4. Separate hostname
5. Separate filesystem
6. Separate hardware abstraction
7. Separate kernel
8. Boot process
9. Strong isolation

### Container Differences:

Containers do not have independent OS or kernel, but they have:

1. Separate network namespace
2. Separate PID namespace
3. Separate mount namespace
4. Separate IPC namespace
5. Separate hostname
6. Separate cgroup limits

This makes them isolated but lightweight.

---

# 7. Ports and Why Containers Fail When Port Is Already Used


```
docker run -d -p 80:80 nginx
```

Meaning:

* First `80` → host port
* Second `80` → container port

If you run second container:

```
docker run -d -p 80:80 nginx
```

It fails because:

* Host port 80 is already taken
* Two containers cannot bind to the same host port

Containers can share internal ports, but not host ports.

---

# 8. Why Containers Are Lighter Than Normal Programs


A container is lighter than a Java program because:

* No kernel
* No OS boot
* No init system
* No virtual hardware
* Only isolation + filesystem

Running a container is essentially running a Linux process with isolation.

---

# 9. Container Lifecycle


1. runc starts PID 1
2. containerd-shim monitors PID 1
3. If PID 1 exits → container stops
4. Writable FS deleted
5. Cgroups released
6. Namespace destroyed

---

# 10. Commands Used in Class

### Install docker:

```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo su
```

### Run nginx:

```
docker run -d -p 80:80 nginx
```

### See running containers:

```
docker ps
```

### See all (including stopped):

```
docker ps -a
```

---

# 11. Summary of the Entire Concept

* A container is not a VM
* A container is a process
* Isolation = namespaces
* Limits = cgroups
* Filesystem = overlay layers
* runc creates the environment
* containerd manages container lifecycle
* Docker is fast because it does not boot OS
* Containers are ephemeral
* VM has a full OS → heavy
* Docker shares kernel → lightweight

---
