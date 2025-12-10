# ReplicaSet & Pods — Class 13 


## PART I — MY notes 



###  setup 

You set up a minimal cluster with **1 control-plane (master)** and **1 worker** node.

Typical steps and commands used 

#### 1. SSH into an instance

```bash
ssh -i nameof.pem ubuntu@<IP-address>
```

* **What / When / Where**: Run on your laptop/terminal to connect to a cloud VM. Replace `nameof.pem` with your SSH key and `<IP-address>` with the VM's public IP.
* **Notes**: Use `ubuntu` (or correct username) depending on image.

#### 2. Initialize control plane (master node)

```bash
kubeadm init
```

* **What**: Bootstraps a Kubernetes control plane on this node (generates certs, starts kube-apiserver, controller-manager, scheduler, writes admin kubeconfig).
* **Where**: Run on the node that will be the master.
* **Output**: `kubeadm` prints a `kubeadm join ...` command to run on worker nodes to join the cluster.
* **Flags you may use**:

  * `--apiserver-advertise-address=<IP>` to specify the API server IP advertised to workers.
  * `--pod-network-cidr=<CIDR>` to set the pod network range required by some CNIs (e.g., Flannel uses `10.244.0.0/16`).
* **Alternate**: If you lost the join command, you can regenerate it (see next).

#### 3. Regenerate the join command if lost

```bash
kubeadm token create --print-join-command
```

* **What**: Creates (or reuses) a bootstrap token and prints the full `kubeadm join ...` command.
* **Where**: Run on the control-plane node.

#### 4. Install a network plugin (CNI)


```bash
kubectl apply -f https://github.com/weaveworks/weave/release/download/v2.8.1/weave-daemonset-k8s.yaml
```

* **What**: Installs Weave Net as a Kubernetes CNI. The CNI is required for Pod-to-Pod networking (Pod IP allocation, veth setup).
* **When**: After `kubeadm init` and before expecting Pod networking to work.
* **Where**: Run on a machine with `kubectl` configured to talk to the control-plane (usually the master).
* **Alternatives**: Calico, Flannel, Cilium, etc. Each has its own manifest and may require `--pod-network-cidr` during `kubeadm init`.

#### 5. Watch system pods come up

```bash
watch -n 1 kubectl get po -n kube-system
```

* **What**: Runs `kubectl get po -n kube-system` every second so you can observe system pods starting (CoreDNS, kube-proxy, CNI DaemonSet pods).
* **Stop**: Press `Ctrl+C`.

---

### Creating a Pod 

created a test pod with:

```bash
kubectl run scalerdemo --image=nginx
```

* **What**: Quick command to start a workload named `scalerdemo` using the `nginx` image.
* **Where**: Any machine that has `kubectl` configured to the cluster (commonly the control-plane/machine with kubeconfig).
* **Important note**: `kubectl run` behavior changed in newer `kubectl` versions. By default it may create Deployments; to ensure you create a plain Pod:

```bash
kubectl run scalerdemo --image=nginx --restart=Never
```

* **Better practice**: In production or reproducible setups, prefer authoring a Pod/Deployment YAML and using `kubectl apply -f pod.yaml`.

Inspect Pods:

```bash
kubectl get po -o wide
```

* `-o wide` shows node assignment, Pod IP, and other extra details.

To confirm which node a Pod landed on run `hostname` on that node (or use `kubectl get po -o wide` to see the node column).

---

### Deleting Pods

used:

```bash
kubectl delete po --all
```

* **What**: Deletes all Pods in the current namespace (default namespace).
* **Important**: The correct flag is `--all` (lowercase). If you delete all Pods that are managed by higher-level controllers (ReplicaSet, Deployment), those controllers will recreate Pods to satisfy desired state.
* **Where**: On the control-plane (kubectl) or any machine with correct kubeconfig.

Then you recreated the pod:

```bash
kubectl run scalerdemo --image=nginx
```

---

### Inspecting container runtime on worker node

On the worker, you ran:

```bash
ps -ef | grep -E "crio|runc|crun"
crictl ps
```

* **What**:

  * `ps -ef | grep ...` checks for container runtime and OCI runtime processes (CRI-O, runc, crun).
  * `crictl ps` shows running containers via the CRI (Container Runtime Interface) tooling.
* **Where**: Run on the worker node (not on control-plane).
* **Why**: Useful for low-level debugging when kubelet asks the runtime to create containers.

