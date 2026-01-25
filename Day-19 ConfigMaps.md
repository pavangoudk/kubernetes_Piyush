# Kubernetes CK 2024 Day 19: ConfigMaps (and Intro to Secrets)

## Intro: what this video covers + practice

# 

- Speaker introduces **video #19** in the CK 2024 series.
- Topic: **ConfigMap and Secret** (speaker notes it should’ve been covered earlier, but was missed).
- Plan: concept + demo; hands-on task in the **GitHub repository (Day 19 folder)**.
- Engagement target: **150 comments + 150 likes** in 24 hours.

## Why ConfigMaps: problem with inline env vars in pod YAML

# 
- Speaker references an earlier video (Day 11) covering:
- multi-container concepts
- commands/arguments
- environment variables
- Prior approach: define environment variables directly in the Pod YAML as **key/value pairs**.
- Why that becomes a bad practice:
- As env vars “grow,” and especially when the same env var is needed across “this pod and 10 other pods,” repeating the values in many YAMLs is not ideal.
- Goal: keep a consistent value for identical environment variables across workloads.

## What a ConfigMap is (as used here)

# 
- Best-practice approach described: *“we take this outside and we create a config map object and we actually place this key value pair inside that config map and then we inject that value inside this pod.”*

## Demo setup: baseline pod with inline env var

### ### BusyBox (demo container image)

# 

- Speaker edits a pod YAML (copies into Day 19 context, removes initContainers for simplicity).
- Pod uses **busybox**.
- Pod includes:
- an environment variable (`name`, `value`)
- a command that prints “app is running”
- `sleep 3600` to keep it running

### Validate env var inside the pod

# 
- Applies the pod YAML.
- Execs into the pod and verifies the env var via `echo $FIRSTNAME` (speaker’s variable name is “first name/first_name” conceptually; he echoes it and sees the value printed).

## Creating a ConfigMap (imperative)

### ### kubectl create configmap (from-literal)

# 

- Speaker removes the env var from the pod YAML and creates a ConfigMap instead.
- Uses imperative creation:
- `kubectl create cm <name> --from-literal=key=value`
- Example shown:
- ConfigMap name: `app-cm`
- Keys/values:
- `first name = p` (speaker’s value is “p/push” contextually)
- `last name = ...` (adds another key/value)
- Notes: for multiple entries, you can continue with line breaks using backslash and repeat `--from-literal`.

### Inspect the ConfigMap

# 
- `kubectl get cm`
- `kubectl describe cm/app-cm`
- Speaker calls out fields (name/namespace/labels/annotations, and **data**).
- Data is displayed as `key: value`.

## Injecting ConfigMap values into a pod (single key)

### ### env + valueFrom + configMapKeyRef

# 

- Speaker looks up syntax and uses the pattern:
- `env:`
- `name: <ENV_VAR_NAME_IN_CONTAINER>`
- `valueFrom:`
- `configMapKeyRef:`
- `name: <CONFIGMAP_NAME>`
- `key: <KEY_IN_CONFIGMAP>`
- Clarification he makes while editing:
- The container env var name can be different (he uses uppercase in the container).
- The ConfigMap key reference points to the **lowercase key** stored in the ConfigMap data.

### Apply changes (pod recreation)

# 
- Applies updated pod YAML; if updates are messy, he deletes and recreates the pod.
- Execs into the pod again and `echo`s the env var; it prints the expected value.

### Confirm via describe

# 
- `kubectl describe pod ...`
- In the Environment section it shows:
- the env var is “set to the key … of the ConfigMap app-cm” (speaker reads this as confirmation of the reference mapping).

## Creating a ConfigMap from a file (imperative alternative)

# 
- Speaker explains: if there are “hundreds of key value pairs,” `--from-literal` line-by-line isn’t practical.
- Alternative approach:
- Put key/value pairs in a file (e.g., `app.config`)
- Create ConfigMap with **`--from-file`**
- Injection into pod works the same way afterward.

## Exam-style note: updating ConfigMaps and seeing changes

# 
- Speaker note: if asked to update/change ConfigMap values, you won’t see changes in running pods unless you:
- force recreate (mentions `--force`), **or**
- delete and recreate the pod (as he did)

## Creating a ConfigMap (declarative)

### ### Dry-run to generate YAML

# 

- Shows how to generate a declarative YAML from the imperative command:
- use `--dry-run=client -o yaml > cm.yaml`
- Opens the generated file and points out structure:
- `apiVersion`, `kind`, `metadata` (common for all Kubernetes objects)
- Instead of `spec`, ConfigMap uses **`data`**
- `data` contains the key/value pairs
- Apply the YAML to create the ConfigMap.

## Injecting an entire ConfigMap at once (efficient method)

### ### envFrom + configMapRef

# 

- Speaker finds the pattern for loading **all** ConfigMap entries into the container:
- `envFrom:`
- `configMapRef:`
- `name: <CONFIGMAP_NAME>`
- He calls this the efficient approach when you have many environment variables:
- “just three lines” to load everything, instead of repeating `valueFrom` for each variable.

## Mounting a ConfigMap as a volume (mentioned, deferred)

### ### ConfigMap as volume mount (introduced, not fully taught)

# 

- Speaker mentions another method:
- Mount ConfigMap as a volume via `volumeMounts` + `volumes` and `configMap:`
- He explicitly postpones details because volumes/storage haven’t been covered yet:
- doesn’t want to confuse; will revisit during the storage section.
- Key distinction he states:
- mounted vs injected: it’s “accessible” via mount path rather than injected as environment variables.

## Wrap-up and why this matters going forward

# 
- Speaker says he covered this because ConfigMap/Secret references will be used in later videos.
- Encourages:
- completing the Day 19 GitHub task
- reaching out via YouTube comments or Discord if stuck (but try yourself first)
- Thanks viewers; asks to complete like/comment targets; closes the video.

# 

#