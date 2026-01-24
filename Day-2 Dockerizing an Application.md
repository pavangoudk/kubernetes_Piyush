Dockerizing a Node.js To‑Do App (CKA 2024 Series — Video 2)
===========================================================

1) Intro: where this video fits in the series
---------------------------------------------

*   Speaker introduces themself and says this is **video #2** in the **Certified Kubernetes Administrator (CKA)** end-to-end playlist (concepts + demos + hands-on), aligned to **2024 curriculum**.
    
*   Recap of video #1: container fundamentals (why needed, how they work, difference vs virtual machines).
    
    *   If you already know container fundamentals, you can skip the prior video; otherwise, start there.
        
*   This video focus: a beginner-friendly **hands-on demo** of **dockerizing an application**:
    
    *   How to write a **Dockerfile**
        
    *   How to “send instructions to Docker engine” to run commands
        
    *   How to run/host the app using a container
        
*   Speaker notes all materials will be in their GitHub repository (mentioned, not shown in transcript).
    

**Section summary:** This is the first practical “dockerize an app” walkthrough in the CKA 2024 sequence.

2) Setup options to follow along (local Docker vs sandbox)
----------------------------------------------------------

### Tools Used: Play with Docker (sandbox)

*   Speaker recommends installing Docker regardless of OS so you can follow hands-on.
    
*   If you _can’t_ install Docker (resource constraints, other issues), use Docker’s temporary sandbox:
    
    *   Go to “Play with Docker” site, sign in/sign up using Docker credentials (Docker Hub account).
        
    *   Then use the lab URL (described as “[labs.playwithdocker.com](http://labs.playwithdocker.com/)”) and click **Start**.
        
    *   Add a new instance to provision a **lightweight virtual machine**.
        
    *   Session has a **4-hour timer**; after that, it terminates.
        
    *   It provides an internal/private IP and allows opening ports to access the app.
        

### Tools Used: Docker Desktop (local installation)

*   Speaker shows Docker Desktop download page with OS options (Mac/Windows/Linux).
    
*   Installation is described as straightforward (“next next next… accept terms”).
    
*   Docker Desktop UI overview:
    
    *   Shows running containers
        
    *   Shows images
        
    *   Distinguishes **local image registry** (“local”) vs **remote repository** (Docker Hub)
        
    *   Mentions integration with **Artifactory**.
        

### Tools Used: JFrog Artifactory

*   Artifactory is described as “a tool by JFrog,” an image repository where you can store:
    
    *   Docker images
        
    *   Other packages (tar/zip/npm packages, etc.)
        

**Section summary:** You can run the demo either locally with Docker Desktop or in a 4-hour Play-with-Docker sandbox; the rest of the video uses the speaker’s local Docker install.

3) Demo plan and starting point (diagram + sample app)
------------------------------------------------------

*   Screen layout described:
    
    *   Left: terminal for commands
        
    *   Right: reference diagram (Docker flow from video #1)
        
*   First step in the flow: create the **Dockerfile**.
    
*   Before Dockerfile: you need an application to dockerize.
    
*   Speaker pulls a **sample to-do list app** from an existing Git repo to use as the demo app.
    

**Section summary:** The hands-on follows the classic flow: app source → Dockerfile → image → registry → pull/run.

4) Get the sample application locally
-------------------------------------

### Tools Used: Git (git clone)

Steps shown (in order):

1.  Create a new folder (named like “day02 code”) and cd into it.
    
2.  Verify it’s empty with ls.
    
3.  Run **git clone** to download the repository into the local folder.
    
4.  cd into the cloned folder (named “getting-started-app”).
    
5.  List files; speaker points out typical contents:
    
    *   README
        
    *   package.json
        
    *   source folder
        
    *   yarn lockfile (mentioned)
        

**Section summary:** The demo app is obtained via git clone and prepared as the source to containerize.

5) Create and edit the Dockerfile
---------------------------------

### Methods Explained: Writing a Dockerfile (instructions in order)

