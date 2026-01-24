- ## Kubernetes Services in CKA 2024 (Video 9): NodePort, ClusterIP, LoadBalancer, ExternalName (+ kind lab setup)

## 0) What the video will cover (and engagement target)

- Topic: Kubernetes **Services**—specifically **ClusterIP**, **NodePort**, **ExternalName**, **LoadBalancer** (and more).
- Speaker planned to cover **Namespaces** too.
- Engagement target: **120 comments** and **250 likes** in 24 hours.

## 1) Where we left off (context from the previous video)

- Previously covered: **ReplicaSet**, **ReplicationController**, **Deployment**.
- Current state: a **Deployment** running **nginx** as a front-end application on **multiple nodes** (speaker mentions four Pods earlier).
- Problem: the Deployment is **not exposed externally**; it’s only reachable “from the cluster” / by accessing nodes.

## 2) Why Services are needed (concept + example scenario)

- The speaker introduces Services as the way to make a front-end app available externally and to connect app layers internally.

### Example application layout (speaker’s scenario)

- **Front end**: running in a container inside a Pod (or a Deployment—speaker says either is fine for the example).
- **Back end**: a **Node.js** application.
- **External data source**: backend connects to it to read/write data.

### What a Service enables in this scenario

- External user accesses the **Service**, and the Service forwards to the **front-end Pod(s)** and returns responses.
- Internally, Services are used to connect:
- front end → back end
- back end → external data source

*Definition (speaker): **“Our service makes our application loosely coupled.”***

*Definition (speaker, close paraphrase): *It ensures Pods are “listening on a certain port” and accessible “only what we have specified.”***

## 3) Service types covered

- The speaker lists:
- **ClusterIP**
- **NodePort**
- **ExternalName**
- **LoadBalancer**
- Then starts with **NodePort** first.

### ### Method: NodePort Service

#### What NodePort is (and why it’s used)

*Definition (speaker): **“This is a service on which the application will be exposed on a particular port and that port is called node port.”***

#### Ports explained (NodePort vs Port vs TargetPort)

The speaker uses an example NodePort: **30001**.

- **NodePort** (external):
- Range given: **30000 to 32767**
- This is the port external users hit (e.g., browser/local system).
- **Port** (service/internal):
- Used by “other services or other applications that are running in the cluster.”
- **TargetPort** (pod/application):
- *Definition (speaker): **“Target port is the port on which our application pod is listening on.”***
- Not exposed externally (in the speaker’s explanation); NodePort traffic is redirected to TargetPort.

The speaker summarizes that understanding the difference between **nodePort**, **targetPort**, and **port** is critical.

#### Multiple Pods / multiple nodes behavior (speaker’s explanation)

- The same NodePort concept works whether:
- one Pod on one node, or
- multiple Pods across multiple nodes (e.g., behind a Deployment).
- To access externally, the speaker describes using:
- **node IP : nodePort**
- Traffic distribution:
- The speaker says it “will redirect the traffic on a round robin basis” and “internally load balance” across the Pods behind the Service.

### ### Tool: VS Code (editor workflow)

- The speaker switches to VS Code to create a Service manifest file.
- They already have nginx Pods running behind a Deployment and reuse them.

### ### Method: Writing a Service manifest (YAML structure)

- The speaker follows the usual top-level fields:
- **apiVersion**, **kind**, **metadata**, **spec**
- Notes the manifest is **case sensitive** (they fix casing issues during troubleshooting).

### ### Tool: kubectl (cluster interaction)

#### Checking Service API version/kind

- The speaker uses kubectl’s “explain” capability to confirm:
- Service kind = “Service”
- apiVersion = “v1”

#### NodePort Service YAML fields they build (in order they describe)

1. **metadata**
- name: a NodePort service name (e.g., “nodeport-svc”)
- labels: `env: demo`
2. **spec**
- type: NodePort (case-sensitive; they correct it later)
- ports (list/array; they emphasize using `-`)
- nodePort: 30001 (they initially make a digit mistake and fix it)
- port: 80
- targetPort: 80
- selector:
- points to the Pods via their labels (they retrieve labels first, then match `env=demo`)

#### Common mistakes they hit (and how they fix them)

1. Selector structure mistake:
- They initially try something like “matchLabels” under selector and correct it to the simpler selector format.
2. Case sensitivity:
- They set the service type incorrectly; Kubernetes rejects it and lists supported values.
- They fix the casing for **NodePort**.
3. NodePort range validation error:
- Kubernetes rejects the nodePort because it’s out of range.
- Root cause: they added an extra zero by mistake.
- They correct it to **30001**, then the Service is created.

#### Verifying the Service exists

- They list Services and confirm the NodePort Service appears with:
- a ClusterIP assigned
- the port mapping showing port 80 and nodePort 30001

## 4) First attempt to access the NodePort Service (and why it fails in kind)

- The speaker tries to access the app using:
- first a Pod IP (then realizes it should be the node IP),
- then retrieves node info and/or uses “describe pod” to find the node,
- then tries node IP + nodePort.
- Even after creating the Service, access still fails.

### ### Tool: kind (Kubernetes in Docker) + documentation

- The speaker concludes it’s time to read the kind docs.
- Key point:
- *Definition (speaker, close paraphrase): kind “does not expose your port to the outside world.”*
- The fix is a kind-specific step:
- “Mapping ports to the host machine” using **extra port mappings** in the kind cluster config.
- They note you **can’t change** this on an existing kind cluster, so they **recreate** the cluster.

