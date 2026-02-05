# CKA 2024 Video 37: Troubleshooting Kubernetes App Failures (Voting App: Services, Selectors, Ports, NetworkPolicy)

## Intro: exam-focused troubleshooting practice

# 

- Host (PE) introduces **Video #37** in the CKA 2024 series.
- Goal: **application failure troubleshooting** “from CKA exam perspective.”
- Mentions a GitHub repo is prepared so viewers can:
- log into the server
- deploy the app
- fix issues hands-on (errors intentionally introduced).
- Engagement target: **100 likes + 100 comments** in 24 hours.

## Sample app overview (before deploying)

# 
- App used: **Example Voting App**, forked from **Docker Samples**.
- Host forked it and:
- made changes “from CKA perspective”
- *introduced errors* so learners can find and fix them.

### ### Docker Compose (brief detour)

# 
- Repo contains:
- `k8s-specification/` (Kubernetes manifests; main focus)
- `docker-compose.yml` (not used for this demo)
- Host explains what Docker Compose is:
- a **single YAML** defining multiple services/containers.
- Mentions Compose capabilities:
- service dependencies (e.g., vote depends on redis being healthy)
- commands, volumes, ports, networks, etc.
- Says they’ll revisit Compose after the CKA series.

## Application architecture (application-level flow)

# 
- There are **5 containers / microservices**:
1. **vote** (frontend) — *Python-based*
2. **result** (frontend) — *Node.js-based*
3. **redis** — in-memory cache / can act as messaging component
4. **worker** — consumes votes from redis
5. **postgres** — database storing votes
- Flow described:
- User votes on **vote** UI
- **redis** collects votes from vote frontend
- **worker** consumes votes from redis and stores into **postgres**
- **result** frontend reads from postgres and shows results

## Kubernetes architecture mapping (pods + services + ports)

# 
- Same 5 workloads deployed as **5 pods**:
- vote pod
- result pod
- redis pod
- worker pod
- postgres (db) pod

### External access

# 
- Two frontend apps are exposed via **NodePort**:
- vote app on **31000**
- result app on **31001**

### Internal service-to-service communication

# 
- Uses **ClusterIP** services for internal connectivity:
- vote → redis (ClusterIP)
- worker → postgres (ClusterIP)
- result → postgres (ClusterIP)
- Worker is described as queue/messaging consumer and “doesn’t have to be exposed” via a service port for external access.

## Deploying the app manifests

### ### kubectl apply

# 

- Host already cloned the repo.
- Goes into `k8s-specification/` and applies everything:
- `kubectl apply -f .`
- Resources created include:
- **Deployments** (5 of them)
- **Services** (4 app services + default Kubernetes service visible)
- **NetworkPolicy** (also created)

### Verification checks

### ### kubectl get

# 

- `kubectl get deploy` → shows **5 deployments** up/healthy
- `kubectl get pods` → pods up/healthy
- `kubectl get svc` → shows:
- two NodePort services (31000, 31001)
- remaining are ClusterIP
- `kubectl get netpol` → shows a NetworkPolicy present, applied to redis via podSelector

## Issue 1: Vote UI not reachable on NodePort

# 
- Host tries to access the vote app via node public IP + `:31000`:
- tries worker node IPs (notes ClusterIP isn’t what you hit externally)
- still not reachable
- Since pods are running, host suspects **service** issue.

### ### kubectl describe svc (vote)

# 
- Describes vote service and observes:
- Type: NodePort
- Ports: internal 5000, external 31000, **TargetPort: 80**
- **Endpoints: missing**
- Host states endpoints are what “exposes the application,” and missing endpoints indicates a mismatch.

### ### kubectl get svc -o yaml (inspect selector)

# 
- Finds service selector mismatch:
- Service selector uses: `app: votes` (with an extra “s”)
- Checks pod labels:
- `kubectl get pods --show-labels`
- vote pod label is `app=vote`

### Fix

### ### kubectl edit svc

# 

- Edits the vote service:
- changes selector from `app: votes` → `app: vote`
- Re-describes service:
- Endpoints now appear.
- Verifies endpoints object exists:
- `kubectl get endpoints` (or `kubectl get ep`)
- Retests NodePort 31000:
- vote UI now works.

## Issue 2: Result UI not working (service targetPort mismatch)

# 
- Result page on **31001** keeps loading / times out.

### ### kubectl describe svc (result)

# 
- Service has:
- endpoints present
- selector `app=result` matches pod label
- Host checks if service TargetPort matches container port.

### ### kubectl get pod -o yaml (result)

# 
- Result pod exposes:
- `containerPort: 80`
- port name field is present (host notes naming ports can be exam-relevant)
- Result service TargetPort is **8080** (mismatch).

### Fix

### ### kubectl edit svc (result)

# 

- Changes `targetPort: 8080` → `targetPort: 80`
- Retests:
- result page now loads (shows result UI, though initially default/uninteresting output).

## Issue 3: Voting submission hangs (vote → redis blocked)

# 
- Vote UI loads, but submitting a vote (selecting cats/dogs) keeps spinning and doesn’t accept.

### Check redis service (seems fine)

### ### kubectl describe svc (redis)

# 

- Redis service is ClusterIP on **6379**
- Selector `app=redis` matches redis pod label
- Endpoints present; no events → service looks correct.

### Suspect NetworkPolicy (traffic restriction)

# 
- Host recalls NetworkPolicy acts like firewall rules inside cluster.

### ### kubectl describe netpol

# 
- NetworkPolicy applied to redis podSelector: `app=redis`
- It allows **ingress** to redis:
- ports: “any”
- **from pods with label** `app=front-end`
- Meaning (host paraphrase with diagram):
- only pods labeled `app=front-end` may reach redis
- all other sources are denied by default.

### Validate frontend pod labels

# 
- `kubectl get pods --show-labels`
- Voting pod label is `app=vote`, not `app=front-end`

### Fix

### ### kubectl edit netpol

# 

- Updates the allowed-from selector:
- changes matchLabels to `app=vote`
- Re-describes netpol:
- now allows ingress from pods labeled `app=vote`
- Retest voting:
- vote submission is accepted.

## Remaining issue: Results not updating (app-specific limitation mentioned)

# 
- After fixing NetworkPolicy, host expects results to update but they don’t.
- Investigates briefly:

### Checks result pod logs

### ### kubectl logs

# 

- Initially sees error like:
- “relation votes does not exist” (noted as seen in logs)

### Restarts result pod

### ### kubectl delete pod (deployment-managed)

# 

- Deletes the result pod; Deployment recreates it.
- New logs show:
- “waiting for DB” (suggests DB connectivity/wait state)

### Checks deployments (result, worker, db)

# 
- Edits/inspects:
- result deployment (doesn’t find an obvious DB connection string there)
- worker deployment (no obvious issue found)
- db service (ClusterIP on 5432)
- db deployment includes env vars like:
- `POSTGRES_USER`, `POSTGRES_PASSWORD` (host notes this isn’t best practice; normally secrets/vaults are used)
- volume mount for persistence (because it’s a database)

### Restarts worker pod

# 
- Deletes worker pod to restart it, but results still don’t update as expected.

### Conclusion about the remaining behavior

# 
- Host says he checked open issues and suspects something specific to the Linux image/environment.
- He concludes:
- for this demo, fixing the identified Kubernetes misconfigurations is “more than enough” for exam perspective.
- real-world app troubleshooting can have many more issues.

## Closing: next topic preview

# 
- Host thanks viewers and says next video will cover:
- **control plane failures troubleshooting** (another “interesting concept”).

# - 

# 

# 

#