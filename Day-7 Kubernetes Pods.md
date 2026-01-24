# Pods + YAML Fundamentals in Kubernetes (CK 2024 — Video 7)

## 1) Intro: what this video covers and why it matters

- Speaker introduces **video #7** in the **CK 2024** series.
- Focus areas:
- **Pods**
- **YAML fundamentals**
- Why YAML now:
- YAML is the language used “throughout this series” and is “heavily used in Kubernetes.”
- Goal is to cover basic YAML fundamentals, syntax awareness, and best practices.
- From this video onward, the series moves into “complete hands-on.”

**Section summary:** This is the transition into hands-on Kubernetes: creating pods and learning YAML as the primary config format.

## 2) Reminder of the Kubernetes interaction flow (user → API server)

- Speaker references the earlier architecture diagram:
- A **user** interacts with the **API server** (on the control plane/master node).
- The user interacts using a client utility:
- Usually **kubectl** for this series
- In managed services, it could be the cloud console or other utilities
- User actions include:
- Provision, update, delete resources
- Get details from the cluster

**Section summary:** kubectl is the main way the user sends requests to the API server to manage cluster resources.

## 3) Pods and “running workload” goal

- Speaker emphasizes the purpose of Kubernetes:
- Running workloads (containers) as pods in the cluster.
- Example workload in this video:
- An **nginx** pod running inside the Kubernetes cluster.

**Section summary:** The hands-on objective is to run an nginx workload as a pod.

## 4) Two ways to create resources: imperative vs declarative

### Technique: Imperative approach

- *“In imperative… you run simple commands… you are instructing… kubectl… do this, run that, get these details.”*
- Example described:
- `kubectl run nginx` (speaker uses this as the style of command)

### Technique: Declarative approach

- You create a configuration file (desired state) and apply it.
- File formats:

- YAML (heavily used)
- JSON (supported, but speaker “hardly know anyone who uses JSON”)
- Declarative workflow:

- Create config file defining desired state (API version, name, image, ports, etc.)
- Run `kubectl create` or `kubectl apply`
- Speaker’s usage guidance:

- Imperative commands are often used for troubleshooting or quick/local actions.
- Declarative is used for production deployments, CI/CD, GitOps, etc.
- Speaker stresses:

- Both are equally important; you should know both.

**Section summary:** Kubernetes work is done either by imperative commands (quick instructions) or declarative YAML/JSON (desired-state files), and both matter operationally and for production workflows.

## 5) Hands-on: create an nginx pod (imperative)

### Tools/Commands: kubectl run, kubectl get pods

- Speaker works in VS Code with a README for notes, and uses a terminal.
- Imperative create:
- `kubectl run <pod-name> --image=nginx`
- Speaker uses a clearer name like “nginx-pod.”
- Verify:
- `kubectl get pods`
- Pod lifecycle note:
- It may show **ContainerCreating** before **Running**.
- Readiness column meaning:
- Example shown: `READY 1/1`
- Explained as: the pod has **1 container**, and **1 is running/ready**.
- If a pod had multiple containers, it could show `2/2`, `1/2`, etc.

**Section summary:** The speaker creates an nginx pod with `kubectl run` and validates it with `kubectl get pods`, explaining readiness as “containers ready / total containers.”

## 6) YAML basics (general YAML syntax, not Kubernetes-specific yet)

### Concept Defined: YAML comments

- *“You can add comments in yaml with the hash sign.”*
- A line starting with `#` is treated as a comment and not executed/processed as data.

### YAML data types (as listed)

- YAML supports:
- List
- Dictionary
- String
- Integer
- (and more)

### Dictionary/object via indentation

- Speaker example uses an “employee” dictionary.
- Key point:
- YAML structure relies on **spaces/indentation**.
- Double quotes around strings are optional.
- Indentation best practice:
- Tabs are “not recommended” because they make YAML “clumsy.”
- Speaker uses **two spaces**.
- Indentation mistake consequence:
- If you mis-indent, a field may become a separate top-level key instead of nested under the parent.

### Lists with dash (`-`)

- Lists are created with `-` items at the correct indentation level.
- Example:
- A list of employee entries with repeated keys (name/age/address).
- Nested structures are supported:
- Example: making “address” itself a nested list (old address/new address).

**Section summary:** YAML is structured by indentation; dictionaries nest via spaces, lists use dashes, and consistent spacing is critical to avoid creating unintended keys.

## 7) Kubernetes Pod YAML: structure and key fields

### Kubernetes YAML top-level fields

- Speaker states a basic Kubernetes manifest has four top-level fields:
1. `apiVersion`
2. `kind`
3. `metadata`
4. `spec`

### Important rule: case sensitivity

- Speaker intentionally makes a mistake to highlight:
- YAML keys for Kubernetes objects are **case-sensitive**.
- Example corrections:
- `apiVersion` (not “APIversion”)
- `kind`, `metadata`, `spec` in correct casing
- `Pod` kind uses capital P (as written by speaker)

### Tool/Command: `kubectl explain`

- Speaker shows how to confirm supported versions/fields:
- `kubectl explain pod`
- Point made:
- Objects can have versions like alpha/beta/v1; use supported versions.

### Sample Pod YAML fields (as built by the speaker)

- `apiVersion: v1`
- `kind: Pod`
- `metadata` includes:
- `name: nginx-pod`
- `labels:` with examples:
- `environment: demo`
- `type: front-end`
- Labels note:
- Inside `labels`, you can use **any key/value pairs** you want.
- `spec` includes:
- `containers:` (a list)
- `name: nginx-container`
- `image: nginx`
- `ports:` (a list)
- `containerPort: 80`

