# Kubernetes RBAC Hands-On: Roles, RoleBindings, kubeconfig, and API Calls (CK 2024, Day 23)

## Intro + where this sits in the series

# 

- Speaker introduces **Day 23** in the CK 2024 series and says they’re around halfway through.
- This video focus: **Role-Based Authorization (RBAC)** in detail, with a demo.
- A hands-on task is mentioned in the GitHub repo under the **day23** folder.

## Recap from prior videos (why RBAC is needed now)

# 
- Speaker references earlier work where they:
- Created a new user **Adam**
- Generated certificates, created a **CertificateSigningRequest (CSR)**, and approved it
- Key point: Adam is **authenticated** but still lacks access because authorization is not set yet.
- *Authentication is set; now we need to set authorization.*

## ### kubectl authorization checks (who can do what)

# 
- Speaker uses a simple target permission: allow a user to **read pods**.
- They first check access for the current user:
- Uses the “can I” authorization check to see if they can read pods.
- Result: **Yes** (for the current user).
- They confirm identity:
- Uses a “who am I” style command to show they are **kubernetes-admin**, part of **cluster-admin** group.
- They test Adam’s permissions using impersonation:
- As cluster-admin, they impersonate Adam with an “as user” option.
- Result: **No** (Adam cannot read pods yet).
- Conclusion: need to grant access via **Role** + **RoleBinding**.

## ### RBAC objects: Role (Role YAML walkthrough)

# 
- Speaker shows a sample Role manifest structure:
- Standard Kubernetes fields: API version, kind, metadata
- Instead of “spec,” it uses **rules**
- They explain API groups (brief detour because it impacts Role rules):
- *“There are two type of groups in Kubernetes: one is a core group, another is named group.”*
- Core group example: Pods use API version like “v1” with nothing after it.
- Named group example: Deployments use an API version that includes a group name (speaker contrasts “v1” vs “v1/apps” style).
- RBAC API group is also a named group (as referenced in their example).
- *If the API group is blank (empty quotes), it indicates the core API group.*
- Role rule fields highlighted:
- **apiGroups**: blank for core APIs (in this example)
- **resources**: pods
- **verbs**:
- *“Verb is the access… the type of access that we want to have.”*
- Example verbs provided: get, watch, list (read-only for pods)
- They apply the Role manifest and confirm it exists by listing roles.
- They describe the Role to show:
- Which resources and verbs are allowed
- Notes that more granular scoping is possible (specific namespace or specific pod), but they keep it broad for the demo.

## ### RoleBinding (binding the Role to a user)

# 
- Speaker notes: creating a Role alone doesn’t apply it to any user.
- They introduce RoleBinding:
- *RoleBinding binds a Role to a user.*
- RoleBinding manifest fields shown:
- API version, kind, metadata
- **subjects**:
- Example subject: kind user, name Adam, plus API group reference
- Speaker notes subjects can be user or group (and others)
- **roleRef**:
- References the Role by name (pod-reader)
- Includes the authorization API group reference
- They apply the RoleBinding, then describe it:
- Confirms RoleBinding points to pod-reader
- Confirms subject is Adam
- Notes you can scope to namespace/resource, but they leave it open.

## Verify: Adam can now read pods

# 
- Speaker re-runs the authorization check impersonating Adam.
- Result: **Yes** (Adam can now read pods because Role + RoleBinding are in place).

## ### Listing and counting Roles (exam-style tip)

# 
- Speaker shows how to list roles across all namespaces (adds an “all namespaces” option).
- They mention an exam-style question: count the number of roles.
- Approach described:
- Remove headers from the output (“no headers” option)
- Pipe into a line-count tool to count lines
- Output example they get: **12** roles

## ### kubeconfig update (logging in as the new user)

# 
- Next objective: actually “log in” and run commands as the user (not just impersonate checks).
- Speaker says this requires adding an entry to **kubeconfig**:
- Set credentials for the user (user name + client key + client certificate)
- Create a context that ties together:
- cluster name (their current kind cluster)
- user name (Adam)

## Troubleshooting: unauthorized due to expired certificate

# 
- After switching context to Adam, speaker gets errors indicating they must log in / unauthorized.
- They pause to investigate and conclude:
- The user certificate is **expired**.
- They check certificate validity using an OpenSSL certificate inspection command:
- Finds it’s not valid after a certain date.
- Root cause: earlier they created the cert with **only one day** validity.

