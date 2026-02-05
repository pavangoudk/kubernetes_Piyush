# CKA 2024 Day 33: Kubernetes Ingress vs Ingress Controller (NGINX Demo + Troubleshooting)

## Intro: guest + why this topic

# 

- Host (Peush / PE) introduces **Day 33** of the CKA 2024 series with guest **Abhishek Bala**.
- They frame the topic as commonly confusing even for experienced folks:
- **Ingress** vs **Ingress Controller**
- They move to a whiteboard-style explanation, aiming to keep it simple.

## Why Ingress? (problem statement before definitions)

# 
- Abhishek starts with the typical Kubernetes deployment reality:
- apps run as **pods** (via Deployment/StatefulSet/DaemonSet).
- pods are reachable **inside** the cluster, not directly from outside.
- Services recap (why people use them):
- Services help with:
1. **Service identification** (and load balancing across pods)
2. **Exposing apps externally** when using:
- `NodePort`
- `LoadBalancer`

### Why `NodePort` is usually not preferred for end customers

# 
- NodePort typically exposes node ports on Kubernetes nodes.
- Kubernetes nodes are often accessible only inside an organization network, so it’s not the best choice for public customers.

### `LoadBalancer` service: common, but has disadvantages

# 
- Using `Service type: LoadBalancer` is the “obvious choice,” but Abhishek lists key drawbacks:

1. **Cloud-provider dependency**

- *Service type LoadBalancer asks the cloud provider to create a load balancer.*
- Example with AWS:
- in EKS / AWS-backed Kubernetes, it can create an **ALB** (Application Load Balancer) in the public/private subnet setup (as described).
- If a cloud provider doesn’t support it via Kubernetes **cloud controller manager**, it won’t work.

1. **Cost**

- Exposing many apps means many load balancers.
- Running multiple load balancers 24×7 can make billing “too high.”

1. **Limited control / security capabilities**

- You’re restricted to what the provider creates.
- You can’t easily say “I want NGINX instead of ALB,” or use specific vendor appliances (examples mentioned: F5, Cisco, Palo Alto).
- You get the load-balancing/security features the provider offers.

### VM-era comparison: features people expect

# 
- In the VM/physical-server world, you’d commonly place in front of apps:
- load balancers
- API gateways
- protections like DoS mitigation
- IP allow/deny lists
- WAF (web application firewall)
- Abhishek says Kubernetes needs an equivalent mechanism—*Ingress is introduced to solve this exact problem.*

## What “Ingress” means (networking definition)

# 
- Abhishek defines it in networking terms:
- *Ingress is the “inward traffic” coming to your application/cluster.*

## The three things to understand: Ingress resource, Ingress controller, load balancer

# 
Abhishek says implementing Ingress requires understanding three pieces:

1. **Ingress Resource (Kubernetes object)**
2. **Ingress Controller**
3. **Load Balancer**

## Why an Ingress Controller is needed (declarative config + dynamic pods)

# 
- In Kubernetes, pod counts and services change frequently (scaling, new deployments).
- Managing a traditional load balancer manually is not flexible.
- Approach:
- DevOps writes an **Ingress resource** (YAML, declarative)
- The **Ingress controller** watches it and configures the real load balancer behavior.

### Key clarification: Ingress Controller ≠ Load Balancer

# 
- Abhishek explicitly clarifies:
- *“Ingress controller is never a load balancer.”*
- *Ingress controller is a controller program (he calls it a “simple Go program”) built by vendors.*
- Examples of who provides controllers:
- NGINX, Traefik, Kong, F5, Amazon (ALB ingress controller), etc.

### The “three steps” summary (repeated for emphasis)

# 
1. Deploy **Ingress Controller**
2. Create **Ingress Resource**
3. Controller creates/configures the **Load Balancer** based on the Ingress config

Host confirms they can proceed to the demo.

## Reading a basic Ingress YAML (what’s inside)

