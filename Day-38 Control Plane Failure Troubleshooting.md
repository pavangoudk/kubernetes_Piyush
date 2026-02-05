# CKA 2024 Video 38: Control Plane Failure Troubleshooting (API Server, Scheduler, Controller Manager)

## Intro: why control plane troubleshooting matters

# - - Host (P) introduces **Video #38** in the CKA 2024 series.
- Focus: *“common control plane failure scenarios”* that a Kubernetes admin must troubleshoot so the cluster stays healthy.
- Context: **CKA exam-specific** scenarios.
- Engagement target: **200 likes** and **100 comments** in 24 hours.

## Scenario 1: `kubectl get nodes` fails → API server not reachable

### ### kubectl

# - - First check: `kubectl get nodes`
- Error received:
- connection refused to **kube-apiserver** endpoint on **port 6443**
- Host ties this to Kubernetes architecture:
- *kube-apiserver is the “first point of contact” whenever you run kubectl / any client interacts with the cluster.*
- Troubleshooting starting point: kube-apiserver.

### ### containerd / crictl (instead of Docker)

# - - Host notes you might usually check containers with `docker ps`, but:
- Kubernetes **1.30**
- since **1.24+**, default runtime is **containerd**, not Docker
- Docker isn’t installed (and won’t be in the exam)
- Use **crictl** (client for containerd):
- `crictl ps` to list running containers
- Host doesn’t see kube-apiserver in the list
- Uses grep:
- `crictl ps | grep api` (only Calico-related matches, no apiserver)

### Check exited containers + logs

# - - `crictl ps -a` shows kube-apiserver container exited ~7 minutes ago.
- Attempting `crictl logs <container-id>` returns an error (container ID not found / runtime failed message).
- Host checks default log directory:
- `/var/log/containers`
- Finds only Calico logs; kube-apiserver logs appear deleted when container exits.

### ### Static Pod manifests: `/etc/kubernetes/manifests`

# - - Host goes to the static pod manifest directory:
- `/etc/kubernetes/manifests`
- Opens kube-apiserver manifest:
- `sudo vi kube-apiserver.yaml`
- Finds the root cause:
- command has an extra character: `kube-apiserverr` (incorrect)
- Fix:
- remove extra `r`, save file
- After a short wait:
- `crictl ps` shows kube-apiserver running
- `kubectl get nodes` works again

## kubectl still failing even if API server is running: kubeconfig issues

### ### kubeconfig (KUBECONFIG)

# - - Host demonstrates a second failure mode:
- API server is up, but kubectl can still fail if kubeconfig is wrong.
- Sets KUBECONFIG to a bogus path:
- `export KUBECONFIG=/t/config` (non-existent)
- `kubectl get nodes` now errors (connection refused/localhost style message).

### Confirm API server is actually running

### ### crictl

# - - `crictl ps | grep api` shows kube-apiserver is running.
- So the issue isn’t apiserver; it’s client config/networking.

### Fix options shown

# - 1. Use correct kubeconfig explicitly:
- `kubectl get nodes --kubeconfig /etc/kubernetes/admin.conf`
- Initially hits permission denied
- Fix permissions:
- `sudo chmod 775 /etc/kubernetes/admin.conf`
- Command then works
2. Restore correct default kubeconfig setup:
- copy/use admin.conf as `$HOME/.kube/config`
- set environment variable correctly (conceptually):
- `export KUBECONFIG=$HOME/.kube/config`
- `kubectl get nodes` works again

### Practice note

# - - Host says Day 38 GitHub folder includes scripts that break the cluster; run them to practice troubleshooting without peeking at the scripts.

## Scenario 2: Pods stuck Pending → Scheduler failure (image pull error)

### ### kubectl run / describe

# - - Host creates a test pod:
- `kubectl run nginx --image=nginx`
- Pod is created but stuck **Pending**.
- `kubectl describe pod nginx`:
- no events
- **Node: none** (not scheduled)

### Identify responsible control plane component

# - - Host prompts: scheduling is handled by **kube-scheduler**.

