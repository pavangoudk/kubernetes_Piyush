# Build a Multi-Node Kubernetes Cluster on VMs with kubeadm (CK 2024, Video 27)

## Intro (what you’ll do)

# 

- Speaker introduces **video #27** and sets the goal: build a **multi-node Kubernetes cluster** on **virtual machines** using **kubeadm**.
- This is a full hands-on from scratch, with a Day 27 GitHub task to practice the steps.

## Installation options (choose based on purpose)

# 
- Speaker says the list isn’t exhaustive; it’s meant to guide choices.

### ### kind / k3s / minikube (local / quick POC)

# 
- Use when you want a fast local setup for testing/POC.
- These options typically use virtualization or containers as “nodes.”
- Not recommended for production due to limitations and lack of true high availability (it’s ultimately one host).

### ### Managed Kubernetes services (cloud-managed control plane)

# 
- If you want the provider to manage control plane components:
- **EKS (Elastic Kubernetes Service)** on AWS
- **AKS (Azure Kubernetes Service)**
- **GKE (Google Kubernetes Engine)**
- Speaker’s definition: *“Self-managed service means… all the control plane components… are managed by a cloud provider… it’s not managed by you.”*

### ### Self-managed Kubernetes (you manage everything)

# 
- If you want to manage control plane + node maintenance (patching, upgrades, etc.):
- Vagrant/VirtualBox (not ideal for production unless the underlying environment is HA)

### Multipass (noted as helpful on Mac; needs sufficient RAM/CPU)

# - - 
- Cloud VMs (spin up multiple VMs and build the cluster manually)
- Bare metal / VMware managed services (mentioned as another route)
- Speaker planned Multipass but dropped it due to resource requirements; proceeds with cloud VMs instead.
- Uses AWS due to having credits, but emphasizes the steps aren’t AWS-specific.

## High-level plan (what the demo will do)

# 
- Provision **3 VMs**:
1. 1 control plane node
2. 2 worker nodes
- Open required ports (speaker references Kubernetes “ports and protocols” docs and a diagram).
- Common setup on **all nodes**:
1. Disable swap
2. Update kernel parameters
3. Install container runtime, runc, CNI plugins
4. Install Kubernetes utilities: kubeadm, kubelet, kubectl
- Control plane-only steps:
1. Initialize control plane with kubeadm
- Speaker’s framing: *Running init “tells this machine to behave as a master.”*
2. Install a CNI network add-on (Calico/Cilium/etc.)
- Worker-only steps:
1. Join workers using the join token output from the control plane
- *The join token is produced by kubeadm init and is run on worker nodes to join them to the control plane.*

## Network ports and firewalling (what must be reachable)

# 
- Speaker highlights key ports and intent:
- Admin access to Kubernetes API server: **6443**
- kubelet API: **10250** (on nodes)
- kube-proxy: **10256** (on workers)
- Control plane component ports (scheduler, controller-manager, etcd ports) should be limited to internal network access (VPC/private network), not open to the world.
- NodePort range for exposing apps: **30000–32767** (workers)
- SSH for admin access: **22**
- Notes Kubernetes docs don’t mention SSH, but speaker says you need it to access nodes.

## ### AWS Security Groups (demo environment setup)

# 
- Speaker creates two security groups:
- One for control plane
- One for worker nodes
- Control plane rules described:
- Control plane component port ranges allowed only within the VPC CIDR
- etcd ports allowed only within the VPC
- SSH (22) allowed only from speaker’s IP
- API server port 6443 allowed within the VPC
- Worker rules described:
- kube-proxy (10256), kubelet (10250), NodePort range (30000–32767) per design
- SSH (22) only from speaker’s IP
- Launches instances:
- 1 “master” VM and 2 worker VMs (later renames them worker01/worker02)
- Uses Ubuntu image and a size with at least ~2GB RAM / 2 vCPU (speaker chooses a medium instance type)
- Uses an SSH key pair for login

## SSH into control plane and start node prerequisites

# 
- Speaker SSHs into the control plane VM and begins executing the common steps.

## ### Disable swap + kernel parameters (all nodes)

# 
- Runs steps to disable swap.
- Updates kernel parameters and says to verify the required values end up as **1**.

## ### containerd (container runtime)

# 
- Speaker installs **containerd** (notes Docker runtime support was removed after Kubernetes 1.24).
- Sets it up as a **systemd service** and generates a config file.
- Calls out a key config adjustment:
- Sets the system cgroup parameter appropriately (false → true in their flow).
- Verifies containerd is active/running via service status checks.

## ### runc

# 
- Downloads and installs **runc**.

