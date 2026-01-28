# Kubernetes Authorization + kubeconfig Basics (CK 2024, Video 22)

## Intro + where this fits

# 

- Speaker introduces **video #22** in the **CK 2024** series.
- Topic: **authorization**, **kubeconfig**, and related concepts.
- Note: this video depends on prior videos, so watch in sequence.

## How `kubectl` commands get authorized (problem statement)

# 
- Example command: `kubectl get pods`
- Speaker’s question: you’re already “logged in,” but *how does the API server authorize what you can do* (get/create/delete pods, etc.)?
- Speaker explains that along with the visible command, **other values are passed in the background**, such as:
- API server (“server details”)
- Client certificate
- CA certificate
- Client key
- “all those details”
- You don’t see them because they’re passed internally.

## ### kubeconfig

# 
- The background details are provided via a file called **kubeconfig**.
- You *can* specify it explicitly:
- `kubectl --kubeconfig <path> ...`
- Useful when you have **multiple kubeconfig files** and need to choose one.
- If you don’t specify it, `kubectl` uses the **default kubeconfig** location:
- `$HOME/.kube/config`
- Speaker says they’ll show content later, but first shows what a “sample kubeconfig” looks like.

### What a kubeconfig file contains (structure)

# 
- Has standard Kubernetes manifest fields:
- `apiVersion`
- `kind`
- Includes arrays/sections:
- **clusters**: you can list multiple clusters (speaker mentions they have 3–4 clusters locally)
- **users**: users with permissions to take actions on the server
- **contexts**
- *“Context are nothing but the combination of user and cluster.”*
- Examples described:
- Create a context like **dev-frontend** for developers to access the dev cluster
- Create a context like **experiment-test** where a user “experimentor” can access only the test cluster
- Context entries reference:
- `cluster`
- `user`
- You can switch/set context using commands they’ve been using earlier in the series (referenced but not re-taught here).

### Viewing the default kubeconfig content

# 
- Speaker opens `$HOME/.kube/config` (using `vi`).
- Notes it’s hard to render due to lots of data, but highlights:
- **contexts** array: contains `cluster`, `user`, and `name`
- **users** section: includes
- `client-certificate-data`
- `client-key-data`
- Speaker points out you can provide cert/key as file paths or as inline data; here it’s inline “data.”

## ### Kubernetes Documentation (kubeconfig reference)

# 
- Speaker navigates docs and searches “kubeconfig.”
- Opens doc: **Configure access to multiple clusters**.
- Notes:
- This doc example shows (under cluster) the **certificate authority** (CA) file path.
- Under user, it should include **client certificate** and **client key**.
- Connects this back to what they saw in the default kubeconfig:
- Inline `client-certificate-data` corresponds to the certificate content.

### Environment variable for kubeconfig (mentioned)

# 
- Speaker says you can export kubeconfig path(s) via an **environment variable** if kubeconfig files are in multiple locations (no exact variable name stated in transcript, just that “this is the environment variable that you use”).

## Authentication vs Authorization (concept separation)

# 
- Speaker references a diagram-style explanation:
- **Authentication** (already covered in previous videos): answers *“who you are”*.
- Mentions earlier topics covered: symmetric encryption, asymmetric encryption (public/private key), certificates, implementation.
- **Authorization**: checks *whether the authenticated user has permission to do a task*.
- Example: does user have permission to run `get pods`?
- Outcome: allow or deny the request.

## ### Authorization modes in Kubernetes

# 
Speaker lists different authorization modes and describes each:

1. **ABAC (Attribute-Based Access Control)**

- *“Attribute based authentication”* (speaker’s phrasing)
- Process described:
- Associate permissions to a user
- Put permissions in a **policy file**
- Add policy file to **API server config**
- **Restart API server**
- Downsides stated:
- Tedious and hard to maintain
- Requires restarting API server for changes
- Not widely used
2. **RBAC (Role-Based Access Control)**

