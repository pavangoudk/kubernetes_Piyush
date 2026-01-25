# Kubernetes CK 2024 Day 17: Autoscaling (HPA, VPA, and More)

## Intro: what this video covers (and why)

# 

- Speaker introduces **video #17** in the CK 2024 series.
- Topic: **autoscaling**, including **HPA (Horizontal Pod Autoscaler)** and **VPA (Vertical Pod Autoscaler)**, plus the “overall concept of autoscaling.”
- Exam note: speaker says this is **not important for the CKA exam**, but is *important for beginners / deep understanding*, so it’s included anyway.
- Mentions engagement targets (comments/likes), then starts.

## Scaling basics (before “autoscaling”)

# 
- Speaker says: *“forget about auto scaling… forget about HPA VPA… let’s just start with what actually scaling is.”*
- Definition: *Scaling is “changing your servers or your workload to meet the demand.”*
- Scaling can be done **manually** or **automatically**.

### Manual scaling example (Deployments)

# 
- Refers back to Deployments and replicas:
- Deployments run **replicas** so “identical copies of a single pod can run… to serve the customer.”
- Manual approach described:
- You specify replica count.
- If you need more, you update the Deployment/ReplicaSet to the desired replicas.
- Why manual scaling breaks down:
- In production, you can’t monitor 24x7.
- With “thousands of nodes… thousands of pods,” manual scaling is “almost impossible,” inefficient, and not recommended.

## Why autoscaling exists

# 
- Need: something that scales **automatically** based on:
- increased demand / workload
- “utilization of resources on the server”

## Horizontal vs vertical autoscaling (pod examples)

### Horizontal pod autoscaling scenario

# 

- Setup: a node with **one pod** using **1 CPU and 1 memory**, users begin accessing the app.
- As users increase (1 → 2 → 3 → more):
- App can hit bottlenecks, stop responding, or show high latency.
- Concept introduced: **horizontal pod autoscaling**
- *“This will add identical pods in the deployment automatically based on the load on the server.”*
- Load examples: increased **CPU** or **memory**.
- Example policy: if average CPU reaches ~60%, “add one pod,” and keep adding as users increase.
- Speaker’s naming rationale:
- “horizontally adding the pods” + “done automatically” ⇒ **Horizontal Pod Autoscaling**.

### Vertical pod autoscaling scenario

# 
- Setup: another workload where the app can tolerate restarts/slowness (not mission-critical).
- Concept introduced: **vertical autoscaling**
- Instead of adding pods, you “automatically resize” the pod to a bigger one.
- Example: from **1 CPU/1 memory** to **10 CPUs / 10 GB memory**.
- Tradeoff emphasized:
- The pod “will be replaced by a bigger pod,” and “it will require a restart.”
- Works when app is “stateless in nature” and can afford downtime.

## Mapping the same concepts to cloud (VM analogy)

# 
- Horizontal autoscaling (cloud example):
- AWS Auto Scaling Group (ASG): start with 1 instance, desired 3, add instances when CPU increases.
- Vertical autoscaling (cloud example):
- Replace an EC2 instance with a “bigger machine” based on CPU/memory utilization.

## Organizing autoscaling types in Kubernetes (workload vs infra)

# 
- Speaker lays out two major types:
- **Horizontal** (also called *scale out / scale in*)
- Add replicas on increased load; remove replicas on decreased load (cost/efficiency).
- **Vertical** (also called *scale up / scale down*)
- Resize machines/apps to meet demand; resize down when demand drops.
- Adds another axis: autoscaling can apply to:
- **Workload (pods)**
- **Infrastructure (nodes)**

### Workload (pods) autoscaling

# 
- Horizontal on workload: add pods to a deployment when CPU hits a threshold.
- Done by **HPA**.
- Important dependency: HPA “only takes metrics from the metrics server.”
- Vertical on workload: resize pod resources.
- Done by **VPA** (speaker calls it a separate project).

### Infrastructure (nodes) autoscaling

# 
- Horizontal on infra: add more nodes when demand increases.
- Managed by **Cluster Autoscaler**.
- Vertical-ish infra capability mentioned: **Node Autoprovisioning** (e.g., add node pools).

### Availability / where these come from

