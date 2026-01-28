# Kubernetes Service Accounts: Tokens, RBAC, and ImagePullSecrets (CK 2024, Video 25)

## Intro: what this video covers

# 

- Speaker introduces **video #25** in the CK 2024 series.
- Focus: **service accounts**
- Create service accounts
- Demo how they work
- How authentication/authorization works with service accounts
- Notes service accounts are a key Kubernetes concept and similar “service account” ideas exist in cloud platforms too.
- Mentions a hands-on task in GitHub under **day 25** folder.
- Like/comment target: **150** in 24 hours.

## Diagram recap + why service accounts matter

# 
- Speaker revisits the earlier diagram and adds **service accounts** to it.
- They recap prior flow:
- Attach a **Role / ClusterRole** to a **user/group** via bindings.
- Speaker introduces two user “types” in Kubernetes:
1. **Human users**
- *“Human user such as myself or somebody else in my team”*
- Used for admin/ops work: troubleshooting, fixing issues, etc.
2. **Service accounts**
- *“Service account is something that application uses… mostly it is used by application bots… automation user or a bot user or a nonhuman user.”*
- Examples of service-account-driven workloads/tools:
- **Jenkins** (CI/CD automation)
- **Prometheus** (monitoring)
- **Datadog** (monitoring)
- Why not use personal user identities for automation:
- Scenario: 100 team members; pipeline triggers via webhook/schedule (no human ran it)
- *It shouldn’t run “under someone’s name” if it’s automation.*
- Hence use an automation/service account.
- Key takeaway:
- Service accounts are preferred when interacting **programmatically** with the cluster/workloads.
- Process is similar to human users: assign **roles and role bindings**.

## ### kubectl: Service accounts already exist by default

# 
- Speaker states: when you create a cluster, a **default service account** is created in **all namespaces**.
- Commands shown:
- `kubectl get sa` (sa = service account)
- Without namespace flag → shows for the **default namespace**
- `kubectl get sa -A` and grep default
- Shows default service account per namespace (speaker sees 5 namespaces → 5 default SAs)

### Change in newer Kubernetes versions (tokens/secrets not auto-created)

# 
- Speaker notes (approximate): prior to **1.24 / 1.22**, Kubernetes created default SA with a default secret/token.
- Now there’s a change: secrets are **not created automatically**.
- Evidence shown:
- `kubectl describe sa default` → no tokens/secrets listed
- `kubectl get sa default -o yaml` → shows apiVersion/kind/metadata only
- Speaker emphasizes there’s no spec/token/secret created “yet.”

## ### Creating a new ServiceAccount

# 
- Speaker creates a new service account for a build server (Jenkins example):
- `kubectl create serviceaccount build-sa`
- Created in the default namespace
- `kubectl describe sa build-sa`
- Shows it exists, still no token/secret yet

## ### Creating a long-lived ServiceAccount token (Secret manifest)

# 
- Speaker shows how to create a long-lived API token for a service account using Kubernetes docs:
- Points to tasks/docs around configuring pods/containers and service account configuration.
- They create a Secret YAML (named like `secret.yaml`):
- `kind: Secret`
- `metadata.name: build-robot-secret` (example name)
- Important: annotation that links the secret to the service account:
- *“This annotation is creating the secret and attaching it to the service account… the service account was build-sa, so this annotation is important.”*
- `type: kubernetes.io/service-account-token`
- Apply and inspect:
- `kubectl apply -f secret.yaml` → secret created
- `kubectl get secrets` → shows data present
- `kubectl describe secret build-robot-secret`
- Contains:
- **token**
- **ca.crt** (certificate authority data)
- **namespace**
- Speaker calls out these three items as part of the secret.

## ### ImagePullSecrets (task + explanation)

# 
- Speaker mentions a GitHub task: **add imagePullSecret to a service account**.
- Why imagePullSecrets exist:
- If images are pulled from **public Docker Hub**, no auth needed.
- In real-world production, images are stored in **private registries**:
- Docker Enterprise Hub
- JFrog Artifactory
- Cloud registries: **ECR**, **ACR**, **GCR**
- Private registry requires authentication/authorization to pull images.
- What an imagePullSecret contains:
- A token/credentials to access/pull images (stored as a Kubernetes Secret)
- Speaker says you shouldn’t pass username/password directly in the pod spec because it’s not secure.
- How to create it (command described):
- `kubectl create secret docker-registry <name> --docker-server ... --docker-username ... --docker-password ...`
- This creates a secret storing credentials “in encrypted format.”
- How to use it in a Pod:
- In pod YAML: `imagePullSecrets: - name: <secret-name>`
- How to attach it to a ServiceAccount:
- In ServiceAccount YAML: add an `imagePullSecrets` section referencing the registry secret.
- Speaker notes it’s important for:
- Hands-on practice
- A later end-to-end project in the course
- Exam/Kubernetes fundamentals

## ServiceAccount authorization: RBAC still required

# 
- Speaker ties service account usage back to authorization:
- A service account token can be used for API calls, but the SA must have permissions.
- They test permissions for the service account:
- Run `kubectl get pods` “as” the service account (impersonation)
- Result: forbidden / cannot list pods
- `kubectl auth can-i get pods --as build-sa` → no
- Fix: create Role + RoleBinding for the service account (imperative demo)
1. Create Role:
- `kubectl create role build-role --verb=get,list,watch --resource=pods`
2. Create RoleBinding:
- `kubectl create rolebinding rb --role=build-role --user=build-sa`
3. Re-test:
- `can-i get pods --as build-sa` → yes
- `kubectl get pods --as build-sa` works
- Speaker reiterates:
- This is a **nonhuman** (automation/bot) identity.

## Using the ServiceAccount token for API calls (mentioned)

# 
- Speaker says you can make REST API calls (like earlier curl examples) by passing the **service account token**, once permissions are attached.

## ServiceAccount token mounted into Pods (where to find it)

# 
- Speaker shows that pods run with a service account (default if not specified):
- `kubectl describe pod <pod>` shows:
- **Service Account: default**
- In the volumes/mount section:
- Token is available under a path like:
- `/var/run/secrets/kubernetes.io/serviceaccount`
- Speaker clarifies:
- *“It’s not actually stored, it is mounted on the pod… as a separate volume… one of the best practice to mount the secret.”*
- They exec into a pod and inspect the directory:
- `kubectl exec -it <pod> -- sh/bash` (they “go inside the container”)
- `cd /var/run/secrets/kubernetes.io/serviceaccount`
- Directory contains 3 files:
- `token`
- `namespace`
- `ca.crt`
- `cat token` shows the default service account token.

## Cleanup: deleting the ServiceAccount

# 
- To delete:
- `kubectl delete sa build-sa`

## Wrap-up

# 
- Speaker concludes service accounts demo and invites questions via comments/Discord.
- Reminds to complete like/comment target and share the series.

# 

# 

# 

# 

#