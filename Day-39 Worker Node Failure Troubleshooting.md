# CKA 2024 Video 39: Worker Node Failure Troubleshooting (NotReady Nodes, CNI Checks, kubelet Service + Logs)

## Intro: worker node failures (exam perspective)

# 

- Host (PE) introduces **Video #39** in the CKA 2024 series (near the end).
- Focus: **worker node failure scenarios** and how to troubleshoot/fix them “from exam perspective.”
- Engagement target: **300 likes** and **100 comments** in 24 hours.

## Initial symptom: worker nodes NotReady

### ### kubectl

# 

- Host logs into the cluster and runs:
- `kubectl get nodes`
- Observation:
- **master/control plane node = Ready**
- **worker nodes = NotReady**
- Host notes:
- If *all* nodes were NotReady/unhealthy, that could indicate a broader **networking issue** (e.g., network add-on not installed or failing).

## Rule-out check: is the network add-on installed and healthy?

# 
Host checks whether the **CNI (Container Network Interface)** add-on (Calico/Flannel/Weave, etc.) is present and running.

### ### kubectl get namespaces

# 
- Runs:
- `kubectl get namespaces`
- Sees namespaces indicating **Calico** components:
- `calico-system`
- `calico-apiserver`
- plus something like `tigera-operator` (new/extra namespace noted)

### ### kubectl get pods -A / per-namespace

# 
- Mentions you can:
- list pods across all namespaces (he references `kubectl get pods -A`)
- then narrow to Calico namespaces
- Checks Calico-related namespaces:
- `kubectl get pods -n tigera-operator`
- similarly for `calico-system` and `calico-apiserver`
- Observation:
- Calico pods appear **running/healthy**, so network add-on is likely fine.

### ### /etc/cni/net.d (confirm CNI config files)

# 
- Host checks the default location for CNI configs:
- `/etc/cni/net.d`
- Notes you may not have permissions in the exam sandbox:
- uses `sudo su` to become root
- Lists files and points out Calico indicators:
- a file like `10-calico.conflist`
- a Calico kubeconfig file
- Host notes:
- If another plugin were used, filenames would reflect it (e.g., Weave/Flannel).

**Conclusion of this section:** CNI appears installed and healthy → focus shifts to worker node-level issues.

## Approach: SSH into each worker and troubleshoot locally

# 
- Host says the exam prompt may be: cluster is broken → log in using context and fix.
- He SSHs into worker nodes (notes SSH command will be provided in the exam).
- Mentions:
- kubectl commands on the worker may fail because kubeconfig isn’t set there; he doesn’t configure it since it’s not needed for the fix.

## Worker Node 1: kubelet service is stopped

### ### kubelet (service)

# 

- Host checks kubelet status using service management:
- `service kubelet status`
- Output indicates:
- *“active = inactive”* / *“dead”* / exited
- Host infers:
- kubelet may have been stopped “voluntarily” or accidentally.

### Fix: start kubelet

# 
- Runs:
- `sudo service kubelet start`
- Verifies:
- `service kubelet status` → active/running
- Returns to control plane and re-checks:
- `kubectl get nodes`
- Worker 1 now reports **Ready**.

## Worker Node 2: kubelet stuck “activating” → check logs and fix config

### ### kubelet (service)

# 

- On worker 2:
- `sudo service kubelet status`
- Status:
- “activating” / not fully up; shows failure state without helpful inline logs.

### ### journalctl (service logs)

# 
- Host uses journalctl to inspect kubelet logs:
- `journalctl -u kubelet`
- Navigation tip:
- press **Shift+G** to jump to the latest logs.
- Log level cues described:
- *“I denotes info”*
- *“E denotes error”* (focus on E lines)

### Error found (client CA path typo)

# 
- Key error message (paraphrased closely from transcript):
- failed to construct kubelet dependencies
- unable to load client CA file at `/etc/kubernetes/pki/...` because the filename/path is wrong

### Locate kubelet config file

# 
- Host goes to kubelet config directory:
- `/var/lib/kubelet/`
- Identifies kubelet config file:
- `config.yaml`
- Opens it with sudo and finds the incorrect client CA file reference.

### Verify correct CA file name

# 
- Lists PKI directory:
- `ls /etc/kubernetes/pki`
- Determines correct file is:
- `ca.crt`

### Fix: update kubelet config and restart

# 
- Edits `/var/lib/kubelet/config.yaml`:
- replaces the wrong CA filename with `ca.crt`
- Restarts kubelet:
- `sudo service kubelet restart`
- Confirms:
- `sudo service kubelet status` → active/running
- Returns to control plane:
- `kubectl get nodes` → worker 2 becomes **Ready**.

## How to find kubelet’s config paths (don’t memorize)

# 
- Host emphasizes you don’t need to memorize paths:
- kubelet service details show the flags and file paths it uses.
- Example given from service details:
- `--bootstrap-kubeconfig=...`
- `--kubeconfig=...`
- `--config=/var/lib/kubelet/config.yaml`

## Closing + next video preview

# 
- Host summarizes:
- exam-style worker node issues often involve kubelet:
- service stopped
- config typo (CA file path)
- checking logs via journalctl
- Next video:
- an end-to-end project: host your own Docker Hub–style registry on Kubernetes, using concepts from the series.
- Thanks viewers and reiterates the like/comment target.

# - -

# 

# 

#