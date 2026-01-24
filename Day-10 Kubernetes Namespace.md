- # Kubernetes Namespaces (CK 2024 Series #10): Why They Matter + Connectivity Demo

## Intro & goals of the video

## 

- Video focus: **Namespaces** in Kubernetes—*why they’re needed*—plus a hands-on task demonstrating **connectivity between services/pods across namespaces**.
- Speaker sets engagement targets (comments/likes) and moves into the technical content.

## What are namespaces, and why are they needed?

## 
- *Namespaces “provide you another layer of isolation within your cluster so that you can separate your objects and resources within a cluster.”*
- Default behavior:
- *“When you create any resource and when you don’t specify the namespace name it by default gets created in a default namespace.”*
- Kubernetes-created namespaces (on cluster provisioning):
- *“Kubernetes itself create a few namespaces by itself… for example… `kube-system`.”*
- Reason given: control plane components live there by default, so keeping it separate reduces accidental changes.

### Why isolation helps (speaker’s rationale)

## 
- Avoiding mistakes:
- If everything is in one namespace, it’s easier to accidentally delete/modify the wrong resource.
- With namespaces, you must explicitly target the namespace, making it “easier to manage.”
- Security and access control:
- *“You can assign different permissions and different RBACs to each of the namespaces.”*

### Example scenario (namespaces and environments)

## 
- Speaker describes separating environments using namespaces, e.g.:
- `kube-system` (control plane)
- a `test` namespace
- a `prod` namespace
- Within a namespace:
- Example pods (nginx and redis) can interact using short names/hostnames.
- Across namespaces:
- If a pod needs to talk to a service in another namespace, it can’t rely on just the short name; it must use:
- *“something called as an FQDN or fully qualified domain name.”*

## ### kubectl (k) — Commands used to inspect namespaces and resources

### List namespaces

## 

- `kubectl get namespaces` (speaker uses alias `k`)
- Notes from output:
- `default` namespace exists
- system namespaces exist (e.g., `kube-system`, `kube-public`, `kube-node-lease`, `local-path-storage`)

### Inspect what’s inside a namespace

## 
- Long form:
- `kubectl get all --namespace kube-system`
- Short form:
- `kubectl get all -n kube-system`
- Speaker observes control-plane-related items in `kube-system`, including pods/services like:
- `coredns`, `etcd`, `kube-apiserver`, `kube-controller-manager`, `kube-proxy`, `kube-scheduler`
- and a DNS service (CoreDNS) described as resolving names/IPs inside the cluster.

### Default namespace behavior

## 
- With `-n default` vs no `-n`:
- Speaker shows it returns the same results because *“by default it will get the information from the default namespace.”*

## ### Namespace creation — Declarative vs imperative

### Declarative (YAML)

## 

- File created: `ns.yml`
- Minimal fields shown:
- `apiVersion: v1`
- `kind: Namespace`
- `metadata.name: demo`
- Applied with:
- `kubectl apply -f ns.yml`
- Speaker corrects a typo: the `V` in `v1` must be lowercase `v1`? (In the transcript, the speaker fixes “V has to be capital V,” but then proceeds successfully—net takeaway: fix API version formatting as needed.)

### Imperative (one-liner)

## 
- Delete namespace:
- `kubectl delete ns demo`
- Create namespace quickly:
- `kubectl create ns demo`
- Speaker takeaway:
- Sometimes imperative commands are faster than writing YAML.

## ### Deployments in two namespaces (demo vs default)

### Create deployment in `demo` namespace

## 

- Command pattern used:
- `kubectl create deploy nginx-demo --image=nginx -n demo`
- Speaker emphasizes:
- If you forget `-n demo`, it will create in `default`.

### Create deployment in `default` namespace

## 
- Command pattern used:
- `kubectl create deploy nginx-test --image=nginx`
- (No namespace flag → goes to `default`)

### Same names across namespaces

## 
- Speaker note:
- You *can* create the same object name in multiple namespaces because they’re isolated.

## ### Pod-to-pod connectivity test (using Pod IPs)

### Get pod IPs

## 

- `kubectl get pods -o wide` (run in each namespace)
- Speaker records two pod IPs (example values shown):
- demo pod IP: `10.244.1.7`
- default pod IP: `10.244.2.7`

### Exec into pods

## 
- Enter a shell:
- `kubectl exec -it <pod> -- sh`
- For demo namespace: add `-n demo`

### Test connectivity via curl

## 
- From one namespace’s pod to the other namespace’s **pod IP**:
- `curl <other-pod-ip>`
- Result:
- Returns nginx welcome page (“Welcome to nginx”), indicating connectivity works.

### Note about ping

## 
- Speaker mentions:
- `ping` isn’t installed in the lightweight nginx image by default; you’d need to install it.
- Uses `curl` instead to validate connectivity.

## ### Scaling deployments

## 
- Speaker scales both deployments to 3 replicas:
- `kubectl scale --replicas=3 deploy nginx-demo -n demo`
- `kubectl scale --replicas=3 deploy nginx-test`
- Verifies with:
- `kubectl get pods` (and `-n demo` where needed)

## ### Exposing deployments as services

### Create service in `demo` namespace (imperative)

## 

- Uses `kubectl expose` to create a service for the deployment.
- Command pattern shown (speaker iterates to correct syntax):
- `kubectl expose deployment nginx-demo --name=svc-demo --port=80 -n demo`
- Verify:
- `kubectl get svc -n demo`

### Create service in `default` namespace

## 
- Command pattern:
- `kubectl expose deployment nginx-test --name=svc-test --port=80`
- Verify:
- `kubectl get svc`
- Speaker notes:
- Default namespace has the pre-created `kubernetes` ClusterIP service plus the new `svc-test`, each with different ClusterIP addresses.

## ### Service-to-service (cross-namespace) name resolution: short name fails

### Attempt curl using only service short name

## 

- From a pod in `demo`, try:
- `curl svc-test`
- From a pod in `default`, try:
- `curl svc-demo`
- Result in both cases:
- *“Could not resolve host name”* (name doesn’t resolve across namespaces by short name)

## ### Fully Qualified Domain Name (FQDN) fix (cross-namespace)

### Check DNS search paths inside the pod

## 

- Speaker inspects:
- `cat /etc/resolv.conf`
- Observations:
- In `demo` namespace, search domains include something like:
- `demo.svc.cluster.local`
- In `default` namespace, search domains include:
- `default.svc.cluster.local`
- Speaker takeaway:
- *Hostnames are namespace-wide, not cluster-wide.*

### Use FQDN to reach service in another namespace

## 
- From `demo` namespace pod → access `svc-test` in `default`:
- `curl svc-test.default.svc.cluster.local`
- From `default` namespace pod → access `svc-demo` in `demo`:
- `curl svc-demo.demo.svc.cluster.local`
- Result:
- Successful nginx response in both directions.

## Final recap (speaker’s conclusions)

## 
- Setup created:
- Two namespaces: `default` and `demo`
- Each has a deployment scaled to 3 pods
- Each deployment exposed via a service
- Connectivity findings:
- **Pod IPs are reachable cluster-wide** (even across namespaces).
- **Service short names don’t resolve across namespaces.**
- Cross-namespace service access requires **FQDN**:
- `<service>.<namespace>.svc.cluster.local`
- Within the same namespace, the service short name is sufficient.

## Closing

## 
- Speaker wraps up the namespaces lesson and previews the next video topic:
- **Multi-container pods**, plus related concepts like **commands** and **arguments**.
