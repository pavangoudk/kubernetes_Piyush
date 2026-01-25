# Kubernetes CK 2024 Day 16: Resource Requests & Limits (CPU/Memory)

## Intro and goal of the video

- Speaker introduces **video #16** in the **CK 2024** series.
- Topic: **resource requests and limits**, a concept the **scheduler** uses to schedule pods onto a node (or set of nodes).
- Plan: first a diagram-based explanation, then hands-on exercises/demos.

## Concept walkthrough (diagram examples)

### Why pods need CPU/memory

- Pods are “*an encapsulation of a container*,” and inside the container “*we run a particular process*.”
- That process “*needs a CPU and memory… to do the execution*.”

### Scheduling with available capacity (2-node example)

- Two nodes:
- **Node 1**: 4 CPUs, 4 GB memory
- **Node 2**: 4 CPUs, 4 GB memory
- Example pods each require **1 CPU + 1 memory** (as described in the diagram).
- Scheduler flow (as described):
1. Checks an available node for sufficient resources.
2. Schedules pods until a node “*reaches its limits*.”
3. If a node is full, it tries the next node.
4. Once all nodes are full, new pods fail scheduling with errors like *insufficient CPU/memory* and remain unscheduled.

### What can go wrong without requests/limits (single-node example)

- One node: 4 CPU, 4 memory (GB in earlier example framing).
- One pod starts needing ~1 CPU and 1 memory, **but nothing is specified** in the pod about what it requests or its limit.
- As load increases, the pod can try to consume the entire node:
- It can take “*entire node memory and CPU*.”
- Failure modes:
- If it tries to exceed memory: pod crashes with **OOM (out of memory)**.
- CPU overuse leads to **CPU throttling**.
- The node may invoke the “*OOM killer*” and start killing pods because the node itself lacks resources.
- Key intent: set boundaries so “*the Pod will be crashed… not the node*” when it exceeds allowed memory.

## Demo setup and metrics visibility

### Cleanup step

- Deletes existing pods (speaker removes previously created pods).

### ### Metrics Server

- Speaker applies a **metrics server** YAML.
- *“Metrics server… exposes the metrics of your nodes… CPU memory utilization… so that it can be used further with different processes like… autoscaling… VPA, HPA.”*
- After deployment:
- Metrics server runs as a pod in the **kube-system** namespace.
- `kubectl top node` shows node CPU/memory utilization (now available because metrics server is running).

## Hands-on: memory requests/limits examples

### Create a namespace

- Creates a namespace called **mem-example** (speaker’s naming).

### ### Pod YAML: memory requests/limits + stress testing (polinux/stress)

- Pod name shown: **memory-demo-ctr** (as referenced).
- Image: **polinux/stress** (used to stress-test the cluster).
- Structure emphasized:
- Resources belong under the **container**, because CPU/memory are “*resources of a container*.”
- `resources:` includes:
- `requests:` (memory, CPU possible)
- `limits:` (memory, CPU possible)

#### Memory request/limit meaning (as explained)

- Example settings shown:
- Request: **100Mi**
- Limit: **200Mi**
- Speaker explanation (close paraphrase): *request is what it will allocate to the pod at the beginning; it can go up to the limit, and beyond that the pod will be killed.*
- Notes on units:
- “*This is not megabytes, this is Mi… almost the same as MB*” (speaker’s approximation for learning).

#### Stress command/args behavior

- Uses `stress` to allocate **150 MB** (speaker describes this as overriding/forcing usage for the test).
- Result observed:
- Pod runs.
- `kubectl top pod` shows about **150Mi** memory usage.
- CPU usage is shown too; speaker notes CPU wasn’t limited/requested in this example.

## Example 2: Exceeding the limit → OOMKilled

### ### “memory-demo-2” YAML scenario

- Modified YAML:
- Request: **50Mi**
- Limit: **100Mi**
- Stress attempts **250 MB**, which is above the limit.
- Result:
- Pod status shows **OOMKilled**.
- `kubectl describe pod` indicates the reason as **OOMKilled**.
- Speaker takeaway: *we prefer to fail the pod (so it can be recreated/adjusted) rather than failing the entire node.*

## Example 3: Requesting more than the node has → Pending (unschedulable)

### ### “memory-3” YAML scenario

- Sets memory request/limit to a very large amount (speaker references **1000Gi**), larger than any node capacity in the lab.
- Stress still set to 150 MB, but scheduling is blocked by the request.
- Result:
- Pod is stuck in **Pending**.
- `kubectl describe` shows scheduling errors including:
- taint/toleration mention for control-plane node
- **insufficient memory**
- “0/3 nodes are available…” style output (speaker reads the gist)
- Speaker takeaway: if you request more than a node can provide, *the pod won’t schedule; adjust to a lower number.*

## Wrap-up: why requests & limits matter

- What speaker says you learned:
1. Pods within request/limit can run normally under load targets.
2. Pods that exceed limits get terminated with **OOM** (memory case).
3. Pods that request too much stay **Pending** due to **insufficient resources**.
- Real-world analogy from speaker:
- A pod might exceed memory due to “*a memory leak*” or “*increased traffic*”; when memory hits 100%, the pod should crash (as enforced by limits).

## Practice guidance and resources

- Speaker recommends using Kubernetes documentation during exam: “*[kubernetes.io/docs](http://kubernetes.io/docs)*” (domain and subdomains available in the exam, per speaker).
- Mentions an assignment task in the **Day 16** GitHub folder and support via Discord/YouTube.