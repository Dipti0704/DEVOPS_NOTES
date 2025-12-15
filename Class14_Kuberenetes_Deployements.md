
# Class 14 — Deep Dive into Deployment (Kubernetes)

---

## PART 1 — My Notes

### What is a Deployment?

A **Deployment** is a higher-level Kubernetes object used to manage applications.

 notes:

* Deployment **internally creates ReplicaSets**
* ReplicaSets **internally create Pods**

So the hierarchy is:

```
Deployment
 └── ReplicaSet
      └── Pods
           └── Containers
```

You never directly manage Pods in production.
You manage **Deployments**, and Kubernetes handles everything else.

---

### Deployment vs ReplicaSet 


* Deployment YAML looks almost **exactly like ReplicaSet YAML**
* The **only visible difference** is:

  ```yaml
  kind: ReplicaSet
  ```

  vs

  ```yaml
  kind: Deployment
  ```

Why is that?

Because:

* Deployment **wraps ReplicaSet**
* Deployment adds **extra intelligence** on top of ReplicaSet:

  * rolling updates
  * rollback
  * revision history
  * pause/resume

ReplicaSet alone cannot do these.

---

### Advantage Chain (Very Important Concept)


#### Advantages of Containers

* Lightweight
* Fast startup
* Isolated processes

⬇️ inherited by

#### Advantages of Pods

* If a **container crashes**, kubelet restarts it
* Shared network & storage

⬇️ inherited by

#### Advantages of ReplicaSets

* If **pod crashes**, it is recreated
* If **node crashes**, pod is recreated on another node
* Easy **scale in / scale out**

⬇️ inherited by

#### Advantages of Deployments

* Everything above
* PLUS:

  * Rolling updates
  * Rollbacks
  * Version history
  * Declarative updates

---

### Why Do We Even Need Deployment?


* Pod crash → kubelet handles it
* Node crash → kubelet is gone → pod will not come back

This is the **key reason** for ReplicaSets and Deployments.

Deployment ensures:

* Even if **nodes die**
* Even if **pods are deleted**
* Desired state is always maintained

---

## Cluster Setup 

 again created:

* 1 master node
* 1 worker node

### Change hostname (Master Node)

```bash
hostnamectl set-hostname controlplane
hostname
```

* `hostnamectl set-hostname` → permanently changes hostname
* `hostname` → verifies the hostname

---

### Initialize Kubernetes Control Plane

```bash
kubeadm init
```

* Converts this machine into a **master (control-plane)**
* Starts:

  * API Server
  * Scheduler
  * Controller Manager
  * etcd
* Prints **join command** for worker node

Run join command on worker node.

---

### Check Node Status

```bash
kubectl get nodes
```

At this point, status shows **NotReady**

Why?
Because **network plugin is not installed**

---

### Check System Pods

```bash
kubectl get po -n kube-system
```

 observed:

* `coredns` pods are not running

Reason:

* CoreDNS requires networking
* CNI plugin not installed yet

---

### Install Network Plugin (Weave)

```bash
kubectl apply -f https://github.com/weaveworks/weave/release/download/v2.8.1/weave-daemonset-k8s.yaml
```

* This installs **Weave CNI**
* Creates a DaemonSet
* One networking pod per node

Watch system pods:

```bash
watch -n 1 kubectl get pod -n kube-system
```

Once CoreDNS is running → cluster is ready.

---

## Working with ReplicaSet (Your Notes)

Create ReplicaSet:

```bash
kubectl apply -f replica-set.yaml
```

### Scaling ReplicaSet

#### Declarative way (recommended)

* Edit YAML:

  ```yaml
  replicas: 5
  ```
* Apply again:

  ```bash
  kubectl apply -f replica-set.yaml
  ```

#### Imperative way

```bash
kubectl scale rs <replicaset-name> --replicas=5
```

* `rs` → ReplicaSet
* `--replicas` → desired pod count

---

## Deployment 

Deployment gives you:

* Update
* Rollback

### Create Deployment

```bash
kubectl apply -f deploy-ng.yaml
```

Watch everything together:

```bash
watch -n 1 kubectl get po,rs,deploy
```

This shows:

