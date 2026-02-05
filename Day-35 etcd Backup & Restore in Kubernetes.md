# CKA 2024 Video 35: etcd Backup & Restore in Kubernetes (Snapshot Save/Restore + Static Pod Updates)

## Intro: why this matters (CKA-relevant)

# 

- Host introduces **Video #35** in the CKA 2024 series, near the end of the playlist.
- Topic: **etcd backup and restore**, end-to-end demo.
- Emphasizes this is:
- an important Kubernetes admin task
- *“for sure one of the task[s] that you will be provided in the CKA exam”*
- Encourages hands-on practice via the GitHub task to save time during the exam.

## What to back up: Kubernetes objects vs etcd (why etcd is preferred)

### Exporting Kubernetes objects to YAML (shown, but called inefficient)

# 

- One approach: back up “objects/resources” by exporting API outputs:
- `kubectl get all`
- and for everything across namespaces (including control plane-related objects):
- `kubectl get all -A`
- redirect output to a file (example):
- `> backup.yaml`
- Host explains why this is not ideal:
- these are “just configuration details” / manifest-derived objects
- they may already exist in Git (deployment.yaml, service.yaml, etc.)
- it may not capture other important data (example mentioned: **persistent volumes**)

### Why etcd backup is the key one

# 
- Host recalls etcd from an earlier control-plane video and defines it:
- *“etcd is a key value data store which stores all the cluster state configuration data and all the manifest objects in the Kubernetes cluster in a key value database.”*
- Reasoning:
- because etcd is the “source of everything” running in the cluster, backing up etcd is sufficient for restoring cluster state.
- When backups are taken:
- before a **cluster upgrade**
- before **major releases**
- whenever a rollback might be needed after a mishap

### Managed Kubernetes note + third-party tools

# 
- etcd access typically only exists on **self-managed clusters**.
- In managed services (GKE / AKS / EKS mentioned), you “won’t get access” to etcd client.
- Alternative tools mentioned:
- **Velero** (third-party tool)
- or exporting via the Kubernetes API (kubectl-based approach), depending on what’s accessible.

## Where etcd runs + where its data/certs live (static pod manifest inspection)

### ### /etc/kubernetes/manifests (static pod manifests)

# 

- Host points to control-plane static pod manifests location:
- `/etc/kubernetes/manifests`
- Lists what’s there:
- `etcd.yaml`, `kube-apiserver.yaml`, controller-manager, scheduler.

### ### etcd.yaml (key flags to capture for backup)

# 
- From `etcd.yaml`, host highlights the etcd container command arguments and the fields relevant to backup/restore:
- **data directory**
- *“this is the directory in which etcd by default stores all of its configuration data”*
- **listen client URLs**
- etcd client endpoint for the local setup:
- `https://127.0.0.1:2379` (port **2379** emphasized)
- certificate files:
- **key file** (private key)
- **cert file** (public cert)
- **CA cert file** (“trusted CA file”)

### Volume mounts explained (host diagram)

# 
- etcd static pod mounts two host paths into the container:
1. data directory (e.g., `/var/lib/etcd`)
2. PKI/certs directory (e.g., `/etc/kubernetes/pki/etcd`)
- Purpose (close paraphrase):
- the container can reference these as local files because they’re mounted into the pod.

## Taking an etcd snapshot backup

### ### etcdctl (installation)

# 

- Host introduces the tool:
- *“etcdctl is a utility that interacts with your etcd server and perform[s] administrative tasks such as taking the backup [and] restore.”*
- Installs it on the node:
- `sudo apt install etcd-client`

### ### etcdctl API version requirement

# 
- Host stresses an important setting:
- `ETCDCTL_API=3`
- Explanation (close paraphrase):
- *without this, etcdctl defaults to version 2 which has deprecated features.*
- Two ways shown:
- prefix per command
- or export once:
- `export ETCDCTL_API=3`

### Snapshot commands + required flags

# 
- Host shows etcdctl snapshot has actions like save/restore/status and requires key flags:
- `--endpoints`
- `--cacert`
- `--cert`
- `--key`

#### Mapping YAML fields to CLI flags (host’s “how to remember” tip)

# 
- Notes naming mismatch:
- CLI uses `--cacert`
- manifest uses something like *“trusted-ca-file”*
- He uses docs as a mapping guide and says practice makes it automatic.

### Snapshot save (actual backup file)