# 
- Speaker states:
- **Only HPA** is available “within Kubernetes by default.”
- **VPA** is “a separate project on GitHub” you install and set up.
- **Cluster Autoscaler / Node autoprovisioning** typically come with managed cloud Kubernetes offerings (AKS, EKS, GKE), not local setups like kind/minikube.

## Other autoscaling styles mentioned

### ### Event-based autoscaling

# 

- Horizontal/vertical scaling usually reacts to metrics (CPU, memory, disk).
- Sometimes you scale based on **events**, e.g.:
- lots of “500” errors
- requests rejected with a specific error code
- Speaker mentions a CNCF project tool: **KEDA** (pronounced as speaker says “kaada”), described as popular.

### ### Cron / schedule-based autoscaling

# 
- Also mentioned briefly as another approach (“and so on”).

## Demo focus: HPA (load test walkthrough)

### Kubernetes version note

# 

- Speaker says the CKA lab version updated to **Kubernetes 1.30**, and they upgraded their environment to 1.30 (mentions June 3).
- Suggests upgrading to match the exam version, though hands-on practice is mostly the same.

### Prerequisite: Metrics Server

# 
- Reiterates: HPA needs **metrics server** to expose CPU/memory metrics.
- Confirms metrics server pod is running in **kube-system**.

### ### Multi-object YAML (Deployment + Service in one file)

# 
- Speaker uses a YAML in the Day 17 folder containing **two objects**:
1. Deployment
2. Service
- Technique explained:
- Use `---` (three dashes) to separate multiple Kubernetes objects in one YAML file.
- You can include “as many as resources” in one YAML this way.

### Deployment details (as described)

# 
- Deployment name: **php-apache**
- Image: an **HPA example** image from an “official” Kubernetes repository (speaker says it’s public).
- Container port: **80**
- Requests/limits set for CPU:
- Request: **200m** CPU
- Limit: **500m** CPU
- Speaker defines “m”:
- *“m is for milli”* and “200m is… 2/10 of 1 CPU.”
- Service:
- Exposes on **port 80**
- Uses selector/labels matching the deployment (speaker references prior coverage).

### Apply the YAML

# 
- Applies the file; since replicas aren’t specified, it creates **1 pod by default**.
- Confirms service exists.

## Creating the HPA object (imperative approach shown)

### ### kubectl autoscale

# 

- Speaker shows using an imperative command to create HPA:
- Target: the deployment `php-apache`
- CPU percent target: `50` (average CPU utilization threshold)
- Min replicas: `1`
- Max replicas: `10`
- Explains how HPA evaluates:
- It “keeps on monitoring your workload”
- Default interval is “every 15 seconds” (speaker says configurable via controller manager).
- If average CPU goes beyond 50% for a short period, it adds replicas.
- When it drops, it scales down toward the minimum.

### Checking HPA status

# 
- `kubectl get hpa` initially shows CPU as **unknown**, then later shows a low % (e.g., 1%) with target 50%.

## Generating load to trigger scaling

### ### Load generator (busybox + wget loop)

# 

- Uses a second terminal to run a pod that generates load:
- `kubectl run -it load-generator`
- Image: **busybox**
- Runs a loop: `while sleep 1; do wget ...; done` (speaker describes frequent requests)
- Speaker notes it prints responses (“OK”), generating traffic to increase CPU load.

### Observing autoscaling behavior

# 
- Uses `--watch` to see changes over time.
- Observations stated in sequence:
- Pods increase (speaker notes it added multiple pods; sees ~5 running at one point).
- HPA shows CPU utilization rising above target (mentions values like 37%, then 117% at different moments).
- Replicas climb (speaker references 5, then 7 replicas while it stabilizes).

## Stopping load and scale-down

# 
- Interrupts the load generator (Ctrl+C), stopping traffic.
- Notes scale-down is **not immediate**; it “will take some time,” possibly “a few minutes.”
- Expectation: replicas gradually return to the **minimum of 1**.

## Closing remarks

# 
- Repeats: autoscaling isn’t part of CKA, but understanding HPA/VPA/autoscaling is valuable.
- Next video topic preview: **liveness and readiness probes**.
- Wraps up with like/share/subscribe and thanks.