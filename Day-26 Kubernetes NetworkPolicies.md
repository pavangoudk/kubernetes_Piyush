# Kubernetes NetworkPolicies: Restrict Pod Traffic with Calico (CK 2024, Video 26)

## Intro + where this fits

# 

- Speaker introduces **video #26** in the CK 2024 series.
- Topic: **Network Policies**—what they are, why they’re needed, and how to implement them.
- This is the **last video in the security section**; next topics: storage, cluster installation, maintenance, troubleshooting.
- Mentions a GitHub repo task under **Day 26** folder and that important docs will be shared there.
- Like/comment target: **180 likes and 180 comments** in 24 hours.

## Why Network Policies are needed (network flow)

# 
- Speaker starts with a conceptual “network flow” example using a **3-tier web application**:
- **Web tier** (frontend) exposed to users on **ports 80/443** (HTTP/HTTPS)
- **App tier** (backend) runs business logic (example: Nginx or Node.js)
- **Database tier** runs a database (example: **MySQL**) on **port 3306**
- Flow described:
1. User accesses frontend (80/443)
2. Frontend calls backend internally (example port 80)
3. Backend connects to database (3306)
4. Responses return back in reverse direction
- Definitions in the speaker’s words:
- *Incoming flow is called “Ingress”*
- *Outgoing flow is called “Egress”*

## Kubernetes version of the same flow (pods + services)

# 
- Speaker maps the same 3-tier app onto Kubernetes:
- Three worker nodes (Node A, Node B, Node C)
- Pods:
- Frontend pod (Nginx)
- Backend pod (Nginx)
- MySQL pod
- Each tier is exposed via a **Service**:
- Frontend service: **port 80**
- Backend service: **port 80**
- DB service: **port 3306**
- Key Kubernetes networking behavior stated:
- *By default in Kubernetes, “all the pods… services everything can communicate with each other.”*
- Example consequence:
- Frontend can directly talk to DB (which the speaker says should be restricted in a production-grade design)

## ### CNI (Container Network Interface) / network provider

# 
- Speaker explains that pod-to-pod/service connectivity is implemented via a “service”:
- *“CNI or network provider (Container Network Interface)… responsible for making sure that pods are accessible within the Kubernetes cluster.”*
- CNI behavior described:
- Deploys **one pod on each node** (as a **DaemonSet**) to enable networking.
- Examples of CNI providers listed:
- Weave Net
- Flannel
- Calico
- Cilium
- Important point:
- CNIs are **Kubernetes add-ons** (not installed by default in vanilla Kubernetes).

## NetworkPolicies: what they do + dependency on CNI

# 
- Speaker’s definition-level description:
- *Network Policy implements “rules / policies / restrictions” in pod networking so only the pods that should communicate can communicate.*
- Enforcement detail:
- NetworkPolicies are **enforced by the CNI plugin**.
- Critical compatibility note:
- *Not every CNI supports NetworkPolicies.*
- Speaker specifically says:
- **Flannel** does **not** support network policies (as stated)
- **kindnet** (Kind’s default CNI) does **not** support network policies
- If you need NetworkPolicies, choose other plugins (examples given: Weave, Calico, Cilium)

## ### kind (Kubernetes in Docker): confirm default CNI and DaemonSet behavior

# 
- Speaker checks the current cluster:
- `kubectl get pods -n kube-system` shows **kindnet** pods
- `kubectl get daemonset -A` (or similar) shows **kindnet** as a DaemonSet with desired/current matching node count
- They show the kind cluster config used earlier:
- 1 control plane + 2 worker nodes
- HostPort mapping (example: 30001) to use NodePort services
- No explicit networking settings → kind created default CNI
- Updated kind config adds:
- `networking: disableDefaultCNI: true`
- Meaning: cluster will come up **without kindnet**, so they can install their own CNI

## Recreate cluster without default CNI (and why nodes become NotReady)

# 
- Speaker creates a new kind cluster (named like “CK-new”) using updated config.
- First attempt fails because NodePort was already bound by an existing cluster.
- They delete the old cluster to free the port, then recreate.
- After creation, nodes stay **NotReady**.
- They describe a node and see events:
- kubelet not ready
- *“container runtime network not ready… CNI plugin not installed”*
- Conclusion:
- Disabling default CNI requires installing a CNI to make the cluster functional.

## Installing a CNI with NetworkPolicy support (Weave Net attempt)

# 
- Speaker references Kubernetes docs: you need a network provider with NetworkPolicy support.
- They choose **Weave Net** and apply a DaemonSet YAML from its repo.
- Applying it creates multiple resources (speaker lists examples):
- DaemonSet
- Service account
- ClusterRole / ClusterRoleBinding
- RBAC resources (roles/rolebindings)
- They verify Weave pods are running and nodes become Ready.

## Demo setup: 3 pods + 3 services for frontend/backend/db