*   Speaker creates an empty Dockerfile using a file creation command, then edits it using a terminal editor.
    

### Tools Used: touch (create file)

*   Used to create an empty file: touch Dockerfile.
    
*   Naming convention: default is **Dockerfile** (capital D).
    

### Tools Used: vi editor

*   Open the Dockerfile in vi.
    
*   vi starts in escape mode; to edit:
    
    *   Press **i** to enter insert mode (speaker notes “insert” appears bottom-left).
        
*   Exit/save behavior described:
    
    *   Press **Esc**, then : then wq! to save and quit.
        
    *   q! quits without saving.
        

### Methods Explained: Dockerfile instructions used (sequence preserved)

1.  **Base image**
    
    *   Need a base image to run the app.
        
    *   Speaker chooses a Node base image rather than Ubuntu/CentOS/Fedora for lightness and convenience.
        
    *   Rationale: _a Linux-based OS with Node already installed_.
        
2.  **Node image selection via Docker Hub**
    
    *   Speaker searches Docker Hub for **node** and notes:
        
        *   It’s an “official” image (safe/stable as described).
            
        *   It has multiple **tags**.
            
        *   _“Tags are nothing but different version of this Docker image.”_
            
        *   _If you don’t specify a tag, it pulls “latest.”_
            
    *   They use a Node image based on **Node 18 Alpine**:
        
        *   _“Alpine is a lightweight Linux based operating system… minimum dependencies… minimum libraries… not take a lot of space.”_
            
3.  **WORKDIR**
    
    *   Creates/uses a working directory inside the container (e.g., /app).
        
    *   _“Work directory is where you will be doing all your work inside the container.”_
        
4.  **COPY**
    
    *   Copy files from local repo into the container.
        
    *   Speaker explains the dot notation:
        
        *   Source . = current local directory
            
        *   Destination . = current working directory in the container (after WORKDIR)
            
    *   _WORKDIR is described as equivalent to “cd into /app.”_
        
5.  **RUN (install dependencies)**
    
    *   Install application dependencies using Yarn.
        
    *   Speaker notes non-developers may not know these commands; it’s a sample app.
        
    *   They emphasize that in many enterprises, developers create Dockerfiles, but ops/DevOps/cloud engineers should understand Dockerfile syntax and flow end-to-end.
        
6.  **CMD (start the app)**
    
    *   Speaker describes this as the command executed when you run the container.
        
    *   They run Node to execute an entry file under src (described as index.js).
        
7.  **EXPOSE**
    
    *   Expose the application on a port (they choose **3000**).
        
    *   _Without exposing, “your application will not be rendered on a public internet or on a specific port.”_
        

*   After saving, speaker confirms the Dockerfile exists and has content.
    

**Section summary:** The Dockerfile is built from base image → working directory → copy source → install dependencies → run app → expose port 3000.

6) Build the Docker image
-------------------------

### Tools Used: Docker CLI help (--help)

*   Speaker shows you can discover commands via:
    
    *   docker --help
        
    *   docker build --help
        

### Methods Explained: Build command and what it does

*   Build command uses a tag/name and a build context:
    
    *   docker build -t .
        
    *   The . means “use all the files inside the current directory” (build context) and the Dockerfile is in the current directory.
        
*   Build output explanation:
    
    *   Docker builds in **layers**:
        
        *   Speaker counts steps in Dockerfile and says each step becomes a layer.
            
        *   _“This is how a Docker creates the image… create the image in layers and then combine it together.”_
            
    *   Layering helps faster processing and maintenance; when pushing/pulling it ships layers and reassembles.
        

### Tools Used: docker images

*   After build, speaker verifies the image exists locally:
    
    *   Image name (their chosen tag), tag defaults to **latest** if none specified.
        
    *   Shows image size (noted as ~217 MB in their example).
        

### Tools Used: Docker Desktop UI (Images)

*   Speaker cross-checks the image in Docker Desktop → Images (local).
    
