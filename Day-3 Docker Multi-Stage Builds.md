Docker Multi-Stage Builds: Reduce Image Size + Key Troubleshooting Commands (CKA Series — Video 3)
==================================================================================================

1) Intro and recap of prior videos
----------------------------------

*   This is **video #3** in the CKA series.
    
*   Previous coverage:
    
    *   Docker/container fundamentals
        
    *   Dockerized a sample to-do app
        
*   This video focus: **multi-stage builds**.
    
    *   _“Multi-stage build is something that we use when we have to reduce the image size… improve the performance of Docker containers… \[a\] best practice.”_
        
*   Speaker notes their audio may sound “rusty” due to cold/throat infection.
    
*   Mentions GitHub repo contains details/links (in description).
    

**Section summary:** Video 3 introduces multi-stage Docker builds to address large image sizes seen in prior demos.

2) Why multi-stage builds: the problem being solved
---------------------------------------------------

*   Speaker restates the issue from earlier Dockerfile:
    
    *   Image was **200+ MB** even though it used **Alpine** (a lightweight base).
        
*   Goal for this video:
    
    *   Use **Docker multi-stage** to reduce image size.
        

**Section summary:** Multi-stage builds are introduced as the fix for unnecessarily large images.

3) Clone the demo application and prepare to write a new Dockerfile
-------------------------------------------------------------------

### Tools Used: Git (git clone)

*   Speaker clones an application (URL referenced as provided in description).
    
*   Then:
    
    *   Lists files (ls)
        
    *   cd into the project directory
        
    *   Confirms there is **no Dockerfile** yet
        
*   Speaker says they know how the project is built (dependencies, Node version, etc.), so they can write the Dockerfile accordingly.
    

**Section summary:** The app repo is cloned locally; Dockerfile will be authored from known app requirements.

4) Create the Dockerfile and write a multi-stage build
------------------------------------------------------

### Tools Used: touch (create empty Dockerfile)

*   Uses touch Dockerfile to create a blank file.
    

### Tools Used: vi editor

*   Opens the Dockerfile in vi and starts writing instructions.
    

### Methods Explained: Multi-stage Dockerfile structure (in the speaker’s order)

#### Stage 1: “installer” (Node build stage)

1.  **FROM base image + stage name**
    
    *   Uses node:18-alpine
        
    *   Adds a stage name:
        
        *   _“Installer is nothing but the name of the stage.”_
            
2.  **WORKDIR**
    
    *   Sets working directory to /app (same pattern as earlier videos).
        
3.  **COPY package metadata first**
    
    *   Copies package.json and package-lock.json using a wildcard pattern.
        
    *   Speaker explains wildcard intent:
        
        *   _“Any file that starts with… package and end with json.”_
            
4.  **Install dependencies**
    
    *   Runs npm install.
        
5.  **Copy remaining project content**
    
    *   Copies the rest of the project content after dependencies are installed.
        
    *   Speaker emphasizes a key idea:
        
        *   _“Till here we are not copying it to the container itself; we are copying it to the layer.”_ (their phrasing while explaining stages/layers)
            
6.  **Build the application**
    
    *   Runs npm run build.
        
    *   Output:
        
        *   _“It will generate a new folder which has all the build artifacts.”_
            

#### Stage 2: “deployer” (Nginx serve stage)

1.  **FROM nginx + stage name**
    
    *   Uses an **nginx** image (speaker says “latest”).
        
    *   Names this stage **deployer**.
        
2.  **COPY only build artifacts from Stage 1**
    
    *   Copies the **build folder** from the **installer** stage into:
        
        *   /usr/share/nginx/html
            
    *   Speaker explains why:
        
        *   Earlier approach copied everything (including node\_modules).
            
        *   Now they only want what’s required to serve the static web app:
            
            *   _“We only need the build artifacts… we don’t need node modules and other dependencies file.”_
                

#### Multi-stage definition (as described)

*   _“This is what we call a multi-stage Docker container or multi-stage build… we specify different stages and we only copy the files that are needed.”_
    
*   Benefits stated:
    
    *   Performance improvements
        
    *   Security improvements (_only copying what’s needed to serve the app_)
        
    *   “Faster,” reduced image size (speaker wording suggests size improvement)
        
    *   Some isolation and “production ready” build
        
*   Speaker saves the Dockerfile with :wq.
    

