Local Kubernetes Setup for CKA 2024: Installing KIND + kubectl, Multi-Cluster Contexts (Video 6)
================================================================================================

1) Intro: why no cloud provider (yet)
-------------------------------------

*   Speaker introduces **video #6** in the **CK 2024** series.
    
*   A common question they received: which cloud provider will be used (AKS/EKS/GKE, etc.)?
    
*   Speaker’s answer: **none** (for now), because learning should follow prerequisites.
    
    *   Before Kubernetes: learn **Docker/containers** (already done).
        
    *   Before managed Kubernetes (AKS/EKS/GKE): learn **Kubernetes itself via local installation**.
        
*   Reason for local install:
    
    *   Gives “maximum learning” and hands-on practice.
        
    *   With managed services, you “won’t even get access to the control plane node,” and troubleshooting scenarios provide “very minimum learning” because much is managed by the provider.
        

**Section summary:** The series will first build Kubernetes fundamentals on a local cluster before moving to managed cloud services, to maximize hands-on learning and troubleshooting exposure.

2) Tool choice for local Kubernetes: KIND (and other options)
-------------------------------------------------------------

*   Speaker says there are multiple ways to run Kubernetes locally:
    
    *   Minikube
        
    *   k3s
        
    *   k3d
        
    *   kind
        
    *   (and others)
        
*   Chosen tool: **kind**.
    
    *   _“Kind is Kubernetes in Docker… one of the most popular local installation of Kubernetes.”_
        
*   Purpose for the series:
    
    *   Install Kubernetes locally and set up a cluster to use for hands-on tasks throughout the course.
        
    *   Also cover exam-relevant items.
        

### Tools Used: kind (Kubernetes in Docker)

*   Speaker uses the kind docs/reference page and describes kind as:
    
    *   _“A tool that is used to run local Kubernetes cluster using Docker container nodes.”_
        
    *   Meaning: it spins up **Docker containers**, and each container acts as a **node** (control plane / worker nodes).
        

**Section summary:** The course standardizes on kind for local clusters, while acknowledging other popular local Kubernetes options.

3) Prerequisites (as listed by the speaker)
-------------------------------------------

*   Prereqs from the kind page:
    
    *   **Go 1.16** installed
        
    *   One of: **Docker**, **Podman**, or **nerdctl**
        
*   Speaker already has Docker installed from earlier videos.
    

### Tools Mentioned: Docker / Podman / nerdctl

*   Listed as acceptable container runtimes/tools to support kind’s operation.
    

**Section summary:** To use kind, you need Go and a container runtime/tooling such as Docker, Podman, or nerdctl.

4) Installing kind (speaker’s environment: Mac)
-----------------------------------------------

### Tools Used: Homebrew (brew)

*   Speaker chooses “installing with package manager.”
    
*   On Mac: runs **brew install kind**.
    
*   Notes the install is simple; then clears the screen after it finishes.
    

### Tools Mentioned: Chocolatey (choco)

*   For Windows, the speaker notes you can install kind via:
    
    *   choco install kind
        
*   _“Chocolatey is a Windows packet manager”_ used to install software/packages.
    

**Section summary:** kind can be installed via package managers; the speaker installs it on Mac with Homebrew.

5) Creating a kind cluster (pinning the Kubernetes version)
-----------------------------------------------------------

### Methods Explained: kind create cluster basics

*   Speaker points to docs: creating a cluster can be as simple as:
    
    *   kind create cluster
        
*   Optional flags mentioned:
    
    *   Specify a different node image via --image
        
    *   Set the cluster name via --name
        

### Version selection approach (exam alignment)

*   Speaker records the video in **May 2024** and states:
    
    *   **Kubernetes 1.29** is the version used “in all of the CKA exams” at that time.
        
*   Advises viewers:
    
    *   Check the **CNCF exam guide** for the current exam version.
        
    *   Differences between versions are usually “very minute,” but using the exam-aligned version is best.
        

### Tools Used: kind node images (release notes)

*   Speaker opens kind release notes and notes:
    
    *   If you don’t specify version, it uses the **latest** (they see 1.30 as latest).
        
    *   They locate the **1.29.x** prebuilt image line and copy it.
        

### Command executed (as described)

*   Runs kind create cluster with:
    
    *   \--image set to the copied **kindest/node:v1.29.4** image (with a SHA)
        
    *   \--name cka-cluster-1
        
*   Output notes:
    
    *   Pulling node image
        
    *   Preparing nodes
        
    *   Writing configuration
        
    *   Starting control plane
        
*   Tooling note from output:
    
    *   Mentions a kubectl cluster-info command for the context.
        

**Section summary:** The speaker creates a kind cluster named cka-cluster-1 using a Kubernetes v1.29.4 kind node image to match exam expectations.

6) Verifying the cluster and introducing kubectl
------------------------------------------------

### Tools Used: kubectl

*   Speaker runs a cluster-info command (as suggested by kind output) to verify the cluster:
    
    *   Gets control plane endpoint/port.
        
    *   Sees **CoreDNS** running.
        
*   CoreDNS quick explanation:
    
    *   _“A service… that provides you DNS functionality within Kubernetes… like a local DNS server.”_
        
*   Strong prerequisite callout:
    
    *   You must have **kubectl** installed; it will be used for the entire series and for any Kubernetes cluster (EKS/AKS/GKE/on-prem).
        

### Installing kubectl (high-level, as described)