- Process described:
- Create a **role**
- Assign permissions (e.g., get/delete/list pods) scoped to namespace/resource/all resources
- Bind role to a **user** or **group of users**
- Benefits stated:
- If role changes, update the role; users aren’t impacted directly
- No need to restart API server
- Components mentioned:
- roles and role bindings
3. **Node authorization**

- Used by nodes/components to interact with each other.
- Example: when API server interacts with kubelet, kubelet needs to know API server is allowed.
4. **Webhook authorization**

- Uses third-party systems (example: **OPA—Open Policy Agent**) to decide access.
- Request goes to the agent, which processes and returns allow/deny.

- Speaker previews: next video will go deep into **RBAC** and implementation.

## ### kind (Kubernetes in Docker) + Docker exec into control plane

# 
- Speaker wants to show API server settings.
- Notes they’re running Kubernetes via **kind** (Kubernetes in Docker), so:
- Can’t SSH to nodes
- Instead uses `docker exec` into the container (node is a container in this setup)
- Commands/actions described:
- `docker ps | grep control` to find control plane containers
- Identifies the relevant one for “cluster2” (their current cluster)
- `docker exec -it <container> bash` to enter as root

## ### Static Pod Manifests location

# 
- Speaker reminds: static pod manifests (API server, controller manager, etcd) are in:
- `/etc/kubernetes/manifests`
- Lists directory and finds `kube-apiserver.yaml`.
- Notes tooling limitation: `vi`/`less` not available inside the container (as observed).

## API server authorization flags (from kube-apiserver startup args)

# 
- Speaker points out the API server is started with many command arguments, including:
- `--authorization-mode=Node,RBAC` (as shown)
- Execution order detail:
- Authorization modes run in the sequence specified.
- Example flow:
- A user runs `get pods`
- Node authorization checked first; not applicable for user → skipped
- Then RBAC evaluated (roles/role bindings) → allow/deny

## API server authorization defaults (doc reference)

# 
- Speaker goes to API server documentation and searches “allow.”
- Key point stated:
- If authorization config is not used, it defaults to **AlwaysAllow**.
- *AlwaysAllow* means any authenticated user can do anything.
- Also mentions **AlwaysDeny** exists (denies everything).
- Best-practice statement from speaker:
- Use one or more modes; typically **Node** plus **RBAC** and **webhook** in many places.

## ABAC authorization policy file (where it fits)

# 
- Speaker ties back to ABAC:
- The policy file is referenced in the API server YAML (and requires restart to take effect).
- Mentions there are many other API server flags/fields.

## kubeconfig vs API server configuration (where each reads settings from)

# 
- Client-side (`kubectl`):
- Uses kubeconfig by default to find certs/keys/server endpoints.
- Control plane components (API server):
- Take their authentication/authorization and cert references from their manifest YAML (e.g., `kube-apiserver.yaml`).
- Speaker emphasis: these files contain the keys, certificates, and authorization parameters needed.

## ### PKI directory for certificates on control plane

# 
- Speaker notes default certificate/key location:
- `/etc/kubernetes/pki`
- Shows examples of files present:
- Root CA cert and key
- API server ↔ kubelet client key/cert
- API server cert/key
- Explains why multiple API server-related certs exist:
- Refers back to prior diagram:
- API server is **server** for kubectl/scheduler/controller-manager/kube-proxy
- API server is **client** when talking to **etcd** and **kubelet**
- You can reuse or create separate key pairs; here separate files exist for:
- API server ↔ kubelet
- API server ↔ etcd
- Practical exam/troubleshooting note:
- If cert file names are wrong or there are cert issues, this is where you check.

## Wrap-up + next video

# 
- Speaker concludes: covered kubeconfig + authorization overview and where key files live.
- Next video: **RBAC (role-based authorization)** with hands-on work.
- Mentions a task in GitHub under **day22** folder and support via YouTube comments/Discord.

# 

#