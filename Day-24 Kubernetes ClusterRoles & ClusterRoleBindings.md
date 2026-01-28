# Kubernetes ClusterRoles & ClusterRoleBindings (CK 2024, Video 24)

## Intro + what this video covers

# 

- Speaker introduces **video #24** in the CK 2024 series.
- Topic: **ClusterRoles** and how to implement them.
- Plan: compare **ClusterRole/ClusterRoleBinding** with **Role/RoleBinding** from the previous video.
- Mentions a hands-on task in the GitHub repo under **day 24** folder.

## Quick recap (security topics covered so far)

# 
- Speaker recaps earlier concepts:
- Authentication vs authorization
- Symmetric encryption, asymmetric encryption, certificates (high-level recap)
- Authorization modes overview:
- Attribute-based authorization (ABAC) (briefly)
- Role-based access control (RBAC) deep dive (previous video)
- Node authorization and webhook (overview)

## RBAC recap: Role + RoleBinding (namespace-scoped)

# 
- Role concept:
- A **Role** has permissions like “list pods, watch pods, get pods.”
- Example role name referenced: “pod reader.”
- RoleBinding concept:
- A **RoleBinding** attaches that Role to a **user** or **group**.
- Result: user/group gets the Role’s permissions.

### Namespace scope key point

# 
- Speaker states:
- *Roles are “namespace scoped,”* meaning they apply only to resources within a namespace.
- Examples of namespace-scoped resources mentioned:
- pods
- deployments
- replica sets
- services (and others)

## Why ClusterRoles exist (cluster-scoped resources)

# 
- Speaker introduces **cluster-scoped resources**, giving examples:
- **namespaces** (cluster-wide)
- **nodes** (cluster-wide)
- “many more”
- To grant access to cluster-scoped resources:
- Use **ClusterRole** (not Role)
- Attach it using **ClusterRoleBinding**

### ClusterRole + ClusterRoleBinding (conceptual mapping)

# 
- ClusterRole can include permissions like:
- list nodes
- watch nodes
- get nodes
- ClusterRoleBinding attaches the ClusterRole to a user/group.
- Speaker summarizes the main difference:
- *Role/RoleBinding is namespace-scoped; ClusterRole/ClusterRoleBinding is cluster-scoped.*

### Note: Roles can still act “cluster-wide” across namespaces

# 
- Speaker adds nuance:
- A normal Role can effectively apply across all namespaces **if you don’t specify a namespace** (as they did in the prior video example).
- In that case, it’s applicable to all namespaces—but it’s still different from cluster-scoped resources like nodes/namespaces.

## ### kubectl `api-resources` (namespace vs cluster scope)

# 
- Speaker shows how to list which resources are namespaced vs cluster-scoped:

1. Namespaced resources:

- `kubectl api-resources --namespaced=true`
- Examples observed/mentioned: cronjobs, statefulsets, replicasets, deployments, services, pods, persistentvolumeclaims, configmaps, bindings, roles, etc.
2. Cluster-scoped resources:

- `kubectl api-resources --namespaced=false`
- Examples observed/mentioned: clusterroles, clusterrolebindings, CSINodes, storageclasses (not covered yet), ingressclasses, certificatesigningrequests (already used earlier), customresourcedefinitions, apiservices, webhooks, plus namespaces and nodes.

## Demo goal: give user Krishna permission to read nodes

# 
- Speaker uses the user **Krishna** (created in prior work).
- First, they test whether Krishna can get nodes:
- Uses the authorization check command “can I get nodes” as Krishna.
- Output highlights:
- nodes are **not namespace scoped**
- Krishna currently **does not** have permission

## ### Creating a ClusterRole (imperative method)

# 
- Speaker notes there are two methods: imperative and declarative; demo uses imperative.
- They run help to show syntax:
- `kubectl create clusterrole --help`
- Key flags shown/mentioned: `--verb`, `--resource`, optionally resourceName.
- They create a ClusterRole:
- Name: **node-reader**
- Verbs: **get, list, watch**
- Resource: **nodes**
- They attempt to type another example and hit an error:
- Using “read” as a verb fails because *“read is not a standard resource verb.”*
- Also hits “already exists” because node-reader was created already.

### Verify ClusterRole

# 
- They list ClusterRoles and grep for node-reader.
- They describe it:
- Resources: nodes
- ResourceName blank → applies to **all nodes**
- Verbs: get, list, watch

## ### Creating a ClusterRoleBinding (bind ClusterRole to Krishna)

# 
- Speaker uses help first:
- `kubectl create clusterrolebinding --help`
- Creates ClusterRoleBinding:
- Binding name: **reader-binding**
- ClusterRole: **node-reader**
- User: **Krishna**
- Verifies:
- Lists clusterrolebindings and greps reader-binding
- Describes reader-binding:
- User Krishna is bound to ClusterRole node-reader on the cluster

## Validate permissions (impersonation, then real context)

# 
1. Authorization check as Krishna:
- “can I get nodes as Krishna” → **yes**
2. Switch kubeconfig context to Krishna:
- `kubectl config use-context Krishna`
3. Run node commands:
- `kubectl get nodes` works (read access)
- `kubectl describe nodes` should work (speaker implies it should, since it’s read access)
- `kubectl delete nodes` fails:
- Forbidden error: user Krishna cannot delete nodes because delete wasn’t granted

## Wrap-up + next video

# 
- Speaker summarizes: ClusterRole + ClusterRoleBinding grant **cluster-level permissions** (demo: nodes) without granting destructive actions like delete.
- Reminds about the GitHub task for Day 24 and support via comments/Discord.
- Next video topic: **Service Accounts**.

# 

# 

# 

#