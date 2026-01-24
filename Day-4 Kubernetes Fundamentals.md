Kubernetes Fundamentals: Why You Need It (CK 2024 — Video 4)
============================================================

1) Intro: what’s covered so far, and what this video will do
------------------------------------------------------------

*   Speaker introduces **video #4** in the **CK 2024** series.
    
*   Recap of prior videos:
    
    *   Containers fundamentals
        
    *   How to containerize an application
        
    *   Why containers are needed
        
    *   Multi-stage builds
        
*   This video is intentionally short and focuses on Kubernetes fundamentals:
    
    *   Why Kubernetes is needed
        
    *   What it takes care of
        
    *   How it’s better than “just running Docker containers”
        
    *   Challenges of running containers without orchestration
        
*   Engagement target mentioned: **150 likes and 150 comments in 24 hours**.
    

**Section summary:** The speaker transitions from Docker fundamentals to the motivation for Kubernetes and what problems it solves.

2) The “containers on a VM” scenario: what breaks without orchestration
-----------------------------------------------------------------------

*   Starting point scenario:
    
    *   A small application with **3–5 containers** hosted on a **virtual machine (VM)**.
        
    *   Everything is healthy; users/dev/ops are happy.
        
*   Failure scenario #1: a container goes down
    
    *   Could be frontend, backend, or database; _in any case it impacts real-time users_.
        
    *   Manual response:
        
        *   Ops/sysadmins **SSH into the VM**
            
        *   Check **container logs**
            
        *   Try to fix quickly to restore user experience
            
    *   Works for small apps, but introduces support coverage problems.
        

**Section summary:** With containers on a VM, failures require people to jump in manually (SSH + logs), which doesn’t scale operationally.

3) Operational coverage problem: off-hours and global users
-----------------------------------------------------------

*   If an issue happens **off-hours**:
    
    *   One or two assigned people can’t work **24/7**.
        
*   If users are global:
    
    *   Need coverage across **time zones**.
        
    *   Implies higher expense (more hiring) and is “not advisable” for small apps.
        

**Section summary:** Manual container operations create expensive 24/7 support requirements, especially for globally used apps.

4) Scale problem: enterprise apps with hundreds/thousands of containers
-----------------------------------------------------------------------

*   Enterprise scenario:
    
    *   **Hundreds or thousands** of containers running on a VM.
        
*   If multiple containers crash simultaneously (e.g., **8–10**):
    
    *   It becomes a “hassle” for a few people to investigate everything at once.
        
    *   In a **production outage**, time is limited; user impact is immediate.
        
    *   Failures could happen frequently (“every 5 minutes, 10 minutes… throughout the day”).
        

**Section summary:** At scale, manual troubleshooting and recovery can’t keep up with frequent or concurrent failures.

5) Single point of failure: VM goes down
----------------------------------------

*   If the **VM hosting the application goes down**:
    
    *   _“Your entire application will go down.”_
        

**Section summary:** Running containers on a single VM creates an obvious high-risk dependency on that VM’s uptime.

6) Release/upgrade challenge: deploying new versions at scale
-------------------------------------------------------------

*   Version rollout example:
    
    *   Current container version **0.9**
        
    *   Need to deploy **1.0**
        
*   Challenge:
    
    *   Doing this for **hundreds of containers** is hard manually.
        
    *   Even with automation, it’s still “a hassle” to implement and manage yourself.
        

**Section summary:** Coordinating upgrades across many containers is operationally painful without an orchestration platform.

7) Exposure and routing challenge: getting traffic to the right components
--------------------------------------------------------------------------

*   If services are APIs:
    
    *   Using an **API Gateway** is suggested as a good approach for routing.
        
*   But for web application components:
    
    *   Need to expose multiple user-facing endpoints.
        
    *   Might require an **external load balancer** plus routing rules.
        
    *   Speaker describes this as a hassle.
        

### Tools Mentioned: API Gateway

*   Proposed for routing when components expose API endpoints.
    

### Tools Mentioned: External Load Balancer

*   Considered for exposing user-facing endpoints and routing, but described as operationally burdensome to configure manually.
    

**Section summary:** Without orchestration, you must design and maintain traffic exposure/routing yourself (API gateway, load balancers, routing rules).

8) What Kubernetes addresses (the “answer” list)
------------------------------------------------

*   Speaker frames Kubernetes as solving the above problems with “minimum/optimized intervention.”
    
*   Areas Kubernetes takes care of (as listed by the speaker):
    
    *   Networking
        
    *   Resource management
        
    *   Security
        
    *   High availability
        
    *   Fault tolerance
        
    *   Service discovery
        
    *   Scalability
        
    *   Load balancing
        
    *   Orchestration
        

**Section summary:** Kubernetes is presented as the orchestration system that automates and standardizes reliability, scaling, networking, and operational management for containerized apps.

9) Important caution: Kubernetes is not always the solution
-----------------------------------------------------------

*   Speaker explicitly warns:
    
    *   _“Kubernetes is not always the solution.”_
        
*   Example: small to-do list app with only a couple of containers
    
    *   Using Kubernetes for that is described as:
        
        *   Wastage of resources
            
        *   Wastage of money
            
        *   Added administrative effort / “toil”
            
*   Even with managed Kubernetes services:
    
    *   Examples given: **AKS**, **EKS**, **GKE**
        
    *   Still requires admin work:
        
        *   Manage/maintain clusters
            
        *   Optimize workloads
            
        *   Plan upgrades and patching
            
*   Speaker suggests evaluating alternatives depending on needs:
    
    *   **Docker Compose**
        
    *   Containers directly on bare metal or a VM
        
    *   A **virtual private server (VPS)** option like:
        
        *   DigitalOcean droplet
            
        *   AWS Lightsail
            
        *   Marketplace images from GCP or Azure
            
    *   Rationale: lower cost, lower admin effort, lower maintenance
        

### Tools Mentioned: Managed Kubernetes Services

*   **Azure Kubernetes Service (AKS)**
    
*   **Amazon Elastic Kubernetes Service (EKS)**
    
*   **Google Kubernetes Engine (GKE)**
    

### Tools Mentioned: Docker Compose

*   Suggested as a simpler alternative for small apps.
    

**Section summary:** Kubernetes adds operational overhead; for small/simple workloads, simpler container hosting options may be more cost-effective and easier to run.

10) Wrap-up and what’s next
---------------------------

*   Speaker concludes they’ve given a basic understanding of why Kubernetes is needed.
    
*   Repeats request to meet the like/comment target and subscribe (optional).
    
*   Next video topic:
    
    *   Kubernetes architecture and basic Kubernetes fundamentals.