You also noted `kill -9` to force-terminate processes — that can simulate container/pod crashes; kubelet will restart containers as per restart policy.

---

### Pod failure & replacement —  observation

* **Pod container crashes** → The node's kubelet ensures the container (and therefore Pod) is restarted on the same node. Kubelet monitors containers and works with the runtime to recreate them.
* **Node crash (machine dies)** → kubelet on that node is down, so local replacements cannot occur. The ReplicaSet (or controller) on control-plane detects pods on the dead node and recreates replacement pods on other nodes—this is why ReplicaSet is needed to survive node failures.

---

### Pod YAML template 

A minimal Pod manifest :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
          name: http
```

* **Important fields**:

  * `apiVersion` — API group and version used.
  * `kind` — resource type (`Pod`).
  * `metadata` — `name`, `labels`, and so on.
  * `spec` — desired state: containers, volumes, restartPolicy, etc.

---

### YAML basics

YAML data types:

1. **Key-value** (mapping):

   ```yaml
   name: nginx
   ```
2. **List**:

   ```yaml
   containers:
     - name: nginx
       image: nginx
   ```
3. **Map** (nested mapping):

   ```yaml
   ports:
     - containerPort: 80
       name: http
   ```

---

## PART II — Instructor notes

> a ReplicaSet ensures a specified number of Pod replicas are always running. It continuously reconciles desired vs actual state. Key components involved: kube-apiserver, kube-controller-manager (ReplicaSet controller), scheduler, kubelet, and etcd. 

### 1. What is a ReplicaSet (simple)

* A **ReplicaSet (RS)** is a Kubernetes controller that ensures a fixed number of Pod replicas matching a label selector are running at any time.
* It does **not** itself run Pods; it **creates Pod objects** via the API server and monitors them.

---

### 2. ReplicaSet YAML (example)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```

* **Explanation**:

  * `apiVersion: apps/v1` — ReplicaSet lives in `apps` API group.
  * `kind: ReplicaSet` — object type.
  * `metadata.name` — ReplicaSet name.
  * `spec.replicas` — desired number of pods.
  * `spec.selector.matchLabels` — label query that selects which pods belong to this RS.
  * `spec.template` — the Pod template used to create Pod objects (same structure as a Pod manifest).
* **Important**: The template’s labels must match the selector. If they don’t, the ReplicaSet cannot count or manage the pods properly.

---

### 3. Full control-plane flow (what happens when you `kubectl apply -f nginx-rs.yaml`)

1. **kubectl → API server**

   * `kubectl` sends a REST request to the API server (POST /apis/apps/v1/replicasets).
   * API server authenticates and authorizes the request. If accepted, admission controllers run (mutating/validating webhooks may change or reject the object).
2. **API server stores RS object in etcd**

   * Key path `/registry/replicasets/default/nginx-rs`.
   * RS spec (replicas, selector, pod template) is persisted.
3. **ReplicaSet controller (in kube-controller-manager)** sees the new RS via watch and acts:

   * It lists current Pods that match the selector. If `current < desired`, RS creates new Pod objects by POSTing Pod manifests to the API server.
4. **Pod objects appear in etcd** (e.g., `/registry/pods/default/web-abc12`).

   * Each Pod has `ownerReferences` that point to the ReplicaSet (so garbage collection knows the relationship).
5. **Scheduler** sees the Pending pods and assigns nodes by writing a `Binding` (or patching `spec.nodeName`).
6. **Kubelet** on each chosen node sees the assigned Pod, instructs the container runtime to pull image and start containers, and updates Pod status back to the API server.


---

### 4. The ReplicaSet reconcile loop

* ReplicaSet controller constantly watches for Pods that match its selector.
* Compare: `desired (spec.replicas)` vs `current (number of matching Pods running)`.

  * If `current < desired` → create new Pods from the template.
  * If `current > desired` → delete excess Pods.
* The RS will replace failed Pods and re-create Pods when they are deleted (unless deleted in a way that leaves orphans — see cascade flags below).

---

### 5. Failure scenarios & RS behavior

* **Single container/pod crash (container inside pod fails)**:

  * Kubelet restarts the container according to the Pod restartPolicy.
  * If the Pod becomes `Failed` or restarts too many times, RS sees `current` reduced and creates a replacement Pod.
* **Node dies**:

  * Node controller marks node NotReady or Pods `Unknown` after some time.
  * ReplicaSet detects missing Pods and creates replacement Pods on other nodes.
