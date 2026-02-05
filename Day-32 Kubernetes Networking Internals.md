# CKA 2024 Video 32: Kubernetes Networking Internals (CRI/OCI/CNI, Pause Container, and Pod-to-Pod Paths)

## Intro: special guest + purpose

# 

- Host (PE) introduces **video #32** of the CKA 2024 series with special guest **Sai**.
- Sai is introduced as someone active in the **CNCF/Kubernetes space**, teaching via **Kubesimplify** and **Kubesimplify Hindi**.
- Sai introduces himself:
- Founder of Kubesimplify
- Works at **Loft Labs** as a Principal Developer Advocate
- Past roles: Field CTO at a company (mentions “coo”), and experience at Oracle, Walmart, HP
- Notes Kubernetes has completed **10 years** and continues evolving each release.
- Session goal: explain Kubernetes networking “in a practical way” and how things work internally.

## History lesson: why Kubernetes moved away from bundling Docker

### ### Docker (as an ecosystem, not just a runtime)

# 

- Early Kubernetes (when Kubernetes started): **Docker was the only container runtime** and was **bundled into Kubernetes**.
- As the ecosystem matured:
- other runtimes emerged (mentions **rkt/rocket**—“no more today”).
- bundling Docker inside Kubernetes became difficult.
- Around ~2016: the community moved toward **standardization** so Kubernetes wouldn’t have to bundle Docker.

### ### OCI (Open Container Initiative)

# 
- Sai explains OCI emerged to define standards/specs:
- *OCI provides specifications like “image spec” and “runtime spec,” and distribution aspects (where/how images are distributed).*
- Docker’s relationship to OCI:
- Docker existed before OCI and “was not OCI compliant immediately.”
- Docker contributed heavily and re-architected to align.

### Docker support removed in Kubernetes 1.24

# 
- Key milestone mentioned:
- In **Kubernetes 1.24**, “Docker support was removed” (described as the culmination of a long journey).

## Modern container-start workflow: Docker → containerd → shim → runc

### ### containerd, shim, runc

# 

- When you run `docker run`:
- request goes to **docker daemon**
- but *the daemon does not start the container directly*
- Docker uses **containerd**:
- *containerd “manages the lifecycle” of the container but “doesn’t start the container.”*
- containerd passes to a **shim**:
- shim also doesn’t start the container; it passes the request further.
- Low-level runtime:
- **runc** (OCI-compliant) starts the container.
- *runc starts the container and exits (short-lived process); shim maintains connectivity between the running container and runc/containerd layer.*

### ### WebAssembly (Wasm) via containerd shims

# 
- Mentions alternative runtimes:
- you can target different runtimes via shims (example: **runwasi shim**).
- *Instead of runc, a WebAssembly runtime can run WebAssembly workloads while using a similar workflow.*

## Kubernetes node-side execution: kubelet + CRI

### ### CRI (Container Runtime Interface)

# 

- Kubernetes flow described:
1. User issues `kubectl run` (or REST API call) to API server
2. Scheduler selects a node and updates the Pod spec
3. The request goes to **kubelet** on the selected node
4. kubelet uses **CRI** to talk to the container runtime
- Container runtime choices mentioned:
- **containerd** (called the default standard now; “more lightweight”)
- **CRI-O**
- Docker (supported via “Mirantis repository” path, per his wording)

## Why networking is pluggable: CNI and network add-ons

### ### CNI (Container Network Interface)

# 

- Sai emphasizes Kubernetes is “highly pluggable/optimizable”:
- choice of container runtime
- choice of networking solution
- Explains kubeadm installs:
- after `kubeadm init`, nodes can be **NotReady** because there is **no default networking implementation** bundled.
- Introduces CNI:
- *CNI is “the container network interface” that configures networking so every pod gets a unique IP and can communicate across nodes.*
- Key requirement:
- Every pod must have a **unique IP** (regardless of node).

### CNI spec vs implementations (analogy to CRI/OCI)

# 
- Sai frames a pattern:
- there is a **specification** (like CRI, OCI, CNI)
- there are **implementations** (like containerd, runc, networking plugins)
- CNI implementations mentioned:
- **Flannel**
- **Weave**
- **Calico**
- **Cilium** (he says he uses it for everything and recommends it; mentions rich ecosystem and built-in network policies)

### Host question: is CNI a minimum standard?

# 
- Host asks if it’s fair to say:
- CNI is a set of standards/bare minimum capabilities; implementations can add more.
- Sai confirms and points to the CNI spec:
- mentions operations like **ADD**, **DEL**, **CHECK**, **GC**, **VERSION**
- notes “plugin considerations” and that requirements are clearly defined.

