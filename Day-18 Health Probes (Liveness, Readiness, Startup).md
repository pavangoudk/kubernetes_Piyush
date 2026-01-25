# Kubernetes CK 2024 Day 18: Health Probes (Liveness, Readiness, Startup)

## Intro: what’s covered + practice

# 

- Speaker introduces **video #18** in the CK 2024 series.
- Topic: **health probes** in Kubernetes: **liveness probe**, **readiness probe**, **startup probe**, and more.
- Format: concept overview + demos + hands-on practice.
- Assignment: task in the **GitHub repo (Day 18 folder)**.

## What “probe” means (before Kubernetes probes)

# 
- Speaker frames “probe” as not a buzzword, but something to understand.
- Definition (close paraphrase): *A probe is “something… monitoring your system and taking some necessary actions” to recover health, grab status, or ensure everything is working fine.*
- Probe = “inspecting / interrogating / monitoring” to take necessary actions.

## What health probes do in Kubernetes (big picture)

# 
- There are **three types**: **startup**, **readiness**, **liveness**.
- Purpose (speaker’s framing): *They “make sure that your application is healthy,” and “self-healing,” meaning when it fails it should restart automatically so users aren’t impacted.*

## Liveness probe (concept)

# 
- *Liveness probe “monitors your application”… continuously after a certain period (e.g., 10–15 seconds).*
- Example failures: intermittent network error or application error recoverable by restart.
- Behavior described:
- Restarts the app if it fails.
- Keeps restarting; if restarts don’t fix it, it “throws the error” so you can investigate.
- You get alerts that liveness probes are failing (something wrong with app or health checks).

## Readiness probe (concept + load balancer example)

# 
- *Readiness probe “ensures that your application is ready before it starts serving the traffic to the user.”*
- Example scenario:
- App runs in **three pods** behind a **load balancer**.
- Users hit the load balancer, which routes to pods.
- If the app takes **20–30 seconds** to fully boot, users might see **502 / 5xx** errors during startup.
- Readiness prevents this by ensuring:
- The pod is only added to the load balancer / allowed traffic **after readiness passes**.
- It “will not [probe]/add” until fully started (speaker’s intent: it won’t be considered ready until it passes).

## Startup probe (concept)

# 
- Used for “slow or legacy container applications” that take a long time to start.
- *Startup probe makes sure the app can wait (e.g., “5 minute”) before other probes become active.*
- Order described:
1. **Startup probe** is active first.
2. Only after it succeeds do **readiness** and **liveness** become active.
- Best-practice note from speaker:
- In production, commonly use **liveness + readiness** for all apps.

## Three probe check mechanisms (applies to all probe types)

# 
- Speaker adds: there are **3 health probe types**, and **each** can use **one of three check methods**:
1. **HTTP**: send an HTTP request to an endpoint every N seconds; success response = pass.
2. **TCP**: try to open a port; based on response, pass/fail.
3. **Command**: execute a command; success exit = pass.

## Demo 1: Command-based liveness probe

### ### BusyBox (command probe example)

# 

- Pod uses **busybox** (speaker notes it has binaries/executables; good for running arbitrary/inline commands rather than a long-running OS).
- Container command sequence explained:
1. `touch /tmp/healthy` (creates empty file)
2. `sleep 30`
3. `rm /tmp/healthy` (removes file)
4. `sleep 600` (wait ~10 minutes)

### Liveness probe configuration (exec)

# 
- Probe is under the **container** (speaker emphasizes indentation/parent-child):
- `livenessProbe:`
- `exec:` with a command array
- command shown: `cat /tmp/healthy`
- Two timing fields explained:
- *`initialDelaySeconds` = wait before the first health check (example: 5 seconds).*
- *`periodSeconds` = interval between checks (example: every 5 seconds).*
- What happens (speaker walkthrough):
- First 30 seconds: file exists → `cat` succeeds → probe passes.
- After 30 seconds: file removed → `cat` fails → non-zero exit code → probe fails → container restarted.
- Speaker notes Unix exit code idea: *0 = success; non-zero (1/127/etc.) = error.*

### Observations shown

