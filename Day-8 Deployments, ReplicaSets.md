# CKA 2024 — Video 8: ReplicaControllers, ReplicaSets, Deployments (Hands-on)

## 1) Why this topic matters

# 

- The speaker frames **Deployments and ReplicaSets** as core Kubernetes concepts because “everything” ultimately runs as a **container in a Pod**, and Pods are commonly managed by **ReplicaSets / Deployments** (and later StatefulSets).

## 2) The problem with a single Pod

# 
- If a user is effectively relying on **one nginx Pod** and it crashes, the user stops getting responses.
- Kubernetes needs an **auto-healing** mechanism plus **high availability** via multiple replicas.

## 3) Replication Controller (RC): what it does (per speaker)

# 
- A **ReplicationController** ensures a specified **number of identical Pod replicas** is always running.
- If a Pod is deleted/crashes, the controller recreates it to maintain the desired replica count.
- Speaker also describes it as enabling load balancing across replicas (see “Quick corrections” below).

### RC manifest structure shown

# 
- apiVersion: v1
- kind: ReplicationController
- metadata: name, labels
- spec:
- replicas: (e.g., 3)
- template: embedded Pod template (metadata + spec.containers…)

### Commands shown

# 
- Find apiVersion/kind details: kubectl explain rc
- Create: kubectl apply -f rc.yaml
- Verify: kubectl get pods, kubectl get rc
- Inspect a pod to see controller ownership: kubectl describe pod <pod>

## 4) ReplicaSet (RS): preferred over RC + key difference

# 
- Speaker calls **ReplicationController “legacy”** and **ReplicaSet “newer/preferred.”**
- Key functional difference highlighted:
- **ReplicaSet supports selectors** (matchLabels) to manage Pods by label, including Pods not originally created by that RS.

### Common gotcha demonstrated: apiVersion for ReplicaSet

# 
- Their first attempt fails because they used v1.
- Fix: ReplicaSet uses apiVersion **apps/v1** (group apps, version v1).
- Confirm via: kubectl explain rs (top shows apiVersion).

### Scaling ReplicaSets: three methods shown

# 
1. Edit YAML and re-apply: change spec.replicas then kubectl apply -f …
2. Edit the live object: kubectl edit rs <name> then change spec.replicas
3. Imperative scale: kubectl scale --replicas=<n> rs/<name>

### Exam tip emphasized

# 
- Prefer faster methods (imperative/short commands) under time pressure; use help/cheat sheet when needed (kubectl scale --help).

## 5) Deployment: why it exists (beyond a ReplicaSet)

# 
- A **Deployment manages ReplicaSets**, and ReplicaSets manage Pods.
- Value-add highlighted:
- **Rolling updates** to avoid downtime when changing versions (e.g., nginx:1.1 → nginx:1.2).
- **Rollback/undo** to revert changes to a previous revision.
- Speaker’s mental model:
- User creates Deployment → Deployment creates ReplicaSet → ReplicaSet creates Pods.

### Commands demonstrated (deployment lifecycle)

# 
- Create deployment from YAML: kubectl apply -f <file>
- Check: kubectl get deploy
- See everything: kubectl get all (shows deploy, rs, pods, and default service)

### Update image (rolling update trigger)

# 
- They update the live Deployment to nginx:1.9.1 using kubectl set image deploy/<name> <container>=nginx:1.9.1 (conceptually; their narration shows the workflow).
- Confirm via: kubectl describe deploy <name>

### Rollout history + undo

# 
- View revisions: kubectl rollout history deploy/<name>
- Roll back last change: kubectl rollout undo deploy/<name>
- After undo, the image returns to the prior value (nginx “latest” in their demo).

## 6) Generating YAML quickly (again): dry-run

# 
- They reiterate the productivity pattern:
- Generate manifests with kubectl … --dry-run=client -o yaml > file.yaml
- Then edit and apply.
- Example shown for deployments: kubectl create deployment <name> --image=nginx --dry-run=client -o yaml > deploy.yaml

## 7) Quick corrections / clarifications (important in real Kubernetes)

# 
- **ReplicaSet/ReplicationController do not “load balance” traffic by themselves.** Traffic distribution is typically handled by a **Service** (ClusterIP/NodePort/LoadBalancer) and kube-proxy/networking, with endpoints updated as Pods come/go.
- **Directly editing a Pod** isn’t a reliable “update” strategy for production because Pods are ephemeral; controllers (RS/Deployments) will recreate Pods from the template. Prefer changing the controller template instead.

-
