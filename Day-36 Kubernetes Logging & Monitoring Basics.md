# CKA 2024 Video 36: Kubernetes Logging & Monitoring Basics (Metrics Server + crictl Troubleshooting)

## Intro: why this video now (before troubleshooting series)

# - - Host (Push/P) introduces **Video #36** in the CKA 2024 series.
- Purpose: a short “basic concepts” video on **logging and monitoring in Kubernetes**, because upcoming videos focus on troubleshooting:
- application failures
- control plane troubleshooting
- worker node troubleshooting
- cluster component issues
- Engagement target: **150 likes** and **100 comments** in 24 hours.

## Kubernetes monitoring: not built-in → install Metrics Server

### ### Metrics Server

# - - Host recalls the autoscaling video (VPA/HPA) and states:
- *Kubernetes “does not come with inbuilt monitoring.”*
- Solution used earlier: install an add-on called **metrics-server** (CNCF project).
- What it provides:
- metrics for monitoring/alerting tools, e.g.:
- CPU utilization
- memory utilization
- Installation options mentioned:
- apply the provided **YAML** from the project repo
- or install via **Helm**

### Installing metrics-server (demo flow)

### ### kubectl

# - - On a cluster with **3 nodes**, host runs:
- `kubectl apply -f <metrics-server.yaml>`
- Checks it in kube-system:
- `kubectl get pods -n kube-system`
- Uses “watch” style waiting for it to come up.

## How metrics flow through the cluster (node → kubelet → metrics-server → API)

# - Host explains the data path conceptually with a node diagram.

### ### kubelet (node agent)

# - - kubelet runs on each node and:
- manages containers at node level (start/stop/restart)
- communicates between worker node and control plane

### ### cAdvisor

# - - Host introduces cAdvisor and explains its role:
- *cAdvisor “collect[s] the metrics” and “aggregate[s] the metrics from container runtime.”*
- then it sends those metrics to **kubelet**
- Emphasis:
- cAdvisor does not directly expose/forward to control plane; kubelet is the authorized node agent.

### ### Metrics API (served by metrics-server)

# - - Flow described:
1. cAdvisor collects metrics (node + pod/container CPU/memory)
2. sends to kubelet
3. kubelet sends to **metrics-server**
4. metrics-server exposes them via **Metrics API**
5. API server consumes it so commands like `kubectl top` can retrieve it

## Troubleshooting metrics-server readiness + certificate issue

# - Host’s metrics-server pod initially fails readiness.

### ### kubectl describe / logs

# - - Host checks:
- `kubectl describe pod -n kube-system <metrics-server-pod>`
- Sees readiness probe failure:
- status code **500**
- `kubectl top node` fails with:
- “metrics API not available”
- Checks logs:
- `kubectl logs -n kube-system <metrics-server-pod>`
- Error indicates certificate verification problems:
- cannot verify kubelet/node certificate
- “no metrics to serve” / certificate validation failure messages

### Root cause and fix

# - - Host concludes it’s related to kubelet certificates not being signed/validated as expected.
- Fix approach shown:
- add an argument to metrics-server container to **disable certificate validation** (the “TLS insecure” style flag referenced from earlier autoscaling video notes).
- Steps he demonstrates:
1. deletes metrics-server deployment to reset:
- `kubectl delete deploy metrics-server -n kube-system`
2. reapplies YAML
3. observes same cert failure
4. edits deployment and adds the missing container argument:
- `kubectl edit deployment metrics-server -n kube-system`
- adds the insecure TLS flag under container args
5. pod recreates and eventually becomes Ready
- He notes:
- In the exam, metrics-server is typically already installed.

### Practical troubleshooting recap (commands used)

# - - Host summarizes the basic pattern:
- use `kubectl describe` to check events/probes
- use `kubectl logs` to inspect stdout/stderr
- if multiple containers exist, specify container name in logs command

## Kubernetes logging: logs are stdout/stderr unless you ship them elsewhere

### ### kubectl logs (stdout/stderr)

# - - Host explains:
- `kubectl logs` shows what the container emitted to **STDOUT/STDERR**.
- Limitation stated:
- logs aren’t automatically aggregated in a third-party system.
- Production expectation:
- as admin/DevOps, you ship logs to tools such as:
- Splunk
- ELK (open-source example)
- then you can do aggregation, charts, alerts, “business analytics,” and “advanced observability.”

## When Docker isn’t available: use crictl for containerd troubleshooting

# - Host shifts to troubleshooting without Docker (common in newer Kubernetes).

### Runtime change reminder

# - - Host notes:
- from Kubernetes **1.24+**, default runtime moved from Docker to **containerd**.
- Therefore:
- `docker ps` may not exist on nodes.
- use **crictl** instead.

### ### crictl (container runtime CLI)

# - - Host compares:
- Docker: `docker ps`, `docker ps -a`
- containerd: `crictl ps`, `crictl ps -a`
- He hits a permission error and changes socket/file permissions (chmod shown) to proceed.

## Scenario: API server down → kubectl fails → debug with crictl

# - Host demonstrates why crictl matters when kubectl can’t talk to the API server.

### Break API server (static pod manifest move)

### ### /etc/kubernetes/manifests (static pods)

# - - Host goes to `/etc/kubernetes/manifests`
- Moves the API server manifest out of the directory (static pod stops).
- After that:
- `kubectl get nodes` fails:
- “connection to the server was refused” (API server not available)

### Use crictl to confirm API server container is missing

### ### crictl ps

# - - Runs `crictl ps | grep apiserver` and sees it’s not running.
- Lists other control plane containers still running:
- scheduler, controller-manager, kube-proxy, coreDNS, Calico, etcd, CSI node driver (as mentioned)

### Restore API server

# - - Moves the manifest back (uses sudo due to permissions).
- Re-checks:
- `crictl ps | grep apiserver` shows API server running again.
- Then kubectl works again.

## Additional crictl use cases: pull images, list objects, view logs

### Pulling images to test node connectivity/registry access

### ### crictl pull

# - - Example commands shown:
- `crictl pull nginx`
- `crictl pull redis`
- Purpose stated:
- troubleshoot image pull issues (networking, registry reachability, container runtime issues)

### crictl docs reference + differences from Docker

# - - Host opens a “debugging with crictl” doc and notes:
- list pods via `crictl pods` (instead of kubectl)
- list images via `crictl images` (like `docker images`)
- Difference highlighted:
- running containers is more complex than `docker run`:
- you create JSON configs (pod/container config) and then run/create via crictl.

### Viewing logs with crictl (container-level, not pod-level)

### ### crictl logs

# - - Host checks help and notes:
- crictl logs fetches logs at the **container** level.
- Workflow demonstrated:
1. `crictl ps` to get the **container ID**
2. `sudo crictl logs <container-id>` (sudo needed for permissions)
3. tail logs in real time:
- `sudo crictl logs -f <container-id>`
- He points out it’s similar to `docker logs -f`.

## Wrap-up: what’s next

# - - Host summarizes what this video covered:
- metrics-server basics + quick troubleshooting
- how Kubernetes logs work (stdout/stderr + need external aggregation)
- crictl basics for debugging when kubectl/Docker aren’t available
- Next video preview:
- **application failure troubleshooting**
- a sample “three-tier” app with multiple services/deployments/pods
- Closing: asks viewers to hit like/comment targets and share the series.

# 

# 

#