# 
- Applies YAML (in Day 18 folder) and watches pod state.
- After ~30 seconds, restarts begin.
- `kubectl describe pod` shows:
- “Unhealthy”
- liveness probe can’t open file (file missing)
- kubelet kills container and restarts it
- Speaker notes restarts continue during the long sleep period.

## Demo 2: HTTP-based liveness probe

### ### HTTP GET liveness probe

# 

- Uses `httpGet:` instead of `exec:`.
- Checks:
- path like `/healthz` (speaker references a health path)
- port **8080**
- `initialDelaySeconds: 3`, `periodSeconds: 3`
- Speaker applies the YAML and checks behavior.

### What the speaker observes (and troubleshoots)

# 
- Initially, describe output shows:
- it was unhealthy at first (took time) and saw HTTP **500**.
- container restarts.
- Speaker concludes the health endpoint/file isn’t present (and/or endpoint not responding):
- Attempts `kubectl exec` into container and checks a health path/file; “no file present.”
- While trying to fix, container restarts mid-session.
- Pod eventually goes to **CrashLoopBackOff** due to continuous probe failures.
- Speaker notes: even after creating a file, it may still fail because:
- it’s trying to open an HTTP connection to the endpoint
- the port might not be open / endpoint not available.

## Demo 3: TCP-based liveness probe

### ### TCP socket liveness probe

# 

- TCP probe checks whether a port is open (example port **8080**).
- Timing used:
- `initialDelaySeconds: 15` (wait before first probe)
- `periodSeconds: 10` (speaker mentions reducing to 5 in discussion)
- First run: pod doesn’t restart; describe shows no issues (port open, probe passes).

### Forcing a failing TCP probe

# 
- Speaker tries changing probe port to **3000** to make it fail.
- Notes you can’t update certain pod fields directly (“invalid pod updates…”), so they do a **force apply** to recreate the pod.
- Clarification from speaker:
- Earlier they were checking the wrong port relative to what the app listened on; then they “swapped it.”
- Now app is on **8080**, but TCP liveness check targets **3000**.
- Result:
- Pod restarts due to liveness failures.
- `kubectl describe pod` shows:
- “Unhealthy”
- liveness probe failed: **connection refused**
- container will be restarted

## Inspecting defaults and extra probe fields

### ### Viewing pod YAML to see defaults

# 

- Speaker runs `kubectl get pod <name> -o yaml` to inspect full config.
- Notes additional probe fields beyond the two they set:
- **failureThreshold** (default **3**)
- **successThreshold** (default **1**)
- Meaning explained:
- *Failure threshold: doesn’t restart instantly; waits for “three consecutive failed attempts.”*
- *Success threshold: after “one successful reply,” it considers the probe healthy.*

### Auto-added tolerations (high availability behavior)

# 
- Speaker points out tolerations present that they didn’t set:
- Kubernetes auto-adds **NotReady** and **Unreachable** tolerations.
- Meaning (speaker framing):
- If node becomes not ready/unreachable, pods get evicted and moved so only healthy pods run on healthy nodes (high availability).

## Readiness probe: same syntax, different behavior

### ### Readiness probe configuration

# 

- Speaker states readiness probe syntax is the same:
- use `readinessProbe:` instead of `livenessProbe:`
- same `httpGet` / `tcpSocket` / `exec` possibilities
- same timing fields (`initialDelaySeconds`, `periodSeconds`, etc.)
- They copy an example from Kubernetes documentation and add it “at the same level” as liveness under the container.

### Key behavioral difference (speaker’s emphasis)

# 
- *Readiness probe “does not restart the application” on failures.*
- *Only when readiness is healthy does the app start serving workload/traffic.*
- Demo outcome:
- Pod “running,” but not ready because readiness fails (HTTP 500).
- Readiness failures shown in `describe`.
- Liveness failures can still cause restarts; in their case it ends in CrashLoopBackOff due to repeated failures.

## Wrap-up and next video

# 
- Speaker closes: probes should now feel “pretty simple,” and encourages completing the hands-on + Day 18 GitHub task.
- Support channels: YouTube comments / Discord community.
- Next video: **ConfigMaps and Secrets** (speaker says they ideally should’ve covered earlier).

#