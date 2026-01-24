Kubernetes Architecture (CKA 2024 — Video 5): Control Plane, Worker Nodes, and Request Flow
===========================================================================================

1) Intro: video focus and expectations
--------------------------------------

*   Speaker introduces **video #5** in the **CKA 2024** series.
    
*   Goal of the video: Kubernetes architecture “in depth,” focusing on:
    
    *   Control plane components
        
    *   Why each component is needed
        
    *   How they work together
        
*   Speaker references prior coverage:
    
    *   Container fundamentals, container architecture
        
    *   Hands-on Docker demos
        
*   Engagement targets:
    
    *   **150 comments** in 24 hours
        
    *   **200 likes** in 24 hours
        

**Section summary:** This video moves from Docker/container basics into Kubernetes cluster architecture, with emphasis on control plane components and how requests flow.

2) High-level Kubernetes architecture: control plane vs worker nodes
--------------------------------------------------------------------

*   Diagram structure:
    
    *   Left: **Control plane (master node)**
        
    *   Right: **Worker nodes** (multiple)
        
*   **Node definition**:
    
    *   _“Node is nothing but a virtual machine.”_
        
    *   Kubernetes “node” is essentially another name for a VM hosting components/workloads.
        
*   **Control plane (master node) definition + analogy**:
    
    *   _“A control plane or a master node is a virtual machine or a node that host many administrative components.”_
        
    *   Analogy: control plane is like a **board of directors**—gives instructions, doesn’t do the ground work.
        
*   Worker nodes:
    
    *   Where the “actual work is happening.”
        
    *   Containers (workloads) run on worker nodes, coordinated by control plane components.
        

**Section summary:** A Kubernetes cluster is framed as control-plane VMs running admin/control components and worker-node VMs running the actual workloads.

3) Pods: how Kubernetes runs containers
---------------------------------------

*   Speaker points to a pod on the worker node.
    

### Concept Defined: Pod

*   _“We cannot run the container in Kubernetes just in its own shell… we have to encapsulate that in something called as a pod.”_
    
*   Analogy:
    
    *   Like a baby in the womb needing a sack for protection; the **pod is that sack**.
        
*   Pod contents:
    
    *   A pod can have **one or more containers**.
        
    *   Typical best practice mentioned: ideally **one container per pod**, but sometimes multiple (helper/monitoring agents/init containers—details later).
        
*   Pod importance:
    
*   _“Pod is the smallest deployable unit in Kubernetes.”_
    
*   Speaker mentions other objects exist (deployments, replicas, services) but keeps focus on pods for this video.
    

**Section summary:** Pods are the smallest deployable unit and are the wrapper Kubernetes uses to run one (typically) or more containers together.

4) Control plane components (what they are and what they do)
------------------------------------------------------------

*   Speaker names the components collectively:
    
    *   API server, scheduler, etcd, controller manager are “control plane components.”
        

### Component: API Server

*   _“API server is the center of control plane… any incoming request from the client will first reach to API server.”_
    
*   Role:
    
    *   Main entry point into the cluster.
        
    *   Receives external requests and interacts with other components “on its behalf.”
        

### Component: Scheduler

*   Purpose:
    
    *   _“Helps you schedule your workload.”_
        
*   Flow described:
    
    *   Receives a request from the API server (e.g., schedule a pod).
        
    *   Finds a suitable node based on constraints like CPU, memory, disk, plus “requests and limits” (more later).
        
    *   Selects the best node for the pod.
        

### Component: Controller Manager

*   Described as a combination of multiple controllers:
    
    *   Node controller, namespace controller, deployment controller, etc.
        
*   Purpose:
    
    *   Ensures controllers are running properly.
        
    *   Monitors Kubernetes objects/workloads and keeps them healthy.
        
    *   Example:
        
        *   If a pod goes down, it detects that and keeps restarting it via the controller behavior.
            

### Concept Defined: etcd

*   _“Etcd is nothing but a key value data store.”_
    

#### Explanation: key-value store vs relational database (RDBMS)

*   RDBMS described as rows/columns with a fixed schema (example: employee table with fields like ID, name, age, sex).
    
*   If you need a new field (e.g., address), you must alter the table because:
    
    *   _“Relational database… have a fixed schema and every record… has to follow that schema.”_
        
*   Key-value store described as NoSQL and schema-less:
    
    *   Data stored in a document format (speaker says generally JSON-like).
        
    *   Stored as key/value pairs (key + value repeated).
        

