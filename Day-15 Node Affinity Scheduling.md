# Kubernetes (CKA 2024, Video 15): Node Affinity Scheduling + Compare with Taints/Tolerations

## What this video covers + what’s next

# 

- Video #15 in the **CKA 2024** series.
- Focus: **node affinity** (speaker calls it a “very important scheduling concept”).
- Includes:
- concept explanation + demo
- *difference between affinity and taints/tolerations* (from the prior video)
- Assignment: sample tasks in the GitHub repository (day 15 folder).
- Engagement target: **170 comments** and **170 likes** in 24 hours.
- Next video preview: **resource requests and limits**.

## Why node affinity (limitations of taints/tolerations)

# 
- Recap of prior approach:
- node has taint `GPU=true`
- pod has toleration `GPU=true` plus an effect
- Limitations stated:
- *“we cannot add multiple conditions and expressions”*
- even if toleration matches taint, it *“does not prevent”* the pod from being scheduled on other nodes that don’t have that taint
- taints/tolerations let a node restrict/accept pods, but don’t provide a guarantee that a pod lands on a specific node
- So: use an additional concept: **node affinity**.

## Node affinity concept (board example with disk labels)

# 
- Setup:
- 3 nodes: node1, node2, node3
- labels:
- node1: `disk=HDD`
- node2 & node3: `disk=SSD`
- Goal described:
- disk/IO intensive workloads should run on nodes with SSD (node2 or node3), not node1.
- Example pods with different affinity rules (operators highlighted):
- `disk != SSD` → schedules on node1 (because condition is satisfied)
- `disk = SSD` → schedules on node2 or node3
- `disk in [SSD, HDD]` → can schedule on node1 or node3 (speaker example to show multiple values)
- Point made:
- node affinity matches **node labels** against **pod’s affinity rules**, using operators like `In`, `NotIn`, etc.

## Affinity “properties” (required vs preferred) + “ignored during execution”

# 
- Two key node affinity types shown (speaker calls them “properties”):
1. `requiredDuringSchedulingIgnoredDuringExecution`
2. `preferredDuringSchedulingIgnoredDuringExecution`
- Shared meaning of the last part:
- *“ignored during execution”* = if the pod is already scheduled, later changes to node labels won’t impact that running pod; it affects only newer pods scheduled later.
- Difference between the two (speaker repeats for clarity):
- **Required during scheduling**:
- pod schedules *only if* label/operator match exists
- if nothing matches, pod stays **Pending** and describe shows a warning like “no affinity matches”
- **Preferred during scheduling**:
- scheduler tries to match, but if it can’t (label missing or resources), it *“even then… will schedule the pod”* on any available node

### Kubernetes documentation (node affinity reference)

# 
- Speaker copies the node affinity YAML snippet from Kubernetes documentation to build the manifest.

## Demo 1: `requiredDuringSchedulingIgnoredDuringExecution` (pod stays Pending until node label exists)

# 
1. Creates `affinity.yaml` (copying a prior pod YAML and removing tolerations).
2. Adds `affinity` under `spec` (same level where tolerations were earlier).
3. Initially hits an error:
- `unknown field spec.affinity`
- Fix: add `nodeAffinity` under `affinity` (because `affinity` can include more than just node affinity).
4. Uses this rule:
- match expression:
- key: `disktype`
- operator: `In`
- values: `SSD`
5. Applies the manifest:
- pod is **Pending**
6. `kubectl describe pod ...` shows:
- 1 node excluded due to control-plane taint
- other nodes: *“did not match pod’s affinity or selector”*
7. Labels a worker node:
- `kubectl label node <worker> disktype=SSD`
8. After labeling:
- pod becomes **Running**
- confirmed it runs on the labeled worker node (`kubectl get pods -o wide` / describe)

## Demo 2: `preferredDuringSchedulingIgnoredDuringExecution` (schedules even without matching label)

# 
1. Creates a second file (named like `affinity2.yaml`) and changes:
- pod name to something like `redis-new`
- switches from **required** to **preferred**
- sets value to something not present (example: `SDD`)
2. Learns the required syntax for preferred:
- needs a `weight` and a `preference` block with `matchExpressions`
3. Applies:
- pod schedules successfully (container creating → running)
4. Takeaway stated:
- preferred tries to match, but if it can’t, it still schedules on an available node.

## Demo 3: `Exists` operator (match label presence even if value blank)

# 
- Speaker highlights one operator: `Exists`
- Changes:
- operator: `Exists`
- removes `values`
- Labels node with a blank value:
- label present but value is null (speaker shows it as `disktype=<null>` / no value)
- Applies a pod using `Exists`:
- pod schedules onto that node because the label key exists, regardless of value.
- Mentions other operators are available in docs (e.g., `NotIn`, equals).

## Quick difference: node affinity vs taints/tolerations (combined approach)

# 
- Example setup:
- nodes with taints/colors: green node tainted `color=green`, blue node tainted `color=blue`, and one node with no taint
- a pod with toleration `color=green`
- Problem described:
- with only taints/tolerations, scheduler might place the “green-tolerating” pod onto an untainted node (because it’s allowed).
- Solution described:
- use **both**:
- taints/tolerations to restrict what a node accepts
- node affinity to constrain where the pod should go (via node labels like `GPU=false` etc.)
- Summary statement:
- *“we use both of those to make sure our nodes are only accommodating the pods that are meant for it”*
- Use case emphasis:
- dedicated “large” or specialized nodes (GPU/AI-ML, high performance, data warehousing) often require combining both controls.

## Wrap-up

# 
- Speaker encourages practice and re-watching if needed.
- Support: Discord community + YouTube comments.
- Reminder: complete like/comment targets; next video expected within 24 hours or after target completion.

# - 

# 

# 

# 

#