### ### CNI plugins (bridge/vlan/loopback, etc.)

# 
- Sai notes CNI is plugin-based:
- mentions plugins like **bridge**, **vlan**, **loopback**
- Mentions there are videos showing how to write a CNI plugin and how multiple actions happen behind “attach IP.”

## Pod networking internals: multi-container pod and the pause container

### ### Pause container

# 

- Introduces a multi-container pod example: pod “shared namespace” with two containers:
- P1: BusyBox
- P2: Nginx
- Key behavior:
- Kubernetes creates **three containers**, not two:
- BusyBox container
- Nginx container
- **Pause container**
- Definition-style explanation (close paraphrase):
- *The pause container “holds the network namespace,” which enables the pod to have a single shared network namespace and therefore a single IP address.*
- Why containers inside pod share IP:
- network namespace is shared → containers can communicate via **localhost**.

### Namespaces referenced

# 
- Sai mentions containers are formed by multiple namespaces (examples he names):
- network namespace
- IPC
- UTS
- (and others; he references “six seven namespaces”)

## Traffic path: pod-to-pod on the same node (virtual ethernet pairs + bridge)

### ### veth pairs + bridge + ARP tables

# 

- Describes node/pod connectivity conceptually:
- Node has an ethernet interface.
- Each pod gets:
- an interface inside the pod namespace (often `eth0`, but naming depends on CNI)
- a corresponding **virtual ethernet (veth)** endpoint on the host (root namespace)
- A **bridge** on the host uses ARP tables (“arp tables and stuff”) to forward traffic to the right port/veth.
- Example flow described (Pod A → Pod B):
- traffic goes from pod `eth0` → host veth → bridge resolves destination → host veth to destination pod → destination pod.

## Practical walkthrough in a Kubernetes playground (KillerKoda)

### ### KillerKoda

# 

- Sai uses **KillerKoda** as a free Kubernetes playground.
- Notes the environment is a **two-node cluster**:
- control plane
- node1

### Create a pod and inspect where it runs

### ### kubectl apply / get pods -o wide

# 

- Applies a pod manifest (YAML referenced but not fully shown in transcript).
- Checks:
- `kubectl get pods -o wide`
- observes pod is running on **node1** and has a pod IP.

### SSH to the node to inspect namespaces and links

### ### SSH + iproute2 tools

# 

- Opens another terminal tab and SSHs into **node1**.
- Runs tools to inspect networking:
- `ip netns list` to list network namespaces
- greps for the pod/nginx-related process
- `lsns` to show namespaces associated with a process ID
- shows multiple namespaces (mnt, pid, ipc, uts, net, user, cgroup)
- enters a specific network namespace and runs:
- `ip link` / `ip addr`
- observes:
- loopback interface
- pod interface (often `eth0`-like), naming may differ by CNI.

### Calico-specific artifacts (interface naming)

### ### Calico

# 

- In this environment, Calico appears to create host-side links with **cali**-style naming.
- He notes:
- interface naming depends on CNI plugin (Calico vs others).

### Tie-back: exec into pod shows same pod interfaces

### ### kubectl exec + ip addr

# 

- Demonstrates equivalence:
- what you see by entering the node namespace aligns with what you see via `kubectl exec` inside the pod (e.g., `ip addr` showing loopback + pod interface).
- Key point:
- node-level inspection is where you can see the veth pairs and host networking details; `kubectl exec` is the Kubernetes-level view.

## Kubernetes networking building blocks called out

# 
- Sai summarizes key components involved:
- CNI assigns pod IP and creates links/veth pairs
- kube-proxy manages IP tables rules (“iptables rules managed by kube-proxy”)
- CoreDNS provides **service discovery** (resolving service names like `service.namespace.svc.cluster.local`)

## Pod-to-pod communication demo + network policies mention

### ### Pod IP communication

# 

- He runs another pod (e.g., `kubectl run nginx --image=nginx`).
- Confirms:
- each pod gets its own IP.
- Tests connectivity:
- from one pod to another via **pod IP** (tries curl; notes curl may not be installed in busybox, so uses nginx side).
- Concludes:
- pod-to-pod connectivity over IP is “default behavior.”

### ### NetworkPolicies

# 
- He notes:
- to restrict traffic between pods/namespaces/apps, use **NetworkPolicies**.
- calls NetworkPolicies important for Kubernetes certifications (CKA/CKS).

## Closing remarks

# 
- Sai and host wrap up:
- this session is about fundamentals and internal understanding; not necessarily direct exam questions.
- Host encourages:
- comment feedback
- subscribe to Sai’s channel for more Kubernetes/cloud-native content
- Host says links/resources will be shared via GitHub/repo and thanks Sai for the collaboration.

# 

#