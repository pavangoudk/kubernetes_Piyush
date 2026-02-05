# CKA 2024 Video 29: Kubernetes Storage (Volumes, PV/PVC, StorageClass)

## What this video covers (and suggested prerequisite)

- Speaker introduces **Video 29** in the **CKA 2024** series on **Storage**.
- Topics listed (in order):
- **PersistentVolume (PV)** and **PersistentVolumeClaim (PVC)**
- Brief intro to **StorageClass**
- “few other storage related concepts” in Kubernetes
- Suggests a prerequisite:
- if you watched the previous Docker storage video, it helps; otherwise, go back to it first.
- Mentions a **GitHub repository** will include:
- notes, diagrams, sample code
- hands-on tasks for viewers to practice

## Demo 1: Basic Kubernetes volume with `emptyDir` (non-persistent)

### ### Pod manifest (YAML)

- Speaker creates a new YAML file (e.g., `redis.yml`) to run a **Redis pod**.
- Starts with a basic Pod spec:
- `apiVersion: v1`
- `kind: Pod`
- `metadata` (name set to something like `redis-pod`; labels optional)
- `spec.containers`:
- container name: Redis
- image: Redis

### ### volumeMounts (mounting storage inside a container)

- Adds `volumeMounts` under the container:
- `name`: `redis-storage`
- `mountPath`: `/data/redis`
- Explains intent:
- container stores data; data “has to reside somewhere,” so attach a volume.

### ### volumes + emptyDir (temporary storage)

- Adds `volumes` at the Pod spec level:
- volume name: `redis-storage` (must match `volumeMounts.name`)
- type: `emptyDir: {}`
- Definition-style explanation (closely paraphrased):
- *`emptyDir` is “temporary storage” that lasts “till the lifecycle of the Pod,” not beyond it.*
- Clarifies persistence:
- *If the pod restarts (delete/recreate), data is gone.*
- It is not “persistent beyond the pod.”

### Apply + fix a YAML mistake

### ### kubectl apply

- Applies YAML and hits an error due to a typo:
- wrote `mouthPath` instead of `mountPath`.
- Fixes it, reapplies, and the pod is created and running.

## Test `emptyDir` behavior: container process restart vs pod recreation

### ### kubectl exec

- Execs into the pod (`kubectl exec -it ... -- sh`).
- Navigates to `/data/redis`.
- Creates a file:
- writes “hello kubernetes” to `test.txt`.
- Installs process tooling (because `ps` wasn’t available):
- uses package manager commands (e.g., `apk update` and installs `procps`).
- Finds the Redis process and kills it (`kill -9 <pid>`).
- Observes:
- Redis process restarts automatically.
- The file still exists in `/data/redis`.
- Key takeaway (as explained):
- data remains across *container process restart* because the volume is attached to the **Pod**, not to the process.

### Delete and recreate pod

- Exits, deletes the pod, reapplies the YAML.
- Re-enters `/data/redis` and confirms:
- file is gone (data not persistent across pod recreation).

## Why persistent storage is needed

- Speaker explains why `emptyDir` isn’t sufficient:
- pods can “crash for so many reasons”
- autoscaling can create new pods
- deployments/rollouts create new pods
- Conclusion:
- stateful apps or apps needing to retain data require **persistent storage beyond pod lifecycle**.

## PV/PVC concept explained with roles and a capacity “pool” diagram

- Introduces two actors:
- **Storage admin**: provisions cluster storage
- **User/operator (DevOps/Kubernetes admin)**: runs workloads and requests storage

### ### PersistentVolume (PV)

- Definition-style phrasing:
- *A PV is “a pool of storage” provisioned by the storage admin that workloads can use.*
- Example:
- storage admin provisions **100Gi** PV capacity.

### ### PersistentVolumeClaim (PVC)

- User creates a PVC specifying:
- requested storage (e.g., **10Gi**)
- access/read-write mode (he calls out “read write mode”)
- Kubernetes matches PVC to PV based on:
- capacity availability
- matching access mode
- When matched:
- a **binding** is created between PV and PVC
- PVC is attached to the pod

### Capacity consumption examples

