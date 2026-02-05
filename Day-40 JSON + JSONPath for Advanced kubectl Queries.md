# CKA 2024 Video 40: JSON + JSONPath for Advanced kubectl Queries (Plus Custom Columns & Sorting)

## Intro: what this video covers

# - - - Host (PE) introduces **Video #40** in the CKA 2024 series.
- Topics:
- **JSON** “from the beginner’s perspective”
- **JSONPath** as a query language to filter/query JSON for meaningful results
- using JSONPath for **advanced `kubectl` commands** against cluster data
- Engagement target: **150 likes** and **100 comments** in 24 hours.

## What happens behind `kubectl get nodes` (human-readable vs API response)

### ### kubectl

# - - - Host runs: `kubectl get nodes`
- Explains the flow (client/server concept):
- user runs a `kubectl` command
- `kubectl` interacts with the **API server** (sends a request like GET nodes)
- server responds with a **JSON payload**
- `kubectl` converts that JSON into a “human readable format” (tabular output)

### JSON output vs YAML output from kubectl

# - - - Shows how to view the raw response:
- `kubectl get nodes -o json`
- includes many extra details (metadata, image details, kernel/runtime versions, etc.)
- Also possible:
- `kubectl get nodes -o yaml`
- Motivation:
- sometimes you want specific extra fields (kernel version, container runtime version, etc.), not the default table

## Creating side-by-side JSON vs YAML (pod manifest example)

### ### kubectl run (dry-run to generate manifests)

# - - - Creates a pod manifest in both formats using dry-run and redirection:
1. JSON file:
- `kubectl run nginx --image=nginx --dry-run=client -o json > pod.json`
2. YAML file:
- `kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml`
- Displays both to compare.

### JSON vs YAML structural difference (how host explains it)

# - - - YAML: fields are separated with spaces/line breaks (indentation).
- JSON: organized with braces/brackets.
- Host defines key JSON building blocks (very close paraphrase):
- *“everything that is there in the curly braces is a dictionary… dictionary is a data type which has key value pair”*
- *“everything inside square bracket is a list”* (items separated by commas)

## JSONPath basics: navigating dictionaries/lists

# - - - Host describes accessing nested fields via a “path”:
- root → nested keys (e.g., metadata → labels)
- Mentions common JSONPath idea:
- start at root (often `$`)
- Kubernetes nuance:
- in `kubectl -o jsonpath=...`, host says `kubectl` “adds the `$` by itself” (so he starts with `.`)

## First JSONPath attempt (why it failed, then corrected)

### ### kubectl -o jsonpath (pod label query)

# - - - Applies the pod YAML first:
- `kubectl apply -f pod.yaml`
- Attempts to query labels:
- `kubectl get pods -o jsonpath='{...}'`
- Initially gets no output and checks docs/cheatsheet.

### Realization: `kubectl get pods -o json` returns a LIST with `.items`

# - - - Host notices the live `kubectl get pods -o json` output structure differs from the earlier dry-run JSON:
- response includes `.items` (a list/array)
- so paths must include `items[0]...`

### Corrected query: get the `run` label value

# - - - Uses JSONPath with items indexing:
- `.items[0].metadata.labels.run`
- Gets output:
- `nginx`
- Formatting trick:
- adds newline using `"\n"` so output prints neatly on a new line

## More JSONPath examples: image and volumeMounts

### ### JSONPath: container image

# - - - Host navigates:
- `.items[0].spec.containers[0].image`
- Returns the image name (nginx).

### ### JSONPath: volume mount path (nested list inside containers)

# - - - Explains containers is a list, and volumeMounts is also a list.
- Uses a path like:
- `.items[0].spec.containers[0].volumeMounts[0].mountPath`
- After fixing spelling/indexing, it prints the mount path successfully.

## JSONPath with nodes: extracting node names (and formatting)

### ### kubectl get nodes -o json

# - - - Host opens the large JSON and identifies:
- `.items[*].metadata.name` contains node names

### JSONPath wildcard

# - - - Uses wildcard `*` to iterate across items:
- `.items[*].metadata.name`
- Output: `master worker1 worker2` (as shown)

### Adds newline formatting

# - - - Adds `"\n"` so results print on separate lines (demonstrates formatting approach again).

## Custom output formatting: Custom Columns (custom-columns)

### ### kubectl -o custom-columns

# - - - Host introduces a different output mode:
- `-o custom-columns=...`
- Goal: print only selected fields as columns (e.g., node name and IP).
- Initially makes a few syntax mistakes, then gets it working.

### Key behavior he discovers

# - - - With `custom-columns`, he notes:
- he **did not include** `.items` in the expression when it started working.
- Example that works for node names:
- `kubectl get nodes -o custom-columns=NODE:.metadata.name`

### Adding another column (IP)

# - - - Adds another column labeled “IP” and points to node address data under status/addresses (array).
- He demonstrates an expression that prints more than desired (includes address types too), and says:
- the better approach is to filter to only **InternalIP**.

### Filter hint (left as practice)

# - - - Host describes the JSONPath filter concept (not fully executed):
- use `?()` and `@` to filter where address type is InternalIP
- *“only that IP address will be returned”*
- He leaves this as practice for viewers.

## Reminder: practice matters (host’s reflection)

# - - - Host says he’s doing JSONPath after a long time and had to refer back to docs and JSON structure.
- Encourages hands-on until you can reliably:
- navigate dictionaries/lists
- retrieve requested fields (exam/interview-style queries)

## Extra topic: sorting kubectl output (`--sort-by`)

### ### kubectl --sort-by

# - - - Host adds one more exam-relevant feature:
- sort output by a field path:
- `kubectl get <resource> --sort-by=<jsonpath-like-field>`
- Notes similar to custom-columns, he doesn’t use `.items` in `--sort-by`.

### Sorting pods example

# - - - Creates additional pods (nginx, redis) so sorting is visible.
- Sorts by name:
- `kubectl get pods --sort-by=.metadata.name`
- Sorts by creation time:
- initially guesses a timestamp field, then checks JSON and finds correct field:
- `.metadata.creationTimestamp`
- `kubectl get pods --sort-by=.metadata.creationTimestamp`
- Observes sorted output by age/creation order (nginx older, redis newer, etc.).

## Closing

# - - - Host concludes the video:
- JSON + JSONPath fundamentals
- advanced `kubectl` querying patterns
- custom-columns and sort-by are important for exam/admin work
- Asks viewers to complete the like/comment target and returns in the next video.

# 

# 

#