## 5) Recreating the kind cluster with port mapping (to make NodePort reachable)

### ### Method: kind cluster config (extraPortMappings)

- They create a kind cluster config YAML (similar to what they used in video 6):
- 1 control-plane node
- 2 worker nodes
- plus an **extraPortMappings** entry mapping:
- containerPort **30001** ↔ hostPort **30001**
- They create a new cluster (named “CK cluster 3” in the narration).
- They delete the earlier cluster first because the name already exists.

**Exam/real-cluster note (speaker):**

- This extra step is **only for kind**.
- In an exam lab / on-prem / cloud server Kubernetes cluster, the NodePort flow should work without this special port mapping step.

## 6) Re-deploying nginx and re-creating the NodePort Service (after cluster recreation)

- The speaker applies the existing Deployment YAML from the prior video (day 8 folder):
- Deployment name: “nginx-deploy”
- label: `env: demo`
- image: nginx (latest)
- containerPort: 80
- replicas: 3
- Then they apply the NodePort Service YAML again.

### Verification steps they show

- Confirm Pods are running (3 Pods).
- Confirm Service exists and describe it to see details (type, ports, etc.).

### Access test (success)

- They test:
- **localhost:30001**
- It returns the nginx page (“Welcome to nginx”), confirming the NodePort exposure works in kind after port mapping.

### Quick recap (speaker)

- Why recreate: kind doesn’t expose NodePort externally by default.
- What changed: added **extraPortMappings** for 30001.
- In non-kind environments:
- the app should be reachable via the **node IP** + **nodePort**
- in kind, they use **localhost** due to port mapping

## 7) Convenience: kubectl alias + shell completion

### ### Technique: shell alias for kubectl

- The speaker sets an alias so they can type a short command instead of kubectl repeatedly (e.g., using “k”).
- They mention placing it in a shell profile and sourcing it.
- They note: you don’t need to do this for the exam.

### ### Technique: bash completion / tab autocomplete

- The speaker points to cheat sheet / quick reference commands that enable completion.
- Result: pressing **Tab** helps autocomplete commands.

### ### Method: ClusterIP Service

#### Why ClusterIP

- The speaker explains a typical multi-tier app again (front end, back end, database).
- Problem:
- Pods have internal IPs, but *“as soon as the pod restart the IP gets changed.”*
- They demonstrate this by:
- checking a Pod IP,
- deleting the Pod,
- and showing the new Pod has a different IP.

#### ClusterIP solution (speaker’s explanation)

- Create a Service of type **ClusterIP** so other Pods/services can use:
- the Service **name**, or
- the Service **ClusterIP**
- They mention the default “kubernetes” Service appears when listing Services.

#### Creating the ClusterIP YAML (what they do)

- They copy the NodePort YAML into a new file for ClusterIP.
- Changes:
- remove the explicit type field (they say default service type is ClusterIP, so it still becomes ClusterIP)
- remove nodePort
- keep port and targetPort
- keep selector (same label)

#### Verifying ClusterIP Service

- They list Services and confirm the new ClusterIP Service exists:
- no external IP (because it’s internal-only)
- exposed on port 80

#### Endpoints explained

*Definition (speaker): **“Endpoint is nothing but the IP of the pods on which the service is listening to.”***

- They show endpoints correspond to the Pod IPs.
- They show how to list endpoints (Endpoints/EP).
- They note the NodePort Service and ClusterIP Service can show the **same endpoints** because both select the same Deployment.

### ### Method: LoadBalancer Service

#### Why LoadBalancer (speaker’s scenario)

- When Pods run on different nodes, you don’t want to give users many IPs.
- For larger apps (e.g., “50 nodes”), giving dozens of node IPs is not practical.
- Desired: one simple address like **“[myapp.com](http://myapp.com/)”**.

#### How it works in Kubernetes (speaker’s explanation)

- In cloud environments (AWS/Azure/GCP):
- you provision an external load balancer,
- then create a Service of type **LoadBalancer** that ties into it.

#### Demo outcome in kind

- They create a LoadBalancer Service manifest (type set to LoadBalancer).
- It does **not** get an external IP because:
- they haven’t provisioned an external load balancer in this environment.
- They mention kind has an optional approach:
- a “cloud provider kind” component that can simulate a load balancer,
- but they do not set it up to avoid confusion.
- Result: it behaves like it has a random NodePort assigned.

### ### Method: ExternalName Service

- The speaker describes ExternalName as mapping the Service to a DNS name:
- type: ExternalName
- instead of labels/selectors, you map it to an external DNS (e.g., a database DNS)
- Purpose:
- internal services can refer to that DNS via the Service name.

## 8) Creating Services imperatively (instead of writing YAML)

### ### Tool: Kubernetes quick reference guide (formerly “cheat sheet”)

- The speaker notes the cheat sheet is now referred to as a “quick reference guide.”
- They show that you can create a Service from the command line using an “expose” workflow (e.g., exposing a Deployment with port/targetPort) instead of writing YAML.
- They recommend practicing this because it’s handy in the exam and saves time.

## 9) Namespaces deferred + what’s next

- The speaker planned Namespaces in this video but defers it since Services took time.
- Next video will cover:
- multi-container Pods
- commands and arguments
- namespaces

## 10) Wrap-up and assignment

- The speaker mentions there’s a small assignment in the GitHub repo (day 9 folder).
- Encourages hands-on practice and reaching out via Discord/comments.
- Closes with a reminder to like/share/comment/subscribe and says they’ll return tomorrow with the next video.