* Pods
* ReplicaSets
* Deployments
  all at once.

---

### Scale Deployment

```bash
kubectl scale deploy nginx-deployment --replicas=3
```

* `deploy` → Deployment
* This updates Deployment spec
* Deployment updates ReplicaSet
* ReplicaSet creates pods

---

### Types of Scaling (Your Notes)

Kubernetes supports **three scaling mechanisms**:

1. **HPA** — Horizontal Pod Autoscaling

   * Scales number of pods based on CPU/memory

2. **VPA** — Vertical Pod Autoscaling

   * Changes CPU/RAM limits of pods

3. **CA** — Cluster Autoscaling

   * Adds/removes nodes automatically

---

---

## PART 2 — Instructor Notes 


---

## Why Deployment Exists (Instructor Explanation)

Standalone Pods:

* Are **not self-healing**
* If they die → Kubernetes does NOT bring them back

ReplicaSets:

* Ensure pod count
* But cannot:

  * do rolling updates
  * rollback versions

Hence:
 **Deployments are the recommended workload controller**

---

## What Deployment Does Internally

Deployment:

* Creates and owns ReplicaSets
* Manages **multiple ReplicaSets over time**
* Tracks **revisions**

Every update creates:

* A **new ReplicaSet**
* Old ReplicaSet is scaled down (not deleted immediately)

---

## Basic Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: app
        image: nginx:1.27
        ports:
        - containerPort: 80
```

---

## What Happens Internally (`kubectl apply -f deployment.yaml`)

### Step 1 — API Server

* Validates YAML
* Checks RBAC
* Runs admission controllers
* Stores Deployment in etcd

### Step 2 — Deployment Controller

* Sees new Deployment
* Checks if ReplicaSet exists
* Creates a new ReplicaSet if not

### Step 3 — ReplicaSet Controller

* Ensures correct pod count
* Creates Pods

### Step 4 — Scheduler

* Assigns pods to nodes

### Step 5 — Kubelet

* Pulls image
* Creates Pod sandbox
* Starts containers

All states stored in **etcd**.

---

## Rolling Updates (Very Important)

Default strategy: **RollingUpdate**

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 25%
    maxSurge: 25%
```

Meaning:

* Some old pods can go down
* Some new pods can come up
* Zero downtime

Other strategy: **Recreate**

* Deletes all pods first
* Then creates new ones
* Used for stateful apps

---

## Updating a Deployment

```bash
kubectl set image deployment/webapp app=nginx:1.28
```

What happens:

1. New ReplicaSet created
2. Old ReplicaSet scaled down
3. New pods gradually started
4. Revision updated in etcd

---

## Rollout Commands

Check rollout status:

```bash
kubectl rollout status deployment/webapp
```

View history:

```bash
kubectl rollout history deployment/webapp
```

Rollback:

```bash
kubectl rollout undo deployment/webapp
```

Rollback to specific revision:

```bash
kubectl rollout undo deployment/webapp --to-revision=2
```

---

## Pause & Resume Deployment

Pause:

```bash
kubectl rollout pause deployment/webapp
```

Resume:

```bash
kubectl rollout resume deployment/webapp
```

Useful for:

* Canary deployments
* Debugging production rollouts

---

## Scaling Deployment 

```bash
kubectl scale deployment webapp --replicas=10
```

Updates:

* Deployment spec
* ReplicaSet
* Pods created automatically

---

## How Deployment is Stored in etcd

* Deployment:

  ```
  /registry/deployments/default/webapp
  ```
* ReplicaSet:

  ```
  /registry/replicasets/default/webapp-xxxxx
  ```
* Pod:

  ```
  /registry/pods/default/webapp-xxxxx-yyyyy
  ```

Each update increments:

* `deployment.kubernetes.io/revision`

Rollback just points Deployment to an older ReplicaSet.

---

## Final Summary 

* Pod → basic unit (not self-healing)
* ReplicaSet → ensures pod count
* Deployment → manages ReplicaSets
* Deployment is **recommended** for stateless apps
* Supports:

  * rolling updates
  * rollback
  * pause/resume
  * declarative updates
* All state is stored in etcd
* Kubernetes continuously reconciles desired vs actual state

---