**Section summary:** A basic Pod manifest follows apiVersion/kind/metadata/spec, with containers and ports defined in spec; labels are flexible key-value metadata.

## 8) Create the pod from YAML (declarative), and delete/recreate

### Tools/Commands: kubectl create/apply/delete

- Speaker renames the file to something like `pod.yaml`.
- Runs:
- `kubectl create -f pod.yaml`
- Notes: `apply` can create or update; create is fine for creation.
- If pod already exists:
- They delete it first:
- `kubectl delete pod <pod-name>`
- Then rerun create to recreate from YAML.
- Verifies with:
- `kubectl get pods` (pod is running)

**Section summary:** Declarative creation uses `kubectl create -f`; if the pod already exists, delete it first or change the name.

## 9) Introduce troubleshooting: image pull errors + fixing in two ways

### Step: intentionally break the image name

- Speaker edits the YAML and misspells the image name.
- Applies the change:
- `kubectl apply -f pod.yaml`
- Warning appears:
- Mentions missing “last applied configuration” annotation; speaker says it can be ignored.
- Result:
- `kubectl get pods` shows `READY 0/1`
- Status: **ImagePullBackOff**
- Speaker definition/meaning:
- *“Image pull back off… it is facing issues while pulling the nginx image from the dockerhub registry.”*

### Tool/Command: `kubectl describe pod`

- To troubleshoot:
- `kubectl describe pod <pod-name>`
- Speaker points to “events” showing the error:
- Pull denied / repository does not exist / may require authorization / insufficient authorization
- Interpretation given:
- Either image/tag is wrong, or you don’t have permission (but nginx is public so likely a typo).

### Two ways to fix

1. **Update YAML + apply again** (already demonstrated earlier)
2. **Edit the live object directly**
- `kubectl edit pod <pod-name>`
- Opens a vi-like editor
- Fix the image field
- Save with `:wq!`
- Output: “pod edited”
- Speaker notes:
- No need to apply again because change is made directly to the running pod.

- After fix:
- `kubectl get pods` shows Running again
- `kubectl describe pod` no longer shows new errors (older event remains, but no current errors)

**Section summary:** ImagePullBackOff is diagnosed via `kubectl describe pod` events, and fixed either by re-applying corrected YAML or by editing the live pod with `kubectl edit`.

## 10) Exec into the pod/container (interactive shell)

### Tool/Command: `kubectl exec`

- Equivalent concept to Docker exec (as referenced by speaker).
- Command pattern shown:
- `kubectl exec -it <pod-name> -- sh`
- Inside the container:
- Uses `pwd` (shows root directory)
- Can inspect logs/files as needed
- Exits the shell with `exit`.

**Section summary:** `kubectl exec -it ... -- sh` provides an interactive shell inside the running container for investigation.

## 11) Generate YAML from an imperative command (dry-run output)

### Technique: dry-run to produce manifests

- Speaker shows a faster way to create manifests:
- Write the imperative command, but don’t actually create the resource; output the manifest instead.

### Tools/Commands: `--dry-run=client`, `-o yaml`, redirect to file

- Example pattern shown:
- `kubectl run nginx --image=nginx --dry-run=client -o yaml`
- This prints the YAML manifest for the would-be pod.
- Redirect to a file:
- Use `>` to write to a new file:
- `... > pod-new.yml`
- Speaker then edits the generated YAML:
- Removes fields like creationTimestamp
- Adjusts labels (e.g., env demo)
- Keeps name/image as desired

### JSON option (also supported)

- Same approach, but output JSON:
- `-o json`
- Can redirect to `pod-new.json`
- Speaker notes:
- JSON will still contain the same overall structure: apiVersion/kind/metadata/spec.

**Section summary:** `--dry-run=client -o yaml` is a practical way to generate correct manifests quickly, then you edit and apply them; JSON output is also available.

## 12) More pod inspection commands: labels and wide output

### Tool/Command: `kubectl describe pod`

- Used to see details like:
- Conditions (ready, scheduled, containers ready)
- Ports (e.g., 80)
- Image and image ID
- Namespace
- Node the pod is running on
- Labels

### Tool/Command: `kubectl get ... --show-labels`

- Shows labels inline:
- Example labels shown: `env=demo`, `type=front-end`

### Tool/Command: `kubectl get pods -o wide`

- Adds extended output:
- Pod IP
- Node name (where the pod is running)
- Speaker contrasts:
- `kubectl get pods` (basic columns)
- `kubectl get pods -o wide` (adds IP and Node)

### Tool/Command: `kubectl get nodes -o wide`

- Similarly extends node information:
- OS image
- Kernel version
- Container runtime
- External IP (sometimes helpful for troubleshooting)

**Section summary:** Use describe for deep detail, show-labels to see grouping metadata, and `-o wide` for quick visibility into pod IPs and node placement (and node OS/runtime details).

## 13) Assignments (three tasks)

1. Create a pod using an **imperative command** with **nginx** image.
2. Generate YAML from task 1 and **update pod name**, then use it to create a **new pod**.
3. Fix a provided/broken YAML and apply changes, following the troubleshooting steps shown.

**Section summary:** The viewer is asked to practice imperative creation, manifest generation, and YAML troubleshooting workflow.

## 14) Closing and next topic

- Speaker asks viewers to support with:
- **150 comments** and **250 likes** in 24 hours
- Next video will cover:
- **Deployments** and **ReplicaSets**
- More hands-on and concepts
