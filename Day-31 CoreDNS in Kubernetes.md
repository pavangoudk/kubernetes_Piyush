# CKA 2024 Video 31: CoreDNS in Kubernetes (Service Discovery, kube-dns, and Troubleshooting)

## Intro: topic + prerequisite

# 

- Host (P) introduces **video #31** in the CKA 2024 series: **CoreDNS concepts** and *“how DNS works in Kubernetes.”*
- Prerequisite recommendation:
- if you’re not clear on DNS fundamentals (e.g., what happens when you type `www.google.com`), watch the previous video (#30) with guest **Pgur**.
- Engagement target:
- aims for **100 likes** and **100 comments**.
- Moves into a hands-on connectivity test to demonstrate Kubernetes DNS.

## Setup: two pods + two services (same naming pattern)

### ### kubectl

# 

- He has deployed two pods in the **default namespace**:
- `nginx` pod
- `nginx1` pod

(both use the **nginx image**)
- He has exposed them with two services and kept the names aligned to show relationships:
- service `nginx` → exposes the nginx pod
- service `nginx1` → exposes the nginx1 pod
- Runs `kubectl get svc` to show both services exist.

## Test 1: service-name DNS resolution fails (curl to hostname fails)

### ### kubectl exec

# 

- Goal: show that pods/services should be reachable “with the help of their name.”
- He execs into the `nginx` pod and runs:
- `curl nginx1`
- Result:
- the curl command **hangs/gets terminated** and does **not** return output.
- He frames the likely cause:
- could be DNS name → IP resolution failure.

## Test 2: curl succeeds by ClusterIP (proves DNS issue, not service routing)

### ### curl

# 

- He identifies `nginx1` service’s **ClusterIP** and tries:
- `curl <nginx1-service-clusterIP>`
- Result:
- it **returns output successfully**.
- Conclusion:
- service works by IP, but **DNS is not resolving** service name to IP.
- He ties it to DNS records:
- service name/hostname is “nothing but a DNS record” mapping name → IP.

## Root cause check: CoreDNS pods aren’t running

### ### CoreDNS (Kubernetes DNS server)

# 

- He explains:
- Kubernetes runs DNS via pods named **CoreDNS**.
- If you can reach services by IP but not by name, there’s likely an issue with the DNS server.
- Checks DNS components:
- `kubectl get pods -n kube-system` → CoreDNS pods are **not present**.
- Checks deployment:
- `kubectl get deploy -n kube-system`
- sees **coredns** deployment exists but has **0 replicas available**.

## Fix: scale CoreDNS deployment back to 2 replicas

### ### kubectl scale

# 

- Runs:
- `kubectl scale deploy coredns --replicas=2 -n kube-system`
- Confirms readiness:
- deployment becomes ready
- `kubectl get pods -n kube-system` shows **two CoreDNS pods running**

## Retest: service-name curl now works

### ### kubectl exec

# 

- Repeats the earlier curl-by-service-name test.
- Result:
- `curl nginx1` now **returns a reply**, confirming CoreDNS fixed service-name resolution.

## CoreDNS vs kube-dns (service name clarification)

### ### kube-dns Service

# 

- He shows DNS is exposed as a Kubernetes Service in kube-system:
- `kubectl get svc -n kube-system`
- service name shown: **kube-dns**
- Clarification (close paraphrase):
- *CoreDNS is the DNS server software running as pods; kube-dns is the Kubernetes Service for CoreDNS.*
- Notes:
- kube-dns service listens on **port 53** and has a **ClusterIP**.

## How pods know where to send DNS queries

### ### /etc/resolv.conf (inside a pod)

# 

- He execs into a pod (e.g., `nginx`) and inspects:
- `/etc/resolv.conf`
- Key points he highlights:
- The `nameserver` entry is automatically created for every new pod.
- It points to **10.96.0.10** (example shown), which he states is the **ClusterIP of the kube-dns service**.
- He points out the `search` domains:
- entries like `cluster.local`, `svc`, `default.svc.cluster.local`, etc.
- describes them as “adding more aliases” to service naming.

## Why /etc/hosts is not the solution at scale

### ### /etc/hosts

# 

- He references `/etc/hosts` and shows pod IP mapping can exist there.
- Explains what manual /etc/hosts editing would imply:
- you *could* add entries to reach other pods/services
- but it’s “impossible” to manage manually with:
- 100 pods, thousands of services, many nodes
- Takeaway:
- DNS centralizes name resolution so you don’t manually update every pod.

## CoreDNS configuration: ConfigMap mounted into CoreDNS pods

### ### kubectl describe pod (CoreDNS)

# 

- Describes a CoreDNS pod and observes mounts:
- service account secret mount (expected)
- a mount for CoreDNS configuration:
- mounted to **`/etc/coredns`**
- He traces it to a ConfigMap volume:
- volume type: **ConfigMap**
- ConfigMap name: **coredns**

### ### CoreDNS ConfigMap (Corefile)

# 
- Lists ConfigMaps in kube-system and describes the **coredns** ConfigMap.
- Notes it contains a **Corefile** with plugins/settings, including (as he narrates):
- `errors` (*redirect errors to stderr*)
- `health` (mentions “lameduck 5s” style health behavior)
- `ready`
- `kubernetes cluster.local` (cluster DNS domain)
- an entry referencing “insecure” for backward compatibility with kube-dns
- `prometheus :9153` (metrics exposed on port **9153**)
- `forward . /etc/resolv.conf` (forwards upstream using the pod’s resolv.conf)
- mentions behavior like load balancing “round robin” and reload on config changes
- He notes you can also view this file from inside the pod at its mounted location.

## Kubernetes docs task suggestion

# 
- He references the Kubernetes documentation DNS task page (says he’ll add it to the GitHub repo).
- Suggests practicing:
- deploy a pod with DNS utils
- run `nslookup`
- check `/etc/resolv.conf`
- check logs (`kubectl logs`)
- check service endpoints for kube-dns
- review ConfigMap settings

## When CoreDNS won’t come up: check the CNI (network add-on)

### ### CNI / Network add-on dependency

# 

- Key statement:
- *CoreDNS only works when a network add-on is installed.*
- In his cluster:
- he uses **Calico** as the networking add-on.
- notes seeing components like **tigera-operator** (used to install Calico).

### AWS EC2-specific Calico issue + required setting

### ### AWS EC2 (source/destination check)

# 

- He mentions a real issue he faced:
- Calico worked on **kind**, but not on **kubeadm** on AWS EC2 until fixes were applied.
- Fix #1 (AWS console):
- for each instance in the cluster:
- go to **Actions → Networking**
- **disable “Source/destination check”**
- he emphasizes: do this on *all VMs part of the cluster*.

### Calico daemonset detail (IP autodetection)

### ### DaemonSet (calico-node)

# 

- He checks daemonsets and highlights **calico-node**.
- Notes:
- calico-node pods may not come up unless a setting is correct.
- Fix #2:
- ensure **IP autodetection** is enabled (he points to this field while describing the daemonset).
- He adds:
- behavior can be inconsistent (“sometimes it works sometimes it doesn’t”)
- configuration depends on the installation method; he offers to help if viewers get stuck.

## Wrap-up + next video teaser

# 
- Recap of what viewers should now understand:
- what CoreDNS is
- what kube-dns is
- how DNS works in Kubernetes (service name → IP)
- why a “local DNS server” is needed in Kubernetes
- Teases **video #32**:
- more Kubernetes networking topics:
- **CNI**, network add-ons, container runtime concepts, etc.
- mentions an “amazing guest” with CNCF-related experience.
- Encourages comments (including guessing the guest’s name) and closes.

#