

# Docker Networking + Subnetting + Core Networking Concepts



# 1. IP Address

An IP address is a **unique routable address** assigned to a device on a network that follows the IP protocol.
An IPv4 address is:

* 32-bit
* Written in dotted decimal
* Range of each octet: 0–255
* Example: `192.168.10.24`

Your note:

> ip address → it is unique routable address given to a device on a network which follows IP protocol.

---

# 2. Ethernet Card (NIC)

From PDF → 

NIC (Network Interface Card):

* Hardware that connects machine to network.
* Converts digital data ↔ electrical signals.
* Adds MAC address to frames.
* Sends/receives frames using CSMA/CD.
* Exists in:

  * Wired (RJ45)
  * Wireless (Wi-Fi)

Everything—browsers, containers, VMs—ultimately send traffic through the NIC.

---

# 3. Ethernet Cable

A physical cable to connect machines/switches.

Types:

* Cat5e → 1 Gbps
* Cat6 → 10 Gbps (55m)
* Cat6a → 10 Gbps (100m)
* Cat7/8 → Datacenter speeds

Carries electrical signals (bits).
PDF Reference → 

---

# 4. Network Switch

A Layer-2 device that:

* Connects multiple devices in LAN
* Forwards frames using MAC address
* Maintains MAC address table
* Avoids broadcasting frames unnecessarily

Switch workflow (PDF → ):

1. Device sends a frame.
2. Switch reads destination MAC.
3. Looks up MAC table.
4. Forwards frame only to specific port.

---

# 5. ARP Protocol

ARP = Address Resolution Protocol.

Purpose:
IP addresses identify a device logically,
but NICs only understand MAC addresses.

Workflow (PDF → ):

1. A broadcasts → “Who has 192.168.1.20?”
2. Device B replies → “I am 192.168.1.20, MAC = 11:22:33...”
3. A stores this in ARP cache.

---

# 6. Gateway

A gateway is the router used to reach networks **outside your subnet**.

Example:
Your system: `192.168.1.10/24`
Destination: `8.8.8.8`
Since 8.8.8.8 is outside /24, traffic goes to gateway → `192.168.1.1`.

---

# 7. NAT (SNAT & DNAT)


### SNAT (Source NAT)

Used for **outgoing** connection to internet.
Private IP → router replaces with public IP.

### DNAT (Destination NAT)

Used for incoming traffic (port forwarding).
Example:
External request → public IP:80
Router rewrites to → `192.168.1.50:80`

---

# 8. Subnetting 

Subnetting = Dividing one large network into smaller networks.

Example network:
`192.168.10.0/24`
Total IPs = 256
Because:
32 - 24 = 8
2^8 = 256

This `/24` is the **subnet mask**.

---

# 9. Subnet Table 

```
------------------------------------------------------------
network | 1 | 2 | 4 | 8 | 16 | 32 | 64 | 128 | 256 |
------------------------------------------------------------
hosts   |256|128|64|32|16|8|4|2|1|
------------------------------------------------------------
subnet  |/24|/25|/26|/27|/28|/29|/30|/31|/32|
------------------------------------------------------------
```

---

# 10. Creating 3 Networks (Engineering, HR, Reception)

Requirement: 3 networks.
Nearest is **4 networks**, so we subnet /24 into **four /26 networks**.

Each /26 network has:

32 - 26 = 6
2^6 = 64 IPs.

---

# Engineering Team Network

Range:
`192.168.10.0 - 192.168.10.63`

Network ID: `192.168.10.0`
Broadcast ID: `192.168.10.63`

Representation:
`192.168.10.0/26`

---

# HR Team Network

Range:
`192.168.10.64 - 192.168.10.127`

Network ID: `192.168.10.64`
Broadcast ID: `192.168.10.127`

Representation:
`192.168.10.64/26`

---

# Reception Network (Next Subnet)

Range:
`192.168.10.128 - 192.168.10.191`
Subnet = `/26`

---

# 11. Notes About IPv4 

* IPv4 = 4 numbers separated by `.`
* Each number = 0–255

Examples:
`66.249.70.102/32` → only one device allowed.

`/32` → all 32 bits for network
No bits for hosts.

`/31` → 2 IPs
`/30` → 4 IPs
`/23` → 512 IPs (32 - 23 = 9; 2^9 = 512)

Your note:

> 192.168.10.0/23 → size = 512
> Range → 192.168.10.0 to 192.168.11.255

You must always pick ranges starting from an even block.

Example:
`192.168.0.1/23` → INVALID
Correct:
`192.168.0.0/23`
or
`192.168.2.0/23`

---

# 12. Docker Networking 


Docker uses:

* Network namespaces
* veth pairs
* Linux bridges
* iptables
* NAT
* Overlay networks

---

## 12.1 Network Namespace

Each container gets:

* Its own `eth0`
* Its own routing table
* Its own ARP table
* Its own firewall rules

Makes container feel like a separate machine.

---

## 12.2 veth Pairs

A veth pair acts like a virtual Ethernet cable.

One end → container (`eth0`)
Other end → Docker bridge (`vethXYZ`)

---

## 12.3 docker0 Bridge

When Docker installs, it creates a default bridge:

```
docker0 → 172.17.0.1/16
```

This bridge:

* Allows containers to talk to each other
* Provides private IPs
* NATs traffic so containers can reach the internet


---

# 13. Types of Docker Networks


| Type    | Use Case                              |
| ------- | ------------------------------------- |
| bridge  | default container communication       |
| host    | container shares host network stack   |
| none    | no networking                         |
| overlay | multi-host networking (swarm)         |
| macvlan | containers appear as real LAN devices |

---

# 14. Bridge Network (Default)

Create your own bridge:

```
docker network create mynet
```

Run containers:

```
docker run -d --network mynet --name app1 nginx
docker run -d --network mynet --name app2 busybox
```

They can ping each other by **name**:

```
ping app1
```

---

# 15. Host Network

```
docker run --network host nginx
```

Container bypasses namespace isolation and uses host network directly.

---

# 16. None Network

```
docker run --network none alpine
```

Container has no network interface.

---

# 17. Overlay Network

Used when containers on separate hosts must talk:

```
docker network create -d overlay myoverlay
```

---



# 18. How Container Access Internet (NAT)

Docker uses iptables:

```
POSTROUTING MASQUERADE rule
```

Enables:

`container → docker0 → host eth0 → internet`

---

# 19. Port Mapping (-p)

```
docker run -p 8080:80 nginx
```

Meaning:

* Host 8080 → Container 80
* Supports DNAT at host level

PDF Reference → 

---

# 20. Inspecting Networks

```
docker network ls
docker network inspect bridge
docker inspect <container>
```

---

# 21. Practical Example: App + DB

```
docker network create backend
docker run -d --network backend --name db mysql
docker run -d --network backend --name app myapp
```

App connects using hostname `db:3306`.

---


