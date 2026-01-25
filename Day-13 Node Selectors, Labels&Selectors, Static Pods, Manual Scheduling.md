# Kubernetes (CKA 2024, Video 13): Node Selectors, Labels/Selectors, Static Pods, Manual Scheduling

## What this video covers + engagement notes

# 

- Video #13 in the **CKA 2024** series.
- Topics the speaker lists for this video:
- **node selectors**
- **labels and selectors**
- **static pods**
- **manual scheduling**
- Guidance: if you haven’t watched earlier videos and aren’t comfortable with prior concepts + hands-on, the speaker recommends watching them first.
- Target: **130 comments** and **130 likes** in 24 hours.
- Speaker will share an assignment for home-lab practice (day 13 folder).

## Kubernetes architecture refresher (scheduler’s role)

# 
- Uses the standard cluster diagram:
- Control plane on the left, worker nodes on the right.
- Mentions components like **kubelet**, **kube-proxy**, and workloads on worker nodes.
- Scheduler’s core function (flow described):
1. Client (**kubectl**) requests a new Pod (example: an nginx pod).
2. **API server** creates an entry in the **etcd** database.
3. **Scheduler** continuously monitors Pods (including ones “yet to run”) and decides which node to place them on (based on scheduling algorithm and factors discussed later).
4. Scheduler provides details to the API server.
5. API server instructs **kubelet** on the chosen node to run the Pod.
6. Kubelet provisions the Pod, reports back to API server.
7. etcd is updated and response returns to the client.
- Key point: *“who takes the decision… and who actually send the request to kubelet… is scheduler.”*

## Static Pods (why scheduler itself runs)

# 
- Speaker notes the “chicken-and-egg” issue: scheduler is a Pod—so who schedules the scheduler?
- Introduces static pods:
- *“there is a concept… called static pods”*
- *“these static pods are… the control plane components that are not managed by scheduler.”*
- Instead, **kubelet** manages them:
- *“kubelet… is responsible for checking… manifests… in a particular directory… [and] it will spin up the pod.”*

### kubectl (viewing scheduler pod)

# 
- Checks scheduler pod in `kube-system`:
- `kubectl get pods -n kube-system | grep scheduler`
- Confirms it runs on the control-plane node and shows it with wide output:
- `kubectl get pods -n kube-system -o wide | grep scheduler`

## kind cluster note + accessing the “node”

# 
- This lab uses **kind** (*Kubernetes in Docker*):
- nodes are Docker containers.
- to “enter” a node, use **docker exec** (not SSH).
- Finds the control-plane container via:
- `docker ps | grep control-plane`
- Enters it with:
- `docker exec -it <container> bash` (speaker corrects earlier command usage)
- Inside the node, checks kubelet is running (`ps -ef`).

### Static pod manifests directory

# 
- Speaker states the default directory:
- `/etc/kubernetes/manifests`
- Lists files there:
- `etcd`
- `kube-apiserver`
- `kube-controller-manager`
- `kube-scheduler`

## Demo: stopping scheduler by removing its static manifest

# 
- Moves/removes `kube-scheduler.yaml` from `/etc/kubernetes/manifests`.
- Result (observed via kubectl):
- Scheduler pod disappears from `kube-system`.
- Other control plane components still run (CoreDNS, etcd, apiserver, controller, kube-proxy, etc.).
- Impact explained:
- Control plane doesn’t fully “go down,” but **new pod scheduling** won’t happen.
- Existing pods continue unless they fail.

### Demo: new pod stuck Pending without scheduler

# 
- Creates a pod:
- `kubectl run nginx --image=nginx`
- Pod stays **Pending**.
- `kubectl describe pod nginx`:
- Node field is blank (not scheduled).

### Demo: restoring scheduler

# 
- Moves `kube-scheduler.yaml` back into `/etc/kubernetes/manifests`.
- Kubelet detects it and starts scheduler again.
- After scheduler returns, scheduler pod is running again in `kube-system`.

## Manual scheduling (bypassing scheduler)

# 
- Speaker frames how scheduler chooses candidates:
- Scheduler scans Pods in Pending and looks for ones **without** a `nodeName` specified.
- If a Pod already has `nodeName`, scheduler treats it as “not my job.”
- Manual scheduling method shown:
1. Generate a pod manifest from `kubectl run`:
- `kubectl run nginx --image=nginx -o yaml > pod.yaml`
2. Edit YAML and add:
- `spec.nodeName: <worker-node-name>`
3. Note: *once a pod is scheduled, you can’t move it node-to-node; you must delete and recreate it.*

### Demo: manual scheduling works even when scheduler is down

# 
- Stops scheduler again (removes scheduler manifest).
- Applies the pod YAML with `nodeName`:
- `kubectl apply -f pod.yaml`
- Pod becomes **Running** even though scheduler is not running.
- Verifies placement:
- `kubectl get pods -o wide` shows it running on the specified worker node.
- Restores scheduler manifest again and waits until it shows `1/1` ready.

## Labels and selectors (refresher + deeper details)

# 
- Speaker says labels/selectors were seen earlier, but adds more detail now.
- Definition-style description:
- *“label… is attached as part of the metadata and it is helpful in filtering that particular resource.”*
- Applies to many Kubernetes objects (pods, services, daemonsets, deployments, etc.).

### Deployment selector matching

# 
- Example structure explained:
- Deployment has labels in `metadata` (deployment-level labels).
- Pod template has labels in `spec.template.metadata.labels` (pod labels).
- Deployment `selector.matchLabels` matches the pod labels so the deployment manages the right pods.

### Adding multiple labels to pods (demo)

# 
- Starts from an existing pod (nginx pod has label `run=nginx`).
- Adds additional labels (examples used):
- `tier=frontend`
- `type=app1` (speaker also mentions “you can add multiple labels”)
- Applies changes; sees patch/conflict behavior and uses `--force` to apply.
- Shows labels on pods:
- `kubectl get pods --show-labels`

### Create a second pod with different labels (demo)

# 
- Copies YAML to a second file (e.g., `pod2.yml`).
- Changes:
- removes `nodeName` (let scheduler manage it)
- pod name becomes `redis`
- label `tier=backend`
- image becomes `redis`

### Filtering with selectors

# 
- Uses label selector to retrieve pods:
- `kubectl get pods --selector tier=frontend` → returns nginx pod
- `kubectl get pods --selector tier=backend` → returns redis pod
- Use case framing:
- helps group “identical” resources and retrieve/filter for actions or reporting.

## Annotations vs labels, and labels vs namespaces

# 
- Opens pod manifest via:
- `kubectl edit pod <name>`
- Notes `metadata.annotations`:
- *“annotation is not labels… it will store additional details… messages… information related to the object itself.”*
- Example annotation mentioned: last applied configuration (helps controller understand changes/rollback), timestamps, etc.
- Labels vs namespaces:
- Namespaces are *“logical separation of the resources in a group”* (examples: test, prod, default, kube-system, kube-public, kube-node-lease, kube-storage mentioned).
- Purpose includes reducing accidental modifications/deletions and “to secure it.”
- Labels are like “tags” applied to resources for grouping/filtering.

### Amazon filter analogy (labels/selectors)

# 
- Products have attributes (category, rating, feedback, etc.) → analogous to labels.
- Filters on the site → analogous to selectors used to retrieve matching resources.

## Wrap-up + assignment

# 
- Speaker asks again to hit the comment/like targets.
- Assignment will be in **day 13** folder.
- Support options: comments + Discord channel; encourages helping others and sharing the playlist.

# 

# 

#