### ### Ingress (Kubernetes manifest structure)

# 

- Abhishek shows a “very basic” Ingress YAML structure:
- `apiVersion: networking.k8s.io/v1`
- `kind: Ingress`
- `metadata` (+ mentions annotations are important; details later)
- `spec` is “crucial”
- `ingressClassName` (he says he’ll explain)
- `rules` that define routing
- Routing examples described:
- **Path-based routing**: when someone hits a path, forward to a service/port.
- **Host + path-based routing**: e.g., host `foo.bar.com` and path `/bar` routes to a particular service.

## Demo plan: full journey (controller → ingress → routing)

# 
- They will:
1. deploy an Ingress controller
2. create/load balancer routing via an Ingress resource
3. show the complete request flow

## App used in demo: simple Flask “Hello World”

### ### Flask

# 

- They use a simple Flask app from the course repo (host says Day 33 folder in the CKA GitHub repo).
- App notes:
- `app.py` returns “hello world”
- runs on port **80**
- Includes:
- `requirements.txt` (Flask listed)

### ### Docker (containerization)

# 
- Dockerfile:
- uses `python:3.7-slim`
- avoids advanced concepts like multi-stage builds (focus is Ingress).

## Building and pushing the container image (with troubleshooting)

### ### Docker CLI

# 

- Attempted `docker build -t <username>/cka-ingress-demo:<tag> .`
- Host notes Docker wasn’t installed (to mimic exam-like environments).
- They install/start Docker and address common issues:

#### Fixes shown during troubleshooting

# 
- Add user to Docker group:
- `sudo usermod -aG docker ubuntu`
- Start Docker service:
- `sudo systemctl start docker`
- Re-login / re-enter session for group change to apply (they “login back” to reflect permissions).
- Navigate to correct folder (they hit “no such file” when not in the right directory).
- Build succeeds.

### Why push to a registry

# 
- Abhishek explains:
- Kubernetes deployments don’t pull images from your local machine by default; they pull from a container registry unless configured otherwise.
- They run:
- `docker login`
- `docker push <image-tag>`
- Push succeeds; image is now in Docker registry.

## Deploy app to Kubernetes: Deployment then Service

### ### Deployment

# 

- In a `k8s/` directory, Abhishek has prepared YAML templates.
- Viewer action:
- replace the image name with their own pushed image.
- Apply deployment, verify:
- `kubectl get deploy`
- `kubectl get pods`
- Pod is running (“hello world application is running”).

### ### Service (ClusterIP)

# 
- They apply `service.yaml`.
- Important concept emphasized:
- **selector** must match pod **labels** for service discovery.
- They verify:
- `kubectl get svc` shows “hello world service” with a ClusterIP.

#### Recording gap note

# 
- Host mentions they had technical difficulties and resumed recording the next day.

### ### curl (service verification)

# 
- Abhishek validates service works internally:
- curl the **Service ClusterIP** from within the cluster environment.
- Output: “hello world”
- Reminder:
- because it’s **ClusterIP**, you must be on the cluster network to reach it.
- alternative would be NodePort for external curl, but they avoid it since the goal is Ingress.

## Create the Ingress resource (address missing until controller acts)

### ### Ingress Resource

# 

- Ingress YAML highlights:
- `apiVersion: networking.k8s.io/v1`
- `kind: Ingress`
- `spec.ingressClassName`
- `rules`:
- host: `example.com`
- path `/` routes to service `hello-world` on port **80**
- Apply ingress:
- `kubectl apply -f ingress.yaml`
- Check:
- `kubectl get ingress`
- Address is **not assigned** yet.
- Explanation:
- *Ingress resource alone does nothing unless an Ingress controller is installed and watching it.*

## Install an Ingress Controller (NGINX) + “two NGINX controllers” warning

### ### NGINX Ingress Controller

# 

