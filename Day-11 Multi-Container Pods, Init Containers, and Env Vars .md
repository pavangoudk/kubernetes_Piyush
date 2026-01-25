# Kubernetes (CKA 2024, Video 11): Multi-Container Pods, Init Containers, and Env Vars (Demo)

## Series context + what this video covers

# 

- This is **video #11** in the **CKA 2024** series.
- Prior topics mentioned: Docker and Kubernetes fundamentals, **Pods**, **Deployments**, **Services**, and more.
- Focus of this video:
- **multi-container Pods**
- “a few other related concepts” (demo includes init containers + env vars)

## Engagement request (non-technical)

# 
- Like/comment target: **120 comments** and **28 likes** in 24 hours.
- Requests learners to share learnings on **LinkedIn** / **Twitter**.

## Concept: What is a multi-container Pod?

# 
- Starting point: a Pod running on a Kubernetes cluster with **one app container** (example: **nginx**).
- You can add **additional containers** to support the app container, such as:
- **init container** (initialization container)
- **sidecar container** (helper container)

### Init container behavior (as described)

# 
- *“An init container… runs before the app container to do certain task.”*
- Startup order:
1. Kubernetes receives the request to start the Pod
2. it **starts init container(s) first**
3. *“As soon as the init container completes… then only the app container will be started.”*
- Dependency relationship:
- *“Starting of app container is totally dependent on the init container.”*
- Resource model:
- *“It shares the resources of the Pod”* (CPU/memory/storage allocated to the Pod are shared among containers).
- Speaker also says it “works in conjunction” with the app container.

### Sidecar container (brief mention)

# 
- *“A sidecar container… runs all the time with the app container… provides certain input or it takes certain output… also referred to as a helper container.”*

### Visual Studio Code (editor)

# 
- Speaker uses **Visual Studio Code** to create and edit a YAML manifest file.

## Demo: Create a Pod manifest (baseline)

# 
- Creates a new file named `p.yml` (spelled in transcript like “P.L”).
- Writes a basic Pod manifest:
- `apiVersion: v1`
- `kind: Pod`
- `metadata` with name + labels (speaker says these should be familiar by now)
- `spec` with `containers` list

### busybox image (used for demo)

# 
- Main container uses **busybox**.
- Speaker describes busybox as:
- *“a generic image provided by kubernetes and maintained by the community”*
- does basic tasks and will be used often later

## Demo: Environment variables inside a Pod

# 
- In the container spec, adds `env` (list of key-value pairs):
- name: `first_name`
- value: `bu`
- Speaker calls it *“a key value pair”* and says env vars are *“within the scope of your pod.”*

### kubectl create + CrashLoopBackOff observation

# 
- Runs:
- `kubectl create -f p.yml`
- `kubectl get pods`
- Pod goes into **CrashLoopBackOff** because (as explained):
- busybox runs a task and exits (nothing keeps it running)

## Demo: Add an init container (waiting for a Service)

# 
- Deletes the Pod and continues.
- Adds `initContainers` at the same level as `containers`:
- init container name: `init-my-service`
- image: `busybox:1.28`

### Command/args pattern (init container)

# 
- Uses `command` and `args`:
- command format shown as `sh`, `-c` (speaker explains `-c` is used because you pass a command string)
- The init container runs a loop:
- **until** `nslookup myservice.default.svc.cluster.local` succeeds
- it echoes “waiting for service to be up”
- sleeps `2` seconds
- repeats until success

### Main container command (keep Pod running)

# 
- Adds a container `command` that:
- echoes “the app is running”
- then sleeps `3600` seconds (so it won’t exit / CrashLoop)
- Speaker highlights two equivalent styles:
- separating `command` and `args` vs combining them in one line

## Demo: Apply manifest, troubleshoot YAML, and observe init state

# 
- First `kubectl create -f p.yml` fails due to a YAML quoting mistake; speaker fixes an extra quote.
- After Pod creation:
- `READY: 0/1`
- `INIT: 0/1` (Pod stuck initializing)
- Uses:
- `kubectl describe pod my-app`
- Describe shows:
- init container created/started
- Pod stuck because service is not provisioned yet
- main Pod reason: **PodInitializing**
- Uses logs:
- `kubectl logs pod/my-app`
- notes: with multiple init containers, specify container with `-c <container-name>`
- Log output shows repeated:
- waiting message
- cannot resolve the service FQDN yet

### nslookup (technique)

# 
- Init container uses **nslookup** to check whether the Service name resolves via cluster DNS.
- The service FQDN pattern used: `myservice.default.svc.cluster.local`.

## Demo: Create Deployment + expose Service to unblock init container

# 
- Creates a deployment:
- `kubectl create deploy nginx-deploy --image=nginx --port=80`
- Exposes it as a service with the exact name expected by the init container:
- `kubectl expose deploy nginx-deploy --name=myservice --port=80`
- After some time:
- init completes
- main app container becomes ready
- Re-checks init logs; now `nslookup` returns an IP and the waiting stops.

## Demo: Verify env vars via exec

# 
- Uses `kubectl exec` to run an inline command:
- `printenv` to list env vars and confirm `first_name` exists
- Then execs into a shell (`sh`) and runs:
- `echo $first_name`
- Speaker notes:
- It defaults to the main app container
- You can exec into the init container for debugging using `-c`

## Demo: Multiple init containers + key learnings

# 
- Copies the init container block to add a second init container:
- name: `my-db`
- also does `nslookup` (for `mydb.default.svc.cluster.local`)
- Tries `kubectl apply -f p.yml` and gets an error:
- Pod update invalid: *“may not add or remove containers”*
- Workaround shown:
- delete and recreate the Pod to include additional init containers

### Observation: init completion gating

# 
- Pod shows `INIT: 1/2`:
- first init container completed
- second is still waiting for its service
- Key statement:
- *“for [the] main application pod… all the init containers inside that pod has to be… completed first.”*

## Demo: Create second Deployment + Service for the second init container

# 
- Creates another deployment (uses a Redis image per transcript) and exposes it:
- service name must match the YAML (`mydb`)
- After creating `mydb` service:
- waits; eventually Pod becomes running
- Mentions you can use `-w` (watch) to avoid re-running `get pods`.

## Close-out: practice + next video

# 
- Assignment is in a **GitHub repository** under a **day 11** folder (to practice multi-container Pods).
- Mentions a **Discord server** for help and community support.
- Next video topics: **DaemonSets**, **CronJobs**, and **Jobs** (other Kubernetes resource types).

#