## ### CNI plugins (binaries)

# 
- Downloads and installs **CNI plugin** binaries.

## ### kubeadm / kubelet / kubectl (Kubernetes utilities)

# 
- Installs kubeadm, kubelet, and kubectl, explicitly using Kubernetes **1.30** to avoid version mismatch issues.
- Verifies versions installed.

## ### crictl (containerd client)

# 
- Speaker explains crictl as a runtime client:
- *“Docker is the client… for containerd we’ll be using crictl.”*
- Used for listing/running container operations similarly to Docker commands.

## Initialize the control plane

## ### kubeadm

# 

- Runs kubeadm initialization with:
- Control plane advertised API server address
- Pod network CIDR
- Notes a warning can be ignored.
- Defines pre-flight checks:
- *“It is doing some validation… those validations are called pre-flight checks.”*
- kubeadm generates:
- Static pod manifests for control plane components (API server, controller-manager, scheduler, etc.)
- Default RBAC objects
- Output includes:
- A **kubeadm join** command with token and hash (used on workers)
- Instructions to set up kubeconfig for kubectl usage on the control plane

## Set up kubeconfig for kubectl on control plane

# 
- Speaker runs the post-init steps that:
- Create `~/.kube/`
- Copy the admin kubeconfig into the default location
- Fix ownership/permissions so kubectl can use it
- Verifies node shows up, but status is **NotReady** until a CNI is installed.

## Install networking add-on (Calico) and troubleshoot CIDR mismatch

## ### Calico / Tigera operator

# 

- Speaker installs Calico using operator-based installation (Tigera operator appears).
- Observes issues:
- CoreDNS pods stuck in ContainerCreating
- Calico pods not coming up
- Troubleshoots by checking Tigera operator logs and finds the root cause:
- Calico IP pool CIDR (default) does not match the pod network CIDR configured during kubeadm init.
- Speaker’s explanation: kubeadm was initialized with one pod CIDR, but Calico expects another by default → mismatch.

### Fix: reset and re-initialize

## ### kubeadm reset

# 

- Speaker resets the control plane using kubeadm reset:
- *This “uninstalls… control plane components” and unregisters the node as control plane.*
- Re-runs kubeadm init with a pod network CIDR that matches Calico’s expected range.
- Re-runs kubeconfig setup and reinstalls Calico.
- Now Calico pods and CoreDNS come up; control plane node becomes **Ready**.

## Worker node setup + join process

# 
- Speaker repeats the common installation steps on the worker node(s) (swap off, kernel params, containerd, runc, CNI binaries, kubeadm/kubelet/kubectl).
- Runs into a crictl permission issue and adjusts permissions; notes it works with sudo.

### Mistake and correction (node mix-up)

# 
- Speaker realizes they SSH’d into the wrong VM earlier and effectively set up control plane on a node they’d named as a worker due to IP/name mismatch.
- Corrects naming/understanding and proceeds with installing the worker properly.

### Join worker to cluster

# 
- Runs the kubeadm join command on the worker (requires sudo).
- Speaker notes: if join hangs or times out, it’s typically a **security group / firewall** connectivity issue (worker can’t reach control plane).

## Using kubectl from worker nodes (kubeconfig copy)

# 
- On the worker node, kubectl initially fails because default kubeconfig isn’t present.
- Speaker attempts copying a kubeconfig from system paths, hits permission errors, and then takes the practical route:
- Copies the kubeconfig content from the control plane’s `~/.kube/config` to the worker node’s default kubeconfig location.
- Fixes directory/file permissions.
- After that, kubectl on the worker can talk to the control plane API server without extra flags.
- Repeats the join and kubeconfig steps for the second worker node; all nodes show **Ready**.

## Wrap-up recap (speaker’s final summary)

# 
- What they did end-to-end:
1. Created 3 cloud VMs (1 control plane, 2 workers)
2. Opened required ports via firewall/security groups
3. On all nodes: disabled swap, updated kernel params, installed container runtime + runc + CNI binaries, installed kubeadm/kubelet/kubectl
4. On control plane: kubeadm init, then installed Calico (and fixed CIDR mismatch via kubeadm reset + re-init)
5. On workers: kubeadm join
6. Copied kubeconfig from control plane to workers so kubectl works there too
- Notes they’ll use this kubeadm-based cluster for future videos (kind was used earlier for beginner convenience).
- Mentions a future “hard way” install (manual binaries) as a good-to-know, though not required for the exam.
- Reminder: clean up cloud resources (stop/terminate), note storage costs even when stopped, and mentions creating an AMI/golden image as an option.

# 