- After first 10Gi claim from 100Gi PV:
- remaining pool becomes 90Gi
- After second 10Gi claim:
- remaining becomes 80Gi
- Example where PVC requests too much (e.g., 90Gi when only 80Gi left):
- PVC stays **Pending**
- pod that depends on it also stays **Pending**
- Example where capacity fits but access mode differs:
- PVC also stays **Pending** until a matching PV exists.

### ### Access modes (as listed)

- He reads/points to Kubernetes access modes:
- **ReadWriteOnce**
- **ReadOnlyMany**
- **ReadWriteMany**
- **ReadWriteOnce** (he notes four options and highlights these common ones)

## Demo 2: Create a PV and PVC, then a Pod that consumes the PVC

### ### PersistentVolume (PV) YAML

- Opens `pv.yaml` and explains fields:
- `kind: PersistentVolume`
- labels
- `capacity.storage`: total capacity of the PV (example: **1Gi**)
- *this is the total pool capacity others can claim from*
- `accessModes`: must align with claims
- `hostPath`: used as the backing storage source in this demo
- Explains hostPath choice:
- not provisioning physical/cloud storage here
- uses a directory on the VM/EC2 host as storage source (e.g., `/home/.../day29`)
- warns:
- *hostPath is not recommended for multi-node clusters*
- used here as a “hack” to demonstrate by scheduling on the same node
- Applies PV with `kubectl apply -f pv.yaml`.

### ### PersistentVolumeClaim (PVC) YAML

- Opens PVC YAML and explains:
- `kind: PersistentVolumeClaim`
- `spec.accessModes`: must match PV’s access mode or it won’t bind
- `resources.requests.storage`: requested size (example: **500Mi** out of 1Gi)
- Applies PVC.
- Checks:
- `kubectl get pv` shows status **Bound**
- `kubectl get pvc` shows status **Bound**
- notes PVC may show PV capacity because it’s bound to that PV.

### ### Reclaim policy (PV reclaim behavior)

- Definition-style explanation:
- *Reclaim policy decides “once the PVC is deleted, what will happen to the PV.”*
- Policies explained:
- *Retain*: PV remains, enters Released state, not available for other claims
- *Delete*: PV is deleted when PVC is deleted
- *Recycle*: PV becomes available again for other PVCs to claim
- Notes StorageClass exists but will be covered after.

## Pod consumes PVC (and scheduling nuance)

### ### Pod YAML: volumes + PVC reference

- Opens pod YAML (e.g., `vi.yml`) and points out:
- `volumeMounts` under container:
- mounts storage into container path (example: `/usr/share/nginx/html`)
- `volumes` under pod spec:
- uses `persistentVolumeClaim.claimName` referencing the existing bound PVC

### ### Node scheduling constraint (control plane)

- He schedules the pod onto the **control-plane/master node** using `nodeName`.
- He calls out a concept question:
- earlier, they said custom workloads shouldn’t be scheduled on control plane nodes
- asks viewers to think: *why was it restricted earlier, and how is it working now?*
- invites answers in comments.

### ### Verify the mount works

- Pod runs; he execs into it and checks the mounted directory.
- Observes:
- files from the hostPath directory appear inside the container at the mount path.
- Uses `curl localhost` against nginx to confirm it serves the content from the mounted directory.

## StorageClass: why it exists and how it fits

### ### StorageClass (concept + provisioners)

- Motivation:
- storage shouldn’t live on the node; it should come from:
- cloud storage
- private datacenter storage
- third-party storage vendors
- StorageClass uses a **provisioner**:
- examples mentioned include Azure/AWS/GCP-style backends and others (speaker lists multiple provisioners such as Azure file, NFS, vSphere, etc.).
- Key YAML field:
- `storageClassName` is specified so PV/PVC associate with that StorageClass.

### Static vs dynamic provisioning

- *Static provisioning*: what they did earlier—manually create PV, then PVC binds to it (and you can reference storage class along with other details).
- *Dynamic provisioning*:
- you create **only the PVC**
- Kubernetes automatically creates the PV for you via the StorageClass.
- If no storageClassName is provided:
- it uses the cluster’s **default StorageClass** (if one exists).
- You can have multiple StorageClasses but typically one default.

## Wrap-up + task reminder

- Speaker summarizes the main storage topics:
- volumes (`emptyDir`)
- PV/PVC binding
- StorageClass and provisioning concepts
- Asks viewers to complete the hands-on task in the “day 29” folder and references the GitHub repo materials.