- They choose NGINX as a popular controller.
- Abhishek points out there are **two** NGINX ingress controllers “in the market”:
1. one developed by the Kubernetes community (open source)
2. one developed/owned by the NGINX company (he mentions ownership context)
- They choose the open-source/community one and follow its docs.

### AWS install option and what gets installed

# 
- They use the AWS install manifest (copy/paste).
- Abhishek advises:
- review what each resource does (good learning):
- ServiceAccount, RBAC, ClusterRole/Bindings, ConfigMaps, Services, Deployments, Jobs, Admission components, etc.
- notes “validating admission webhook” wasn’t previously covered in the series, but viewers can read about it.

## IngressClassName: why it matters (avoid “race conditions”)

### ### IngressClassName

# 

- Abhishek explains why this field exists now:
- clusters may have multiple teams and multiple ingress controllers (e.g., Kong + NGINX).
- without specifying, multiple controllers might try to act on the same Ingress → “race condition” / routing issues.
- If `ingressClassName: nginx` (example), then only the NGINX controller should watch it.

### How to confirm controller’s class

# 
- They inspect the ingress controller pod arguments:
- controller starts with an argument indicating `--ingress-class=nginx` (as described).
- Key warning:
- *If you don’t mention ingressClassName, you can “mess up” the ingress configuration and make troubleshooting difficult.*

## Why the Ingress address didn’t populate: no cloud controller manager (kubeadm on EC2)

# 
- They check ingress again: address still not populated.
- They check logs:
- controller shows it found a valid ingress class and recognized the Ingress resource.
- Root cause stated:
- This is a kubeadm cluster on AWS EC2 **without cloud controller manager**, so LoadBalancer services don’t get external IPs automatically.

### Evidence: ingress-nginx service is LoadBalancer with External IP pending

# 
- They look at the ingress controller service:
- in ingress-nginx namespace
- service type is **LoadBalancer**
- external IP shows **Pending**
- Abhishek connects it to cloud controller manager absence.

## Workaround: change ingress controller Service type to NodePort

### ### kubectl edit (Service)

# 

- They edit the ingress controller service:
- change `type: LoadBalancer` → `type: NodePort`
- Verify service type change succeeded.

### Recreate the Ingress resource

# 
- They delete and re-apply the Ingress resource to ensure updates take effect.
- After some time:
- `kubectl get ingress` shows the **address/IP populated** (they note it can take time).

## Testing the Ingress: why direct IP curl returns 404

### ### curl + Host-based routing

# 

- They curl the ingress IP directly and get:
- **404 Not Found**
- Explanation:
- Ingress rules specify host `example.com`, so the load balancer only routes when the **Host** matches.
- Requirement:
- resolve `example.com` → ingress IP.

### Two ways to map hostname to IP (as described)

# 
1. Update local `/etc/hosts` (assigned as viewer homework)
2. Use curl host-resolution override (shown in demo)

- Abhishek demonstrates using curl resolution:
- curl to `http://example.com` while resolving it to the ingress IP (same effect as `/etc/hosts`).
- Result:
- “hello world” response confirms ingress routing works.

### Host summarizes traffic flow layering

# 
- Previously: client → **Service** → Pod
- Now: client → **Ingress** → Service → Pod
- *Ingress is an extra layer on top of the service.*

## Assignment hints + common mistakes

# 
- Host explains:
- mapping hostname to IP is like creating an “A record” (ties back to their DNS lesson).
- Common mistake called out:
- using the same IP from inside the cluster in `/etc/hosts` won’t work for laptop access.
- you should use the **node IP** (they correct themselves after initially saying “node port”).
- Additional practical note:
- allow inbound traffic on **port 80** in EC2 security groups for node access (since nodes must accept that traffic for the demo path).

## Wrap-up

# 
- They close by:
- inviting questions in comments
- pointing viewers to a Discord community for help
- thanking Abhishek and encouraging viewers to appreciate his contribution.