* **Manual Pod deletion**:

  * If you delete a Pod (without deleting ReplicaSet), RS will create a new one to keep the desired count.

---

### 6. Deleting the ReplicaSet

```bash
kubectl delete rs nginx-rs
```

* **What happens**: Deleting a ReplicaSet will delete the ReplicaSet object. By default, Kubernetes will also delete all Pods owned by it (ownerRef cascade deletion). The RS controller stops managing Pods after deletion.
* **Cascade behaviors**:

  * `kubectl delete rs nginx-rs` uses default deletion which usually cascades to owned pods.
  * To delete RS but leave pods orphaned:

    ```bash
    kubectl delete rs nginx-rs --cascade=orphan
    ```
  * To explicitly ensure cascade:

    ```bash
    kubectl delete rs nginx-rs --cascade=true
    ```

---

### 7. How the ReplicaSet stores relationships in etcd

* Pod objects created by the RS have `metadata.ownerReferences` that point to the RS object. This enables the garbage collector to delete owned Pods if ReplicaSet is deleted and determines ownership for reconciliation. The PDF shows `/registry/replicasets/...` and `/registry/pods/...` layouts. 

---

### 8. Commands & their purpose, where to run, useful flags

#### Initialize cluster (control-plane)

```bash
kubeadm init
```

* Run on the node that will be control-plane. Use flags for advertise address and pod CIDR if needed.

#### Generate join command (control-plane)

```bash
kubeadm token create --print-join-command
```

#### Install CNI plugin (any machine with kubectl talking to the API server — usually the control-plane)

```bash
kubectl apply -f <CNI manifest URL>
```

* Example:

```bash
kubectl apply -f https://github.com/weaveworks/weave/release/download/v2.8.1/weave-daemonset-k8s.yaml
```

#### Create a Pod quickly (kubectl client)

```bash
kubectl run scalerdemo --image=nginx --restart=Never
```

* `--restart=Never` forces creation of a Pod object instead of a Deployment (depending on `kubectl` version).
* Better: create a YAML and `kubectl apply -f pod.yaml`.

#### Show Pods (control-plane or any kubectl client)

```bash
kubectl get pods          # in default namespace
kubectl get pods -A       # all namespaces
kubectl get pods -o wide  # node and IP info
```

#### Delete pods

```bash
kubectl delete po --all
```

* Deletes **all pods** in the current namespace (be careful).

#### ReplicaSet create/update

```bash
kubectl apply -f nginx-rs.yaml
```

* Use `apply` for declarative management and `kubectl create -f` for one-time creation.

#### Inspect ReplicaSet

```bash
kubectl get rs
kubectl describe rs nginx-rs
kubectl get pods -l app=web
```

* `-l app=web` lists pods matching the RS selector.

#### Inspect scheduler / pod binding events

```bash
kubectl describe pod <pod>
```

* Useful to see scheduling events and reasons if `Pending`.

#### Inspect owner references

```bash
kubectl get pod <pod> -o yaml
```

* Look for `metadata.ownerReferences` to see which controller owns the pod.

---

### 9. Practical debugging tips  
* If Pod remains `Pending`, run:

  ```bash
  kubectl describe pod <pod> -n <ns>
  ```

  Look for reasons: `0/3 nodes available` (insufficient resources, taints, volume attach issues).
* `ImagePullBackOff` → check image name, imagePullSecrets.
* `ContainerCreating` stuck → likely CNI or volume attach issues.
* `CrashLoopBackOff` → check `kubectl logs <pod> -c <container>` and liveness probes.

---

### 10. Key takeaway points (concise)

* A **Pod** is the smallest deployable unit — one or more containers co-scheduled on same node.
* A **ReplicaSet** ensures a desired number (`spec.replicas`) of Pods matching a label selector are running.
* The control plane (API server, etcd, scheduler, controllers) + kubelet + container runtime coordinate to create, schedule, and run Pods.
* ReplicaSets reconcile continuously and handle common failure scenarios (pod crash, node crash, manual deletion).
* Use YAML manifests and `kubectl apply` for reproducible infrastructure as code.

---

## Appendix — Minimal examples

### Pod YAML (`pod.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: web
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

### ReplicaSet YAML (`nginx-rs.yaml`)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```

Apply:

```bash
kubectl apply -f nginx-rs.yaml
```

Delete (and cascade delete owned pods):

```bash
kubectl delete rs nginx-rs
```

---