*   Speaker suggests searching for kubectl install docs (example: Linux install via curl download + verify + install).
    
*   Verify kubectl:
    
    *   kubectl version --client
        
*   Version note:
    
    *   Speaker’s kubectl client version differs from cluster version because kubectl was installed earlier.
        
    *   _Ideally you should use the same version as the Kubernetes version._
        

**Section summary:** kubectl is the primary CLI for all Kubernetes interaction; the speaker verifies both cluster connectivity and kubectl client version.

7) Inspecting nodes: confirming this is a single-node cluster
-------------------------------------------------------------

### Tools Used: kubectl get nodes

*   Speaker runs kubectl get nodes and explains how it works (linking back to prior architecture video):
    
    *   kubectl sends request to API server
        
    *   API server validates/authenticates
        
    *   API server fetches results from etcd and returns them
        
*   Output observed:
    
    *   One node: cka-cluster-1-control-plane
        
    *   Status: Ready
        
    *   Role: control-plane
        
    *   Version: **v1.29.4**
        
*   Interpretation:
    
    *   This is a **single-node cluster**; no separate worker nodes yet.
        
*   Speaker’s desired next step:
    
    *   Control plane and workers should be separate nodes, so they’ll create a multi-node cluster.
        

**Section summary:** The first kind cluster is a single node (control plane only), so the speaker proceeds to configure and create a multi-node cluster.

8) Creating a multi-node kind cluster using a config YAML
---------------------------------------------------------

### Methods Explained: kind cluster configuration via YAML

*   Speaker goes to docs section “configuring your kind cluster.”
    
*   Notes default behavior:
    
    *   Single node where control plane and worker are in the same node/container.
        
*   Uses a YAML config that defines nodes/roles:
    
    *   kind: Cluster
        
    *   apiVersion
        
    *   nodes:
        
        *   role: control-plane
            
        *   role: worker
            
        *   role: worker
            
*   Speaker creates a new local file config.yml, opens in vi insert mode, pastes YAML, then saves with :wq!.
    

### Tools Mentioned: YAML (configuration format)

*   Speaker says YAML prerequisites/details will be covered in the next video; for now treat it as a config file.
    

### Creating the second cluster (multi-node)

*   Re-runs kind create command with changes:
    
    *   New name: cka-cluster-2
        
    *   Same --image (v1.29.4)
        
    *   Adds --config config.yml
        
*   Cluster creation output includes:
    
    *   Preparing nodes
        
    *   Starting control plane
        
    *   Installing **CNI** (network plugin)
        
    *   Installing storage class
        
    *   Joining worker nodes
        
*   Speaker explains “joining”:
    
    *   Worker nodes must be joined to the control plane so they become part of the same cluster (context set among them).
        

### Tools Mentioned: CNI (Container Network Interface) plugin

*   Mentioned as installed during cluster creation (“network plugin”).
    

### Confirming multi-node

*   Runs kubectl get nodes again; now sees:
    
    *   1 control plane node
        
    *   2 worker nodes
        
    *   All Ready
        
    *   All v1.29.4
        

**Section summary:** A YAML config enables a 3-node kind cluster (1 control plane + 2 workers), and the output shows networking (CNI) and worker-node joining steps.

9) Managing multiple clusters: contexts and the CKA exam workflow
-----------------------------------------------------------------

### The “tricky situation”

*   With two clusters created, the speaker asks:
    
    *   How do you know which cluster kubectl is querying?
        
*   They attempted a command and realized it wasn’t correct (then pivots to reference docs).
    

### Tools Used: Kubernetes docs “cheat sheet”

*   Speaker goes to kubernetes.io/docs and searches for **cheat sheet**.
    
*   Uses the cheat sheet as a quick reference for commands.
    

### Tools Used: kubectl config get-contexts

*   Speaker runs kubectl config get-contexts:
    
    *   Sees two contexts:
        
        *   kind-cka-cluster-1
            
        *   kind-cka-cluster-2
            
    *   A \* indicates the **current context**.
        

### Tools Used: kubectl config use-context

*   Uses kubectl config use-context to switch:
    
    *   Switches to kind-cka-cluster-1 and confirms kubectl get nodes shows one node.
        
    *   Switching to cluster 2 shows three nodes.
        

### Exam tip (explicit)

*   In the CKA exam:
    
    *   You must **switch to the correct context** before each task.
        
    *   The exam provides the command to run; you should copy/paste it.
        
    *   If you forget to switch context, you may do correct steps in the wrong cluster and fail the task.
        

### Docs allowed in exam (as described)

*   Speaker says two sites are accessible during the exam:
    
    *   kubernetes.io/docs (and subdomains)
        
    *   kubernetes.io/blog (and subdomains)
        
*   Practical advice:
    
    *   Don’t rely on searching constantly; it wastes time.
        
    *   Practice commands enough to be fast; use docs for long commands if needed.
        

**Section summary:** Context switching is a core CKA exam habit—kubectl config get-contexts to see options and kubectl config use-context before every task to ensure you’re operating in the right cluster.

10) Wrap-up and what’s next
---------------------------

*   Speaker frames this video as a “warm-up” / prerequisite setup.
    
*   Next video topics:
    
    *   Create a simple pod
        
    *   Imperative vs declarative approaches
        
    *   YAML basics (format, writing from scratch, validating correctness)
        
*   Encourages viewers to complete comment/like targets and share the video.