### ### kube-system control plane pods

# - - Checks control plane pods:
- `kubectl get pods -n kube-system`
- Finds:
- `kube-scheduler-master` has **ImagePull** / no healthy containers.

### ### kubectl logs / describe (scheduler)

# - - `kubectl logs -n kube-system kube-scheduler-master` indicates it’s failing to pull image.
- `kubectl describe pod -n kube-system kube-scheduler-master` shows events:
- ImagePullBackOff
- failed to pull image

### Fix: correct the scheduler image tag

### ### Static Pod manifest edit

# - - Opens scheduler manifest:
- `sudo vi /etc/kubernetes/manifests/kube-scheduler.yaml`
- Sees image tag is wrong (ends with something like `v1.30...200x`).
- Checks correct version by comparing with kube-apiserver image tag (uses **1.30.2**).
- Updates scheduler image to match (e.g., `1.30.2`), saves.
- Waits for pod to come up.
- After scheduler is running:
- nginx pod becomes Running (now scheduled).

## Scenario 3: Deployment not maintaining replicas → Controller Manager crashloop (bad command)

# - - Host discusses expected behavior:
- a Deployment/ReplicaSet should recreate pods if one is deleted.
- After simulating failure and deleting a pod:
- only one pod remains even though replicas should be two.

### Identify responsible component

# - - Ensuring desired state == actual state is handled by **kube-controller-manager**.

### ### kube-controller-manager failure detection

# - - `kubectl get pods -n kube-system` shows:
- `kube-controller-manager-master` in **CrashLoopBackOff**

### ### kubectl describe pod (controller-manager)

# - - `kubectl logs` doesn’t show much, so host uses `kubectl describe`.
- Error:
- *“executable file not found in the path”*
- Conclusion: the command is wrong.

### Fix: correct the controller-manager command typo

### ### Static Pod manifest edit

# - - Opens:
- `sudo vi /etc/kubernetes/manifests/kube-controller-manager.yaml`
- Finds a misspelling: extra “e” in `kube-controller-managere` (incorrect).
- Corrects it, saves.
- Watches pod transition: Pending → ContainerCreating → Running.
- After controller-manager recovers:
- the missing replica pod is created
- Deployment returns to desired state.

## Scenario 4: Scaling fails → Controller Manager can’t load CA cert (wrong hostPath)

# - - Host scales deployment:
- attempts to scale replicas from 2 → 4
- Deployment shows only 2/4 ready; pods don’t increase.

### ### kubectl logs (controller-manager)

# - - Checks controller-manager logs and finds:
- *“unable to load client CA provider… open /etc/kubernetes/pki/ca.crt: no such file or directory”*
- Host recalls TLS/cert files are provided via mounted hostPath volumes.

### Fix: correct certificate directory mount path

### ### Static Pod manifest volume/hostPath check

# - - In controller-manager manifest:
- VolumeMount uses mountPath: `/etc/kubernetes/pki` (expected)
- HostPath mistakenly points to `/etc/kubernetes/pk` (missing the “i”)
- Corrects hostPath to `/etc/kubernetes/pki`, saves.
- Tails logs (`kubectl logs -f ...`) and sees normal startup messages (e.g., starting controllers).
- After fix:
- pods scale up to 4 successfully.

## Cluster status & helpful built-in commands

### ### kubectl cluster-info

# - - Host shows:
- `kubectl cluster-info` (control plane endpoint, CoreDNS info)

### ### kubectl cluster-info dump

# - - Runs:
- `kubectl cluster-info dump`
- Notes it outputs a lot of details (describes components, inspects logs, etc.).

## Recommended reading + practice approach

# - - Host references Kubernetes docs links (to be shared in GitHub):
- “debug cluster”
- “debugging cluster with crictl”
- Practice method:
- in Day 38 folder, run provided scripts **one by one**
- don’t open scripts first (so you don’t spoil the failure)
- fix issues by checking:
- manifests
- logs
- directories
- commands
- Next video preview:
- networking and worker node failure scenarios (also important).

# 

# 

#