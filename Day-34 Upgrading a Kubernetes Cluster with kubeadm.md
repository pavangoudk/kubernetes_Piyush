# CKA 2024 Video 34: Upgrading a Kubernetes Cluster with kubeadm (Control Plane + Workers)

## Intro: what this video covers + engagement target

# - - Host introduces **Video #34** in the CKA 2024 series.
- Topic: *“how we can upgrade a Kubernetes cluster end to end”*:
- control plane components
- worker nodes
- hands-on upgrade process
- Engagement goal: **300 likes + 300 comments** in 24 hours.

## Pre-req concept: node maintenance with drain / cordon / uncordon

# - Host explains these concepts first because they’re needed for worker node upgrades.

### Example scenario (4-node cluster)

# - - Cluster: **1 master + 3 worker nodes**
- Workloads:
- **nginx deployment** with **3 replicas**, one per node (shown as part of a single Deployment).
- **MySQL pod** as a **standalone pod** (not managed by Deployment/ReplicaSet/ReplicationController), running on **worker node 1**.

### ### kubectl drain (node evacuation + unschedulable)

# - - Maintenance goal: take **worker node 1** out for maintenance / replacement.
- Command concept shown:
- `kubectl drain <node-name>`
- Host explanation:
- *“draining the node means we are emptying it… evicting all the workloads running on the node.”*
- What happens to the **nginx** pod replica on worker node 1:
- it is evicted
- the Deployment controller notices replicas drop from 3 → 2
- a **new pod** is created on another available node (not on worker node 1).

### ### Cordon (implicit in drain)

# - - Host emphasizes drain does **two things**:
1. evicts workloads (**drain**)
2. marks node **unschedulable** (**cordon**)
- Definition-style phrasing:
- *“cordoning means mark it as unschedulable.”*

### Important caveat: standalone pods get deleted permanently

# - - The standalone **MySQL pod** is evicted during drain.
- Because it’s not managed by a controller:
- *it “will be gone forever… deleted… would not be recovered.”*
- Implication stated:
- data/config in that pod would be lost.

### Scheduling behavior after drain

# - - If you create a new pod (example: “redis pod”) after draining worker node 1:
- it will schedule on other nodes, **not** on worker node 1 (still unschedulable).

### ### kubectl uncordon (make node schedulable again)

# - - After maintenance/replacement:
- run `kubectl uncordon <node-name>`
- Host clarifies:
- uncordon makes the node schedulable again for **new** pods.
- it does **not** automatically move previously rescheduled pods back to the node.

## Kubernetes versioning basics (major / minor / patch)

# - Host explains Kubernetes version format before upgrading.

- Example format: `1.30.2`
- **Major**: `1`
- **Minor**: `30`
- **Patch**: `2`
- Notes:
- Major has stayed at **1** so far.
- Minor releases are frequent (every “2–3 months” per host).
- Patch releases are even more frequent (bug fixes / small enhancements).

### Upgrade rule: one minor version at a time

# - - Host stresses Kubernetes policy:
- you cannot jump from `1.28` directly to `1.30`.
- must go `1.28 → 1.29 → 1.30`.

### Support window: only 3 latest minor versions

# - - Host explains Kubernetes supports only the **latest 3 minor versions** at a time.
- When a new minor version appears, the oldest supported minor falls out of support.
- Definition-style phrasing:
- *“out of support does not mean you won’t be able to use it… it means there won’t be any new bug fixes [or] enhancements.”*

## Upgrade order: control plane first, then workers

# - Host references the kubeadm upgrade doc’s high-level sequence:

1. upgrade the **primary control plane node**
2. upgrade **additional control plane nodes** (for HA clusters)
3. upgrade **worker nodes**

### Why “additional control plane nodes” matter (HA)

# - - Current lab: **1 control plane + multiple workers** (not HA).
- HA concept:
- multiple masters + a load balancer in front of API servers
- if one master is down, others still serve API traffic
- Clarifies impact if control plane is down:
- administrative operations (API server, controller actions, kubectl calls) are impacted
- existing workloads keep running unless there’s an issue

## Worker upgrade strategies (three approaches)

# - Host outlines upgrade strategies and selects rolling update for the demo.

### 1) All-at-once upgrade

# - - Drain all workers, upgrade them, then uncordon.
- Not recommended for production because nodes become unavailable simultaneously → user impact.

### 2) Rolling upgrade (one node at a time)

# - - Drain worker 1 → upgrade → uncordon
- Repeat worker 2, then worker 3
- Users not impacted because other nodes stay available.
- Host says this is what they’ll do.

### 3) Blue/green (new infra, then replace)

# - - Provision new worker nodes already running the target version (e.g., `1.30`).
- Join them to the cluster, then delete old nodes.
- Pros:
- faster, low impact (seconds)
- Cons:
- requires extra infrastructure (harder on on-prem due to procurement/approvals)
- easier with managed services (GKE/AKS/EKS/OpenShift mentioned)

## Start of hands-on: initial cluster version check

### ### kubectl

# - - Host SSHs into the **master node** and runs:
- `kubectl get nodes`
- Initially sees cluster at **v1.30.2** (he notes that’s the kubelet-reported version and each component has its own versioning).

### Control plane component versions (release artifacts context)

# - - Host references Kubernetes downloads/changelog and notes control plane components/images:
- kube-apiserver, controller-manager, scheduler, kube-proxy, kubectl, etc.
- Mentions:
- kubeadm is upgraded separately (not part of the same bundle in his explanation).

