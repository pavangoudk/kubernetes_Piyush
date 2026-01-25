# Kubernetes (CKA 2024, Video 12): DaemonSets + Basics of CronJobs and Jobs

## What this video covers (and exam relevance)

# 

- Video #12 in the **CKA 2024** series.
- Topics:
- **CronJobs**, **Jobs**, and the “important concept” of **DaemonSet**.
- Exam note from speaker:
- CronJobs/Jobs: *“not really important from CKA perspective… more on the CKAD curriculum”* (still covered for fundamentals).
- DaemonSet: *“important from this exam perspective and from the kubernetes point of view as well.”*
- Engagement target: **200 likes** and **120 comments** in 24 hours.

## DaemonSet: how it differs from Deployments/ReplicaSets

# 
- Recap framing: previously covered replicas and how Pods get deployed across nodes.
- Deployments example:
- You set `replicas: 3` → creates three Pods (e.g., nginx1, nginx2, nginx3) across nodes, *“irrespective of number of nodes.”*
- DaemonSet behavior:
- *“It will create the replicas in all the nodes—one replica each node.”*
- Example:
- 3 nodes → 3 Pods (one per node).
- Node changes:
- Add a new node → DaemonSet detects it and creates a new replica on that node.
- Delete a node → the DaemonSet replica on that node is deleted.
- Speaker contrasts this with deployments/replicasets/standalone pods (this auto node-following behavior is specific to DaemonSets).

### Why we use DaemonSets (use cases listed)

# 
- Monitoring agent (example: an “agent” running on every node)
- Logging agent (stream/report logs to third-party systems like **Splunk** or **ELK**)
- Control plane / cluster components and networking components:
- **kube-proxy** (speaker: deployed as a DaemonSet; responsible for pod networking discussed earlier)
- CNI (Container Network Interface) plugins, deployed as DaemonSets:
- **Weave**
- **Flannel**
- **Calico**
- Reliability behavior:
- If a DaemonSet pod is deleted, it is recreated (controller ensures desired state).

### kubectl (commands + technique references)

# 
- Speaker uses `kubectl` to create, inspect, and switch clusters/contexts, and to explore DaemonSets.

## Demo: Creating a DaemonSet YAML (by modifying a Deployment)

# 
- Speaker reuses a Deployment YAML from earlier videos (doesn’t write from scratch).
- Key changes described:
1. Change `kind` to **DaemonSet**
2. API version is the same as deployment (speaker suggests verifying via `kubectl explain daemonset`)
3. Remove `replicas` field:
- *DaemonSet doesn’t need replicas because it runs one per node automatically.*
- Applies the manifest and checks Pods.

### Why only 2 pods on a 3-node cluster?

# 
- Observation: only **two** DaemonSet pods running.
- Explanation:
- The **control-plane node** has a **taint**.
- Custom workloads won’t schedule there unless they can tolerate it.
- Speaker flags **Taints and Tolerations** as important later; will cover in depth.

### Recreate behavior (delete pod test)

# 
- Deletes one DaemonSet pod.
- Verifies it returns to two pods again (recreated).

## Demo: Confirm kube-proxy DaemonSet in kube-system

# 
- Checks DaemonSets across namespaces:
- Uses `-A` for all namespaces, or specifies namespace **kube-system**.
- Notes kube-proxy runs in **kube-system**.
- Output interpretation described:
- Sees DaemonSets like **kindnet** (from kind) and **kube-proxy**
- kube-proxy shows desired/current/ready matching node count (e.g., 3/3/3)

## Demo: Single-node cluster behavior

# 
- Switches kubectl context to a **single-node** kind cluster.
- Confirms:
- Only one node (control-plane).
- DaemonSets (kube-proxy, kindnet) show **1 desired / 1 ready**.
- Correspondingly, only one kube-proxy pod and one kindnet pod.

## Inspecting DaemonSet-managed pods

# 
- Switches back to the main cluster.
- Describes inspecting a DaemonSet-created pod:
- `kubectl describe pod ...`
- Notes a key difference:
- It shows **“controlled by DaemonSet”**
- Whereas in deployment scenarios, it shows **“controlled by deployment”**
- Mentions it runs on port 80 and uses nginx (from earlier manifest).

## CronJob overview (basics)

# 
- Speaker references documentation and defines usage:
- *CronJob executes a job “on a certain time… day… or after a certain interval,” using a schedule format.*
- Cron syntax overview (five fields, five stars by default):
1. Minute: 0–59
2. Hour: 0–23
3. Day of month: 1–31
4. Month: 1–12
5. Day of week: 0–6 (Sunday=0, Saturday=6)

### Cron schedule examples given (in order)

# 
1. Every Saturday:
- Set day-of-week field to **6**, others `*`
2. Every Saturday at 11:00 PM:
- Day-of-week = 6
- Hour = 23
- Others `*`
3. Every Saturday at 11:45 PM:
- Day-of-week = 6
- Hour = 23
- Minute = 45
- Others `*`
4. Every N minutes:
- Use `*/n` in the minute field
- Example: every 10 minutes → `*/10`
- Example: every 5 minutes → `*/5`

### CronJob manifest behavior (example described, not run)

# 
- Example schedule: every minute.
- It spins up a pod (name “hello”) using **busybox:1.28** and runs:
- prints time (via Unix `date`)
- echoes “hello from kubernetes cluster”
- Use cases mentioned:
- generate reports (daily/weekly/monthly)
- cleanup tasks (e.g., clearing older logs)

## Job overview (basics)

# 
- Similar to CronJob, but:
- *“it does not execute on a certain time… it executes once and it is then completed.”*
- Typical usage described:
- installation scripts
- provisioning workflows / automation pipelines
- does some operation and completes

## Wrap-up + assignment

# 
- Speaker recap: covered **CronJob**, **Job**, and **DaemonSet**.
- Assignment mentioned:
- GitHub repository, **day 12** folder.
- Support channels:
- comments
- Discord community
- Reminder to meet like/comment target and watch next video.

# 

#