# 
- Speaker creates the demo app using a single manifest file containing multiple resources separated by `---`.
- Manifest contents (as described):
1. **Frontend pod**
- Label: `role=frontend`
- Runs Nginx on port 80
2. **Frontend service** on port 80
3. **Backend pod**
- Label: `role=backend`
- Runs Nginx on port 80
4. **Backend service** on port 80
5. **Database (MySQL) pod**
- Exposes port 3306
- Has environment variables (mentioned)
- Label matches MySQL selector used later
6. **DB service** on port 3306
- They apply the manifest and verify:
- Pods are Running
- Services exist:
- frontend (80)
- backend (80)
- db (3306)
- plus the default Kubernetes service

## Connectivity test before NetworkPolicy (frontend can reach backend and DB)

# 
- Speaker execs into the **frontend** pod.
- Tests:
- `curl backend:80` returns default Nginx page (service name used as hostname)
- For DB, installs telnet utility:
- `apt-get update`
- `apt-get install telnet`
- `telnet db 3306` connects successfully
- Outcome: frontend can reach both backend and DB by default.

## ### NetworkPolicy (YAML structure explained, then applied)

# 
- Speaker shows a generic NetworkPolicy structure:
- `apiVersion: networking.k8s.io/v1`
- `kind: NetworkPolicy`
- `metadata.name`
- `spec`
- `spec.podSelector.matchLabels` identifies which pods the policy applies to
- You can define **Ingress**, **Egress**, or both
- Ingress rule can allow only traffic from pods matching a label selector
- Then they move to the policy used for the demo:
- Policy name: **db-test**
- `podSelector.matchLabels: name=mysql` (matches the MySQL pod label in their manifest)
- `policyTypes: [Ingress]`
- `ingress.from.podSelector.matchLabels: role=backend`
- Meaning: only pods with `role=backend` can reach the selected MySQL pod
- `ports`: only allow **TCP 3306**

### Apply and inspect policy

# 
- Apply:
- `kubectl apply -f netpolicy.yaml`
- Verify:
- `kubectl get netpol`
- `kubectl describe netpol db-test`
- Description confirms:
- Applied to pods with label `name=mysql`
- Allows Ingress on port 3306 from pods with label `role=backend`
- *“Not affecting egress traffic”* since only Ingress rules were specified

## Test after policy: frontend blocked, backend should be allowed (issue appears)

# 
- Speaker execs into frontend pod again and tries:
- `telnet db 3306`
- Result: hangs/timeouts (blocked) — expected
- Then execs into backend pod:
- Installs telnet
- Tries `telnet db 3306`
- Unexpectedly, it does **not** connect (they begin troubleshooting)

## Root cause + fix: Weave Net compatibility issue → switch to Calico

# 
- Speaker explains they spent significant time troubleshooting and found the issue:
- Weave Net is **out of date** (last commit ~2 years ago per their check)
- They are using **Kubernetes 1.30**, and Weave may not be compatible with this Kubernetes version or their kind setup
- They attempted to switch plugins but cleanup wasn’t straightforward:
- Removing resources alone wasn’t enough; needed deeper cleanup on nodes
- Decision: create a new cluster and use **Calico**
- Speaker states Calico is popular and widely used by cloud providers (mentions GKE uses Calico or Cilium)

## Calico setup on a multi-node kind cluster (doc-guided)

# 
- Speaker references a Calico doc specifically for **multi-node kind**.
- Kind cluster config differences mentioned:
- Adds a **pod subnet range** to avoid clashes with existing subnets
- Keeps extra port mapping they used earlier
- Steps shown:
1. `kind create cluster --config <kind-calico.yaml> --name cka`
2. `kubectl get nodes` shows NotReady initially (Calico not installed yet)
3. Install Calico via “manifest way” (single apply command)
4. Verify Calico pods:
- `kubectl get pods -A -l k8s-app=calico-node`
- Notes these are DaemonSet pods (one per node)
- Uses watch until ready; becomes ready quickly (~43 seconds stated)

## Re-run the 3-tier demo on Calico (baseline connectivity)

# 
- Apply the same manifest again (3 pods + 3 services).
- Validate connectivity:
- Frontend → backend via curl works
- Frontend → DB via telnet connects
- Backend → frontend via curl works
- Backend → DB via telnet connects
- Confirms: default connectivity is open among pods/services.

## Apply NetworkPolicy again on Calico (now behaves as intended)

# 
- Apply the same NetworkPolicy YAML.
- Describe confirms:
- Allow traffic to DB port 3306 only from pods labeled `role=backend`
- Test results:
- Frontend → DB: telnet now times out (blocked) — expected
- Backend → DB: telnet connects (allowed) — expected

## Diagram recap: what changed with the policy

# 
- Before: all pods/services could communicate.
- After policy:
- Backend → DB is **allowed**
- Frontend → DB is **restricted** (blocked)
- Any other new service/pod in the cluster would also be blocked by default from DB unless explicitly allowed by policy.

## Production note: default-deny approach

# 
- Speaker mentions a stronger production pattern:
- Start with a **default deny** policy (deny all traffic)
- Then explicitly allow required flows “one by one” as needed.

## Wrap-up

# 
- Speaker recommends:
- Use **Calico** (and mentions Cilium as another option)
- Avoid Weave Net for kind in their setup due to being outdated
- Security section is now complete; next section is **storage**.
- Reminds about GitHub tasks and support via comments/Discord.

# 

# 

# 

# 

# 

#