## ### OpenSSL (attempted renewal, then decision to recreate user)

# 
- Speaker attempts to renew/sign using OpenSSL with:
- The CSR, the CA certificate, and the CA private key
- They fetch CA materials from the control plane node, but run into errors:
- CA key load issues and a mismatch error (“certificate and private key does not match”).
- They decide not to spend time on renewal:
- *They won’t cover certificate renewal now.*
- Instead, they create a **new user**: **Krishna** (following the earlier Day 21 steps).

## Create new user Krishna (repeat CSR workflow, then RBAC)

# 
- Speaker generates:
- A new private key
- A new CSR
- They create a new CSR manifest:
- Name set to Krishna
- Request field is Base64-encoded CSR content
- They increase expiration seconds so it won’t expire quickly
- They try applying the CSR manifest while still in the invalid Adam context, and it fails due to invalid credentials.
- They switch to a valid admin context and re-apply:
- CSR created, status pending
- They approve it themselves (as admin)
- They validate Krishna’s access:
- Reading pods as Krishna is initially **forbidden**
- Authorization check shows **no**
- Fix: update the RoleBinding:
- Change the bound subject from Adam to **Krishna**
- Apply the updated RoleBinding
- Re-test:
- Authorization check for Krishna becomes **yes**

## kubeconfig again (Krishna credentials + certificate extraction)

# 
- Speaker returns to kubeconfig setup for Krishna:
- Sets credentials for Krishna (client key + client certificate)
- Mentions needing an “embed cert” style option (and notes earlier a misspelling caused errors)
- They hit a “certificate file not found” issue, so they extract the issued cert:
- Export CSR output as YAML
- Take the issued certificate data
- Decode it from Base64
- Remove line breaks
- Write it to a certificate file (Krishna CRT)
- They set credentials again, create a context for Krishna, and switch to it.

## Troubleshooting: malformed certificate, then fixed

# 
- After switching to Krishna context, they hit a **malformed certificate** error.
- They inspect and compare certificate outputs and conclude the generated CRT file was corrupted.
- They regenerate the certificate file and retry:
- Now Krishna can run pod listing successfully.
- They demonstrate RBAC scoping:
- Krishna can read pods.
- Krishna cannot list deployments:
- The error indicates deployments in the apps API group are forbidden because access was only granted for pods.

## Quick recap (speaker’s condensed summary of what they did)

# 
- Because troubleshooting may have been confusing, speaker summarizes the core flow:
1. Create a new user (key + CSR)
2. Create a Kubernetes CSR object (request must be Base64-encoded)
3. Approve CSR as admin
4. Extract issued certificate data, decode it, save it as a cert file
5. Create Role + RoleBinding (authorization)
6. Add user credentials and context in kubeconfig
7. Switch context and run commands as that user
- They also show the **imperative** approach for creating:
- Role (with verbs and resources)
- RoleBinding (binding role to user)

## ### kubeconfig viewing

# 
- Speaker shows how to view kubeconfig content via a kubeconfig “view” command.
- Notes kubeconfig now includes:
- The new user Krishna
- Client certificate and key data
- A new context, and the current context is Krishna

## ### curl (calling the Kubernetes API directly)

# 
- Speaker explains kubectl calls the Kubernetes API server and returns results.
- If you need to call the API programmatically:
- Client libraries exist (Python, Node.js/JavaScript, etc.)
- You can also use curl
- They demonstrate a curl call to the API server endpoint for listing pods:
- Uses the control plane address and port
- Calls an API path for namespaces and pods
- Passes:
- client key
- CA certificate
- client certificate
- plus an option to allow the connection as shown in their example
- Response is returned in **JSON** (they note it’s API server output).

## Wrap-up + what’s next in “security” topics

# 
- Speaker says this certificates + CSR + authorization topic spanned three videos to cover concepts and hands-on.
- Remaining security topics they list:
1. **ClusterRole and ClusterRoleBinding** (next video)
2. **Service accounts**
3. **Network policies**
- They remind there’s a hands-on task in the day23 GitHub folder and support via comments/Discord/community.

# 

# 

#