#### What etcd stores (cluster data)

*   etcd stores “every single information about the cluster,” including:
    
    *   Cluster info/state
        
    *   Node details
        
    *   Pods
        
    *   Configuration
        
    *   Secrets
        
    *   “Every other relevant data”
        
*   Update behavior:
    
    *   Any cluster change applied by the API server is “instantly updated” in etcd.
        
*   Access rule (as described):
    
    *   _“Only API server will interact with the etcd database… and it will only have the authority to apply those changes.”_
        
    *   API server also retrieves data from etcd for queries (e.g., how many pods are running).
        
*   Availability note:
    
    *   Control plane components (including etcd) must be available “all the time.”
        
    *   Speaker says later videos will go deeper and include hands-on and failure scenarios.
        

**Section summary:** API server is the gateway, scheduler picks nodes, controller manager reconciles desired vs actual state, and etcd is the key-value source of truth for cluster state (read/write via API server).

5) Worker node components: kubelet and kube-proxy
-------------------------------------------------

### Component: Kubelet

*   _“Kubelet… is a node based agent”_ that receives instructions from the API server.
    
*   Example action:
    
    *   If instructed to delete a pod, kubelet deletes it and reports back to the API server.
        
*   Communication role:
    
    *   _“It enabled the communication between worker node and the control plane node.”_
        
*   Flow link:
    
    *   After kubelet executes an action, API server updates etcd accordingly (as described).
        

### Component: Kube-proxy

*   Networking role on the node:
    
    *   _“Enables the networking within the node… allows your pods to communicate with each other.”_
        
    *   Creates **iptables rules** enabling pod-to-pod networking.
        
    *   Enables pods and services to communicate.
        

**Section summary:** Kubelet executes control-plane instructions on each node; kube-proxy implements node-level networking via rules (iptables) to enable connectivity among pods and services.

6) End-to-end request flow example: creating a pod
--------------------------------------------------

*   Speaker walks through a “create pod” request with arrows and steps.
    

### Tools Mentioned: kubectl

*   _“Kubectl is a type of client that helps you interact with the cluster and its control plane components.”_
    
*   Used by a user (admin/DevOps) to send requests.
    

### Step-by-step flow (as described, in sequence)

1.  **User** uses **kubectl** to send a request to the **API server**.
    
2.  **API server** performs checks:
    
    *   Authenticates the request (user permissions)
        
    *   Validates the request (supported/valid)
        
3.  API server sends the change to **etcd**:
    
    *   etcd doesn’t create the pod itself; it creates an entry that the pod is created/requested.
        
4.  **etcd** responds to API server that the entry is created.
    
5.  **Scheduler** (running continuously) notices a pod needs scheduling:
    
    *   Finds a suitable node
        
    *   Informs API server of the node choice / scheduling decision metadata
        
6.  **API server** contacts **kubelet** on the selected worker node:
    
    *   Instructs kubelet to schedule/create the pod on that node
        
7.  **Kubelet** creates the pod and responds back to API server.
    
8.  **API server** updates **etcd** that the pod has been created.
    
9.  **API server** returns the final response to the **user**.
    

**Section summary:** The create-pod path is shown as: kubectl → API server (auth/validate) → etcd entry → scheduler picks node → API server → kubelet creates pod → API server updates etcd → response to user.

7) End-to-end request flow example: querying pods (read path)
-------------------------------------------------------------

*   Example: user asks how many pods are running in a namespace.
    
*   Flow described:
    
    1.  API server receives request, authenticates/validates.
        
    2.  API server retrieves the information from **etcd**.
        
    3.  API server returns results to the user.
        
*   Key point:
    
    *   API server doesn’t need to “check the cluster” because etcd already stores the details.
        

**Section summary:** Read requests are served by the API server pulling current state from etcd and responding to the client.

8) Wrap-up and what’s next
--------------------------

*   Speaker reiterates the goal: understand the purpose of control plane components and worker node components (kube-proxy, kubelet) and the flow.
    
*   Promises later deep-dives:
    
    *   Each component in detail
        
    *   Failure scenario simulations (to learn troubleshooting and “which component is responsible”)
        
*   Support options:
    
    *   Comments
        
    *   Discord community channel for “cka 2024 help”
        
*   Closes by asking viewers to meet the comment/like targets and says next video is coming soon.
