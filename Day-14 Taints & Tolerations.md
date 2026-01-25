# Kubernetes (CKA 2024, Video 14): Taints & Tolerations + Node Selectors (Demo)

## What this video covers + why it matters

# 

- Video #14 in the **CKA 2024** series.
- Main focus: **taints and tolerations** (speaker calls it *“very important… yet confusing”*).
- Also covered: **node selectors**.
- Goal: explain from basics + do a demo so it becomes “a piece of cake.”
- Target: **200 comments** and **200 likes** in 24 hours.

## Taints & Tolerations: core idea (board explanation)

# 
- Example cluster: **3 nodes**
- Node 1 is “specialized” for AI workloads (has GPUs).
- Speaker’s taint example:
- Add a taint on node 1: `GPU=true`
- Scheduling behavior described:
- When a pod is scheduled, the scheduler tries nodes.
- Node 1 has a taint that effectively says: only pods with matching toleration (`GPU=true`) can be scheduled there.
- A pod without matching toleration:
- is rejected by the tainted node
- gets scheduled on another available node instead.
- To schedule onto the tainted node:
- Add a toleration to the pod:
- *“Toleration is something that will enable the pod to be scheduled on the node in which there is a taint.”*
- With toleration `GPU=true`, the scheduler can place it on node 1.
- Key “direction” clarification:
- *“we are not actually scheduling the pod on a particular node… we are actually instructing the node to accept only certain pods.”*
- Memory aid (explicit):
- **Taint → on the node**
- **Toleration → on the pod**
- Why use it (example reason):
- Keep expensive/special nodes (GPU, high memory, etc.) reserved for AI/ML workloads.
- Non-AI workloads can run on other nodes.

## Taint/Toleration “effect” types (as listed)

# 
- Tolerations include:
- key/value (and operator)
- **effect** (scheduling behavior)
- Effects named:
1. `NoSchedule`
2. `PreferNoSchedule`
3. `NoExecute`
- Differences (as explained):
- **NoSchedule**: applies to **new pods** (blocks scheduling of new pods without toleration).
- **NoExecute**: applies to **existing and new pods** (can evict existing pods that don’t tolerate the taint).
- **PreferNoSchedule**: tries to apply, but *“does not guarantee”* enforcement.

### kubectl (taints/tolerations/node labeling workflow)

# 
- Speaker uses kubectl commands to taint nodes, inspect nodes/pods, generate YAML, and apply changes.

## Demo: Taint worker nodes and observe Pending pod

# 
1. Cleanup existing pods (nginx/redis from earlier).
2. Confirms nodes:
- 3 nodes total: 1 control plane + 2 workers.
3. Adds taints to worker nodes:
- Command pattern shown:
- `kubectl taint node <node> GPU=true:NoSchedule`
- Speaker fixes a spelling/case mistake (`NoSchedule`).
4. Verifies taint exists:
- `kubectl describe node <node>` and grep for **Taints**.
5. Attempts to schedule an nginx pod:
- `kubectl run nginx --image=nginx`
- Pod stays **Pending**.
6. Describes the pending pod to see why:
- Message includes:
- control-plane node has its default taint (`kubernetes.io/control-plane`)
- both worker nodes have **untolerated taint** `GPU=true`
- Conclusion: pod can’t be scheduled anywhere without toleration.

## Demo: Add toleration via YAML (Redis example)

# 
- Generates a Redis pod manifest:
- `kubectl run redis --image=redis --dry-run=client -o yaml > redis.yml`
- Adds tolerations under `spec` (same level as restartPolicy):
- Uses the documented structure:
- key, operator, value, effect
- Example toleration used:
- key: `GPU`
- operator: `Equal`
- value: `true`
- effect: `NoSchedule`
- Applies:
- `kubectl apply -f redis.yml`
- Verifies:
- Pod is **Running**
- `kubectl get pods -o wide` shows it placed on a worker node (it can tolerate the taint).

## Demo: Remove a taint to allow the previously pending nginx pod

# 
- Removes the taint from worker node 2:
- Uses the same taint command but with a **trailing `-`** to delete it (speaker notes this was the missing piece).
- Confirms node taint is gone (describe node).
- Checks pods again:
- The nginx pod becomes scheduled.
- `kubectl get pod -o wide` shows nginx on worker node 2.

## Why taints/tolerations aren’t a “guarantee” of placement

# 
- Speaker emphasizes:
- Taints/tolerations **restrict** what can land on a node.
- They do **not guarantee** that a tolerating pod will land on that tainted node; it could still go to other nodes if allowed.

## Node Selectors: choosing a node via labels (concept)

# 
- Node selectors shift “choice” to the pod:
- *“instead of node make that decision… [we] give the decision to the pod to which node it has to be deployed on.”*
- Mechanism:
- Add **labels** to nodes.
- In the pod spec, use `nodeSelector` to match those node labels.
- Speaker compares with taints:
- If a node lacks the matching label, the pod won’t schedule there.
- Also notes taint still matters (a tainted node would reject pods without toleration).

## Demo: nodeSelector with node labels (and troubleshooting)

# 
1. Cleans up and creates a new pod manifest:
- `kubectl run nginx-new --image=nginx --dry-run=client -o yaml > new-nginx.yml`
2. Adds `nodeSelector` in pod spec:
- Example label match used:
- `GPU=false`
3. Applies the manifest:
- Initially it appeared running, but speaker realizes the node label had been removed earlier and the pod was already running from before.
4. Deletes the pod and re-applies:
- Now the pod is **Pending**.
5. Describes the pending pod:
- Error includes:
- control-plane taint issue
- and **“nodes did not match the pod’s node affinity/selector”**
- Reason: no node has the required label.
6. Labels a node:
- Command pattern:
- `kubectl label node <node> GPU=false`
7. After labeling:
- Pod schedules successfully.
- `kubectl get pods -o wide` shows nginx-new placed on the labeled worker node.

## Key takeaway: taints/tolerations vs nodeSelector

# 
- Taints/tolerations:
- *restriction applied on the node* (node decides what it accepts).
- Node selector:
- pod decides where it wants to go by matching node labels.

## Node selector limitation (and what’s next)

# 
- Limitation stated:
- can’t use expressions / logical AND / multiple complex conditions.
- Next topic preview:
- **node affinity and anti-affinity** (to handle richer scheduling rules).
- Assignment:
- practice tasks in **day 14** folder (GitHub repo).
- support via Discord + YouTube comments.

# 

# 

# 

#