*   Points out layer view (Docker Desktop shows many layers).
    

**Section summary:** The image is built locally with docker build -t ... ., verified via docker images and Docker Desktop; Docker uses layered builds.

7) Push the image to Docker Hub (registry step)
-----------------------------------------------

### Tools Used: Docker Hub (remote registry)

*   Speaker creates a new Docker Hub repository (public) (e.g., “test-repo”).
    
*   Notes initially there are no tags/builds.
    

### Methods Explained: Tag then push

1.  **Tag the image**
    
    *   You must tag the local image with the Docker Hub repo name (username/repo:tag).
        
    *   After tagging, docker images shows both:
        
        *   Original local image name
            
        *   New username/test-repo:latest tag
            
2.  **Push the image**
    
    *   First push attempt fails due to:
        
        *   Misspelling (fixed)
            
        *   Then “requested access… denied” because not authenticated
            
    *   Speaker logs in, then push succeeds.
        

### Tools Used: docker login

*   Used to authenticate to Docker Hub with the same credentials used to sign up.
    
*   Success message: login succeeded.
    

### Tools Used: docker push

*   Push publishes layers and returns a SHA256 digest (unique ID for the image, as described).
    
*   Speaker verifies on Docker Hub that a new tag appears.
    

### Observation: compression effect on size (as described)

*   Speaker compares sizes:
    
    *   Local image ~217 MB
        
    *   On Docker Hub ~81 MB
        
*   Explanation given: it “automatically compress\[es\] that file.”
    

**Section summary:** Registry workflow is: create repo → tag image with username/repo:tag → docker login → docker push → verify tag in Docker Hub.

8) Pull and run the container, then access the app
--------------------------------------------------

### Tools Used: docker pull

*   Speaker pulls the image.
    
*   Since the image already exists locally (built on same machine), it says “up to date.”
    
*   Key behavior highlighted:
    
    *   _If changes exist, Docker downloads only the changed layer, not the entire image._
        

### Tools Used: docker run

*   Speaker checks docker run --help briefly for options.
    
*   Run command options used:
    
    *   \-d detach mode (run in background)
        
    *   \-p port binding: external 3000 to internal 3000
        
*   Running outputs a **container ID**.
    

### Tools Used: docker ps

*   Used to confirm the container is running and shows uptime.
    
*   Since no name was provided, Docker assigns a random container name (example shown).
    

### Accessing the application

*   Because port 3000 is mapped, speaker accesses the app at:
    
    *   localhost:3000
        
*   App behavior shown:
    
    *   A simple to-do list interface where items can be added/removed.
        

**Section summary:** After pulling, docker run -d -p 3000:3000 ... starts the app container; docker ps confirms it; app is reachable on localhost:3000.

9) Basic troubleshooting: exec into the container + a best-practice teaser
--------------------------------------------------------------------------

### Tools Used: docker exec

*   Used to “go inside” the container, similar to SSH (speaker’s analogy).
    
*   They run it in interactive mode (described as using -it).
    
*   Shell nuance:
    
    *   bash fails because Alpine-based images use sh.
        
    *   They switch to sh and get in successfully.
        

### What they observe inside the container

*   They land in /app by default (because of WORKDIR).
    
*   ls shows the application files, including node\_modules.
    
*   Speaker says having node\_modules like this is “not a best practice” (stated but not fixed in this video).
    

### Best-practices next step (preview only)

*   Speaker notes the image size (~200 MB) is too big for an Alpine-based simple app.
    
*   They tease the next video will cover how to reduce image size and apply best practices, asking viewers to comment if they know what to use.
    

**Section summary:** docker exec -it sh is used for troubleshooting; the speaker previews optimization/best practices to reduce image size in the next video.

10) Closing notes
-----------------

*   Encourages viewers to dockerize any sample app from GitHub or their own and practice.
    
*   Mentions all links, Dockerfile, and diagrams will be shared in the GitHub repo.
    
*   Sets engagement target again (comments/likes) and says next video will come soon.
