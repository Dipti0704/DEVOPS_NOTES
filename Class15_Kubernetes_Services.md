
# Class 15 — Kubernetes Services (Deep Dive)

---

## PART 1 — Notes (Expanded & Explained)

### What is a Service?

From your notes:

> services = load balancers

More clearly:

A **Kubernetes Service** is an abstraction that acts like a **load balancer + service discovery mechanism** for Pods.

Any load balancer must have **two core capabilities**:

1. **Distribute load**

   * Traffic should be spread across multiple instances.
2. **Service discovery (health awareness)**

   * It should know when:

     * a new instance comes up
     * an existing instance goes down

Kubernetes Services provide both.

---

### Story Explanation (Blue Service & Red Service)

scenario:

* There are **3 nodes**
* Node 1 runs **Blue service**
* Node 2 runs **Red service**
* Blue service wants to talk to Red service

#### Can Blue directly talk to Red Pod?

Yes, technically it can (using Pod IP).

#### Should we do this?

No.

#### Why not?

Because:

* Pod IPs are **not stable**
* If many services talk to **one Pod**, that Pod gets overloaded
* No load distribution
* If that Pod crashes → communication breaks

---

### Why Load Balancer Is Needed

Correct approach:

* Blue service talks to a **load balancer**
* Load balancer forwards traffic to **Red Pods**
* If:

  * a new Red Pod comes up → traffic is shared
  * an old Red Pod crashes → traffic is stopped to it

In Kubernetes, this **load balancer abstraction is called a Service**.

---

### ClusterIP (Core Kubernetes Service)

 notes:

> in kubernetes this is called clusterIP

* **ClusterIP** is the default Service type
* It provides:

  * a **stable virtual IP**
  * a **stable DNS name**
  * internal load balancing across Pods

ClusterIP has:

* an **IP address**
* a **port**

Pods also expose ports called **targetPorts**.

---

### Why Direct Pod Communication Is a Problem

You listed these problems correctly:

1. Pod crashes → new Pod gets a **new IP**
2. Multiple services talking to **one Pod**

   * No load distribution
   * Other Pods unused
   * Single point of failure

That’s why **Services exist**.

---

### Ports in a Service 

When two services communicate:

* **Service Port (`port`)**

  * Port exposed by the Service (virtual IP)
* **Target Port (`targetPort`)**

  * Actual port on the Pod container

Flow:

```
Client → ServiceIP:port → PodIP:targetPort
```

---

### Commands 

```bash
kubectl get svc
```

* Lists all services in the current namespace

```bash
kubectl create ns batch2027
kubectl get ns
```

* Creates and lists namespaces

```bash
kubectl run mypod --image=nginx -n batch2027
```

* Creates a Pod in `batch2027` namespace
* Note: Correct flag is `--image=nginx`

```bash
kubectl get po -n batch2027 -o wide
```

* Shows Pod IP and node placement

```bash
curl <IP-address>
```

* Tests direct Pod access (not recommended in real setups)

---

### Types of Services 

 types:

1. **ClusterIP**

   * Internal-only communication
2. **NodePort**

   * Exposes service on each node
3. **LoadBalancer**

   * Cloud-managed external LB
4. **External IP**

---

### NodePort 

Problem:

> what if an external machine wants to talk to the service?

Solution:

* Create a **NodePort Service**

How NodePort works:

* Kubernetes picks a port in range **30000–32767**
* This port must be free on **all nodes**
* Kubernetes automatically finds a free port

Flow:

```
External Client → NodeIP:NodePort → ClusterIP → Pod
```

Important points from your notes:

* NodePort **always creates a ClusterIP internally**
* NodePort forwards traffic to ClusterIP
* ClusterIP does the actual load balancing

---

## PART 2 — Instructor Notes 


---

## Why Kubernetes Services Exist

Pods in Kubernetes are **ephemeral**:

* They get new IPs when restarted
* They scale up/down dynamically
* They can be deleted and recreated by ReplicaSets or Deployments

Because of this:

* You **cannot rely on Pod IPs**
* You need a stable abstraction

That abstraction is a **Service**.

---

## What a Kubernetes Service Provides

A Service gives you:

* Stable virtual IP (**ClusterIP**)
* Stable DNS name
* Load balancing across Pods
* Traffic routing using **kube-proxy**
* Access from inside or outside the cluster

---

## How a Service Works Internally

1. You create a Service with a **selector**

   ```yaml
   selector:
     app: backend
   ```
2. Kubernetes finds all Pods with matching labels
3. **kube-proxy** programs networking rules using:

   * `iptables` or
   * `IPVS`
4. Traffic sent to Service IP is forwarded to backend Pods

---

## Service Types (Detailed)

### 1. ClusterIP (Default)

Used for **internal communication only**.

Use cases:

* Microservice → microservice calls
* Backend → database
* Internal APIs

Example YAML:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
```

* `port` → Service port
* `targetPort` → Pod container port

Access inside cluster:

```bash
curl http://backend-svc
```

Full DNS:

```bash
backend-svc.default.svc.cluster.local
```

---

### 2. NodePort

Exposes Service on every node’s IP.

Port range:

```
30000 – 32767
```

YAML:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-nodeport
spec:
  type: NodePort
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 32080
```

* `nodePort` is optional
* If omitted, Kubernetes auto-assigns

Access:

```text
http://<NodeIP>:32080
```

Limitations:

* Exposed on all nodes
* No TLS termination
* Not ideal for production internet traffic

---

### 3. LoadBalancer (Cloud Environments)

Used in cloud providers:

* AWS
* GCP
* Azure
* DigitalOcean

Flow:

```
Internet → Cloud LB → NodePort → ClusterIP → Pod
```

YAML:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-lb
spec:
  type: LoadBalancer
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
```

After creation:

```bash
kubectl get svc backend-lb
```

You will see:

* `EXTERNAL-IP`

---

### 4. ExternalName Service

Used to map a Service to an **external DNS name**.

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: database.company.com
```

No selectors, no Pods.

---

## Service DNS Resolution

Every Service gets a DNS name:

```
<service-name>.<namespace>.svc.cluster.local
```

Example:

```text
redis.default.svc.cluster.local
```

This is handled by **CoreDNS**.

---

## kube-proxy Role

kube-proxy:

* Runs on every node
* Programs `iptables` or `ipvs` rules
* Routes:

  ```
  ServiceIP → PodIP
  ```

It does **not** proxy traffic itself; it configures kernel networking.

---

## Which Service Type Should You Use?

| Requirement               | Service Type |
| ------------------------- | ------------ |
| Internal microservices    | ClusterIP    |
| Local / bare-metal access | NodePort     |
| Cloud production traffic  | LoadBalancer |
| Stateful apps / DB        | Headless     |
| External services         | ExternalName |

---