# 
- Builds the snapshot save command using:
- endpoint from **listen-client-urls** (localhost:2379)
- cert/key/ca file paths from etcd.yaml
- output file path:
- `/opt/etcd-backup.db` (he notes `.db` “should be the extension”)
- Troubleshooting moment:
- uses `--endpoints` (plural), not `--endpoint`
- uses `sudo` since permissions weren’t adjusted
- Confirms success:
- “snapshot saved” at `/opt/etcd-backup.db`
- Checks size:
- uses `du -sh` and notes ~**56 MB** (later sees **58 MB** via status output)

### Snapshot status (verify snapshot metadata)

# 
- Runs a status command (table output) against the snapshot file.
- Output fields mentioned:
- size (more precise)
- total keys
- hash (unique identifier)

## Simulated failure scenario: delete workload objects

# 
- Host shows why restore matters by deleting resources:
- deletes a Deployment (example: nginx)
- deletes a Service (example: nginx service)
- Frames it as:
- accidental deletion / mishap during an upgrade or maintenance window
- Goal: restore the cluster to the previous state using snapshot restore.

## Restoring from snapshot (data-dir restore + static pod reconfiguration)

### Restore prerequisites (from docs)

# 

- Host references the official restore flow:
- stop API server instance
- restore state
- restart API server
- Mentions recommendation:
- restart other control-plane components (scheduler, controller-manager, kubelet) to avoid stale data.

### Tooling note: etcdctl vs etcdutl

# 
- Host highlights a version-based tool shift:
- docs show **etcdutl** as newer
- they’re on etcd **3.5.x**, still using **etcdctl**
- *etcdctl restore is deprecated and removed in 3.6; in 3.6 you should use etcdutl*
- Says for the exam lab: the needed utility will be present.

### Snapshot restore command (restore into a NEW directory)

# 
- Runs snapshot restore with:
- snapshot file: `/opt/etcd-backup.db`
- `--data-dir` set to a new path:
- `/var/lib/etcd-restore-from-backup`
- Corrects a flag typo:
- uses `--data-dir` (hyphen), not `data__directory` style.

### Compare directories (old vs restored)

# 
- Original etcd data dir:
- `/var/lib/etcd` (contains member/ data structure)
- New restored dir:
- `/var/lib/etcd-restore-from-backup` (similar structure)

## Point etcd static pod to the restored data directory

### ### /etc/kubernetes/manifests/etcd.yaml (edit required in TWO places)

# 

1. Update the etcd flag:
- `--data-dir=` from `/var/lib/etcd` → `/var/lib/etcd-restore-from-backup`
2. Update the **volume mount path/hostPath** reference
- because etcd.yaml mounts `/var/lib/etcd` from the host into the container, that host path must also be changed to the restored directory.

- Host initially changes only the flag, then realizes the mount path must be updated too.

## Restarting control plane components (static pods) + kubelet restart

### ### Static pod restart approach (move manifests out and back)

# 

- To force restart of static pods:
- move `*.yaml` from `/etc/kubernetes/manifests` to a temp dir (stops them)
- move them back (kubelet recreates/restarts the containers)

### Observed issue: changes not reflected until kubelet restart

# 
- After edits, etcd appeared to still point to the old directory.
- Host restarts kubelet:
- `sudo systemctl daemon-reload`
- `sudo systemctl restart kubelet`
- After that, describing the etcd pod shows correct paths:
- data dir and mount align to `/var/lib/etcd-restore-from-backup`

### Verification: objects return after restore

# 
- Runs checks:
- `kubectl get pods` → deleted pods/resources are back
- `kubectl get services` → service restored
- Conclusion:
- restore succeeded and previous cluster state returned.

## Further reading: HA etcd / control plane topologies

# 
Host suggests reading docs about multi-node, HA setups (beyond the single-control-plane demo).

### ### HA topology: stacked etcd

# 
- Multiple control plane nodes, each running control plane components + etcd member.
- A load balancer distributes API server traffic.
- etcd runs “stacked” with control plane nodes.

### ### HA topology: external etcd

# 
- Separate etcd cluster provisioned outside the control plane nodes.
- Control plane nodes connect to the external etcd cluster.
- Trade-off noted:
- higher availability but requires separate infrastructure → higher cost.

## Closing: practice + engagement target

# 
- Host reiterates:
- complete the GitHub task (exam-style)
- practice end-to-end backup/restore
- Updated engagement target for this video:
- **200 likes** and **150 comments** in 24 hours
- Mentions there are 4–5 videos remaining and offers help via comments/Discord.

# - 