**Section summary:** The Dockerfile is split into a Node “installer” stage that builds artifacts and an Nginx “deployer” stage that ships only the build output to a static-serving directory.

5) Build the image (and fix common Dockerfile syntax errors)
------------------------------------------------------------

### Tools Used: docker build

*   Speaker runs docker build -t . (names image “multistage”).
    
*   Encounters errors and fixes them:
    
    1.  **Unknown instruction** due to misspelling **WORKDIR**
        
        *   Opens file and corrects it.
            
    2.  **Unknown flag** when copying “from installer”
        
        *   Correct syntax requires --from=installer (speaker notes it should be equals, not hyphen).
            
*   Re-runs build successfully.
    

### Tools Used: docker images

*   Verifies new image:
    
    *   Image name: “multi-stage”
        
    *   Created seconds ago
        
    *   Size shown ~**195 MB**
        
*   Speaker highlights the comparison:
    
    *   Even though this app is “bigger,” includes Nginx, and is heavier than the earlier app, **image size is lower**.
        

**Section summary:** Building multi-stage images may involve syntax gotchas (WORKDIR spelling, correct --from= usage); final image shows reduced size versus the earlier single-stage approach.

6) Cleanup old images (local disk management)
---------------------------------------------

### Tools Used: docker image rm

*   Speaker notes many images can occupy local storage.
    
*   Demonstrates cleanup with:
    
    *   docker image rm 
        
*   Initially runs an invalid command due to missing rm, then corrects it.
    
*   Explains outcome:
    
    *   It “untagged” and removed the image.
        

**Section summary:** Use docker image rm to remove unused images and free local space.

7) Run the container and validate it’s working
----------------------------------------------

### Tools Used: docker ps

*   Checks running containers; none are running initially.
    

### Tools Used: docker run

*   Runs the multi-stage image, mapping port **3000:3000** (speaker describes exposing on 3000).
    
*   Starts container and then checks with docker ps to confirm it’s running.
    

**Section summary:** The image is run as a container and validated via docker ps.

8) Troubleshooting and inspection commands (logs, exec, inspect)
----------------------------------------------------------------

### Tools Used: docker logs

*   For investigation, speaker uses docker logs :
    
    *   _“These are the standard output log… STDOUT message generated from the container itself.”_
        

### Tools Used: docker exec

*   Enters container interactively (-it) using sh.
    
*   Speaker quizzes/observes default directory:
    
    *   Because the running stage is the second stage (nginx) and no WORKDIR was set there:
        
        *   _“It took the default slash.”_ (/)
            
*   Inside the container:
    
    *   Navigates to logs under /var/log.
        
    *   Notes minimal tooling in the lightweight OS:
        
        *   ls not available (as stated)
            
        *   vi not available (as stated)
            
    *   Finds Nginx log directory and checks logs with cat.
        
    *   Observes logs are empty at the moment.
        
*   Verifies where the app files are:
    
    *   No /app folder in the final container (because final stage is nginx-based).
        
    *   Static site files are in:
        
        *   /usr/share/nginx/html
            
    *   Sees index.html and other static/metadata files.
        
    *   Emphasizes: only required static files were copied into the final image.
        

### Tools Used: docker inspect

*   Uses docker inspect to view configuration and runtime details:
    
    *   MAC address
        
    *   IPv4 address assigned to container (local IP)
        
    *   Sandbox key/host details
        
    *   Exposed ports and host port mapping (localhost:3000)
        
    *   Nginx version and other generated details
        
*   Calls it “really handy” for troubleshooting.
    

**Section summary:** The key debugging workflow shown is: docker logs for STDOUT, docker exec -it ... sh to inspect files and logs inside the container, and docker inspect to view networking/port/config metadata.

9) Additional Docker best practices (mentioned, not fully taught)
-----------------------------------------------------------------

*   Speaker says there are more best practices, including:
    
    *   Running containers as a **non-root user**:
        
        *   _“It should be run as a non root user… unprivileged user.”_
            
*   Encourages viewers to research Docker best practices (Docker website mentioned) and comment what they find.
    
*   Reiterates multi-stage build is one best practice already covered.
    

**Section summary:** Beyond multi-stage builds, the speaker flags “run as non-root” as another important production best practice and asks learners to research more.

10) Closing
-----------

*   Asks viewers to hit comment/like targets to support next video release.
    
*   Thanks viewers and signs off.