### Component skew rule (compatibility)

# - - Host explains allowable version skew:
- If `kube-apiserver` is at version **X**:
- scheduler/controller-manager can be at **X-1**
- kubelet/kubectl can be at **X-2**
- Adds: ideally keep everything on the same version.

## Realization: cluster already at latest 1.30 → creates new 1.29 cluster

### ### kubeadm upgrade plan

# - - On Ubuntu, host checks available kubeadm versions via apt.
- Runs:
- `kubeadm upgrade plan`
- It reports the cluster is already at latest within the 1.30 series, so there’s nothing meaningful to upgrade for the demo.
- Host pauses and **recreates a cluster** at **v1.29.6** to demonstrate upgrading to **1.30**.

## Upgrade execution (from 1.29.6 → 1.30.2) using kubeadm docs

### Step 0: confirm current state

### ### kubectl

# - - After reinstall:
- `kubectl get nodes`
- Cluster shows:
- 1 master, 2 workers
- version **1.29.6**

### Step 1: update package repository to target minor (1.30)

### ### apt repository file (Ubuntu)

# - - Confirms using community package repo (`pkgs.k8s.io`) via a command from docs.
- Edits:
- `/etc/apt/sources.list.d/kubernetes.list`
- Changes repo minor from `1.29` → `1.30`
- Runs:
- `sudo apt update`
- `sudo apt-cache madison kubeadm` (to list available kubeadm versions)

### Step 2: upgrade kubeadm on the control plane

### ### apt install kubeadm

# - - Uses the docs command pattern:
- `sudo apt-get install -y kubeadm=<version>`
- Notes a copy/paste character issue from documentation (a comma/character mismatch), then reruns successfully.
- Verifies:
- `kubeadm version` shows **1.30.x**

### Step 3: run kubeadm upgrade plan

### ### kubeadm upgrade plan

# - - Shows:
- current cluster version: **1.29.6**
- kubeadm version: **1.30.0** (as installed)
- target cluster version: **1.30.2**
- Notes:
- kubelet must be upgraded manually after control plane upgrade.
- CoreDNS and etcd have their own versions (separate projects), already latest in his output.

### Step 4: apply the control plane upgrade

### ### kubeadm upgrade apply

# - - Runs:
- `sudo kubeadm upgrade apply v1.30.2`
- Initially hits permission issues and resolves by:
- adjusting kubeconfig file permissions (`chmod` shown)
- running with sudo (needs root)
- Upgrade completes with success message:
- cluster upgraded to **1.30.2**

### Step 5: drain the control plane node (for kubelet/kubectl upgrade steps)

### ### kubectl drain

# - - Runs drain on the master node.
- Observes behavior:
- drain cordons the node (scheduling disabled)
- Encounters daemonset restriction:
- cannot delete daemonset-managed pods (Calico, CSI node driver mentioned)
- Uses:
- `--ignore-daemonsets`
- He notes he had to rerun because it got stuck, then it completes.
- `kubectl get nodes` shows master is **SchedulingDisabled**
- Confirms control plane static pods remain (controller-manager, scheduler, kube-proxy listed), but workloads are moved off.

### Step 6: upgrade kubelet and kubectl on the control plane

### ### apt install kubelet + kubectl

# - - Runs apt install for:
- `kubelet=<target-version>`
- `kubectl=<target-version>`
- Restarts kubelet service:
- `sudo systemctl daemon-reload`
- `sudo systemctl restart kubelet`
- Checks:
- `systemctl status kubelet`
- `kubectl get nodes` now shows master node version updated (kubelet reports it).

### Step 7: uncordon the master

### ### kubectl uncordon

# - - Runs:
- `kubectl uncordon <master-node>`
- Node returns to **Ready** and schedulable.

## Worker node upgrades (rolling, one at a time)

### Worker 1

# - 1. Update repo on worker to `1.30`
- edit `/etc/apt/sources.list.d/kubernetes.list`
2. Upgrade kubeadm on worker
3. Run:
- `kubeadm upgrade node` *(host clarifies this upgrades local kubeadm config, not kubelet itself)*
4. Drain worker from control plane (worker lacked kubeconfig)
- `kubectl drain worker01 --ignore-daemonsets`
5. Upgrade kubelet + kubectl on the worker node via apt
6. Restart kubelet on the worker:
- `systemctl daemon-reload`
- `systemctl restart kubelet`
7. Uncordon from control plane:
- `kubectl uncordon worker01`

- Host notices version didn’t update until kubelet restart on the worker (then it reflects correctly).

### Worker 2

# - - Repeats the same sequence:
- update repo to 1.30
- upgrade kubeadm
- `kubeadm upgrade node`
- drain from control plane
- upgrade kubelet/kubectl on worker 2
- restart kubelet
- uncordon
- Final check:
- `kubectl get nodes` shows all nodes at **1.30.2** and **Ready**.

## Final verification + wrap-up

# - - Host verifies versions:
- `kubectl version` (client is 1.30.x)
- `kubeadm version` (1.30.2)
- `kubelet --version` (1.30.2)
- Wrap-up message:
- upgrade requires:
- upgrade control plane components
- upgrade kubeadm/kubectl/kubelet
- drain/uncordon during node work
- Mentions GitHub task:
- practice drain/cordon/uncordon
- install a specific Kubernetes version and upgrade it following docs
- Support channels:
- YouTube comments
- Discord community
- Closes with encouragement to hit the likes/comments target and “happy learning.”

# 

# 

#