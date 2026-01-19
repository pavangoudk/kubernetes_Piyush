## Dockerizing an Application: Certified Kubernetes Administrator Series - Video 2

### Overview 📚
This video provides a hands-on demonstration for beginners on how to dockerize an application, a fundamental skill for Kubernetes certification and containerized environments. Starting from scratch, it shows how to write a Dockerfile, build Docker images, and run containers by sending instructions to the Docker engine. The presenter walks through practical steps using a sample to-do list app, emphasizing key commands and best practices. The tutorial highlights the entire lifecycle from local development to pushing images to Docker Hub and running applications in different environments, enabling viewers to grasp core Docker operations essential for Kubernetes administrators.

### Summary of Core Knowledge Points ⏱️
- **00:00 - 01:09 | Introduction and Prerequisites**  
  The video series accompanies the Certified Kubernetes Administrator curriculum of 2024. The first video explained container fundamentals, while this one focuses on dockerizing an application step-by-step. Viewers are encouraged to install Docker or use the “play with Docker” online sandbox to follow along practically.

- **01:09 - 03:55 | Using Docker Sandbox Environment (Play with Docker)**  
  Explanation on accessing the free Docker playground environment. Users sign up/sign in to Docker Hub, navigate to labs.play-with-docker.com, start a sandbox instance with a 4-hour timer, and get a Linux VM with private IP. This is useful if Docker cannot be installed locally.

- **03:55 - 05:44 | Installing Docker Desktop**  
  Instructions for downloading and installing Docker Desktop on Windows, Mac, or Linux. After installation, the Docker Desktop GUI shows local containers, images, and remote repositories like Docker Hub and JFrog Artifactory.

- **05:44 - 07:47 | Preparing the Application Source Code**  
  Creation of a local working directory followed by cloning a sample to-do app repository via Git. Directory contents are reviewed to confirm the source code and other files are ready for dockerization.

- **07:47 - 09:21 | Creating the Dockerfile**  
  Using the `touch` and `vi` commands to create and edit the Dockerfile, starting in insert mode to add necessary instructions.

- **09:21 - 16:36 | Writing Dockerfile Instructions**  
  Key Dockerfile instructions and their purposes:  
  - **FROM**: Specifies the base image (node:18-alpine for lightweight Node environment).  
  - **WORKDIR**: Sets the working directory inside the container (/app).  
  - **COPY**: Copies local files to the container’s work directory.  
  - **RUN**: Runs commands inside container — here, installs dependencies using `yarn install --production` to optimize package installation.  
  - **CMD**: The command that runs when the container starts (runs `node src/index.js`).  
  - **EXPOSE**: Opens port 3000 for accessing the app externally.  
  This segment also explains the importance of these steps and notes that Dockerfiles are typically written by developers, though DevOps roles should understand the entire process.

- **16:36 - 21:11 | Building Docker Image and Image Layers**  
  Using `docker build -t day02-todo .` to create the image. The build process happens in layered steps corresponding to each Dockerfile instruction, allowing efficient caching and transfer. The layers are shown with unique IDs and size details.

- **21:11 - 24:38 | Managing Docker Images Locally and on Docker Hub**  
  Listing images locally with `docker images`. Introduction to pushing images to Docker Hub: first create a repository on Docker Hub, then tag the local image appropriately before pushing.

- **24:38 - 26:45 | Tagging and Pushing Image to Docker Hub with Authentication**  
  Tagging the image in the format `username/repo:tag`, then logging in to Docker Hub (`docker login`) before successfully pushing it. Explains common error when unauthenticated and importance of login even for public repos.

- **26:45 - 28:40 | Image Compression and Viewing Uploaded Layers**  
  Docker Hub compresses images automatically; local image of 217 MB compressed to 81 MB remotely. Users can view image layers and metadata within Docker Hub’s UI.

- **28:40 - 31:21 | Pulling Image on Another Environment and Running Container**  
  Pulling the image via `docker pull`. Running the container using `docker run` with flags:  
  - `-d` for detached mode (run in background)  
  - `-p 3000:3000` to map container port to host port  
  After startup, the app is accessible via `localhost:3000`. This emulates deployment workflow across dev, test, and prod environments.

- **31:21 - 33:10 | Accessing Running Containers and Troubleshooting**  
  Docker exec allows shell access inside a running container. Alpine base images use `sh` instead of `bash`. Listing working directory contents and acknowledging some practices like node_modules inclusion could be improved.

- **33:10 - 34:37 | Optimizing Image Size and Best Practices Preview**  
  Discusses the unexpectedly large image size for a simple app and hints at best practices to reduce image size and improve Dockerfile efficiency. Viewers encouraged to experiment and engage for upcoming videos that cover optimization techniques.

### Key Terms and Definitions 📝
- **Dockerfile**: A text file with instructions on how to build a Docker image; contains commands like FROM, COPY, RUN, CMD, and EXPOSE.  
- **Base Image**: The initial image from which your container image is built, e.g., `node:18-alpine` includes Node.js on a lightweight Linux.  
- **Docker Build**: Command to create a Docker image from a Dockerfile.  
- **Image Layers**: Incremental changes generated by Dockerfile commands that optimize image storage and transfer.  
- **Docker Hub**: Centralized registry service to store and share container images publicly or privately.  
- **Detached Mode (-d)**: Running a container in the background.  
- **Port Mapping (-p)**: Mapping a container port to a host port to expose containerized applications externally.  
- **Yarn**: A package manager for JavaScript used to install app dependencies.  
- **Docker Exec**: Command to run a shell or command inside a running container.  
- **Alpine Linux**: A minimal Docker base image known for its small size.  

### Reasoning Structure 🔍
1. **Premise:** To dockerize an application, you need a container image with your app and dependencies.  
2. **Reasoning:** Write a Dockerfile specifying a lightweight base image, copy application files, install dependencies, set the command to run the app, and expose required ports. Every instruction creates a layer for efficiency.  
3. **Conclusion:** Build the image locally, tag it, then push it to Docker Hub for sharing. Deploy the container on any environment by pulling the image and running it with port bindings. Use troubleshooting commands to interact with running containers. This creates a consistent, portable app environment across multiple stages.

### Examples 🎯
- **Using Play with Docker sandbox**: Demonstrates how users can run Docker commands and experiments without installing Docker locally, helpful for limited-resource systems.  
- **Sample To-do List App**: A simple Node.js app is cloned from GitHub and dockerized to illustrate real-world Dockerfile creation, image build, push to repo, and container run processes.  
- **Port Binding Example**: Explains mapping host port 3000 to container port 3000 so the app is accessible on localhost, a common scenario in containerized apps.

### Error-prone Points ⚠️
- **Docker Hub authentication required for pushing even public repos**: Avoid the error requested access denied by running `docker login` before push.  
- **Base image choice impacts image size and dependencies**: Starting with a full OS base image (e.g., Ubuntu) without preinstalled node will increase image size unnecessarily compared to using node:alpine.  
- **Shell differences in containers (bash vs sh)**: Alpine images use `sh` shell; trying to run `bash` will fail.  
- **Layer creation behavior**: Not all Dockerfile commands create layers, some RUN commands may be squashed depending on Docker version or build flags.  
- **Inclusion of node_modules in the image**: Having local node_modules inside the image is not best practice as it can increase size and cause inconsistencies.

### Quick Review Tips / Self-Test Exercises ✍️
**Tips (No Answers):**  
- What is the purpose of the `FROM` instruction in a Dockerfile?  
- How does Docker use image layers to optimize image builds?  
- Which command maps container ports to host ports?  
- Why must you run `docker login` before pushing images to Docker Hub?  
- How do you access an interactive shell inside a running container?  

**Exercises (With Answers):**  
1. **Fill in the blank:** The command to build a Docker image from the current directory and tag it as `myapp` is `docker __ -t myapp .`  
   *Answer:* `build`  
2. **Q:** Which Dockerfile instruction copies content from the local machine into the container?  
   *A:* `COPY`  
3. **Q:** How do you run a container in detached mode and expose port 3000 on the host?  
   *A:* `docker run -d -p 3000:3000 imagename`  
4. **Q:** What lightweight base image was used in the video demo for Node.js?  
   *A:* `node:18-alpine`  
5. **Q:** Which command do you use to push a Docker image to Docker Hub after tagging it?  
   *A:* `docker push username/repo:tag`  

### Summary and Review 🔄
This video thoroughly covers dockerizing a Node.js application starting from creating a Dockerfile with base image selection, file copying, dependency installation, and port exposure. It guides through building an image, verifying local images, pushing images to Docker Hub with authentication, pulling images in other environments, and running containers with port bindings. Troubleshooting with Docker exec and initial best practice considerations for image size optimization were introduced. The skills demonstrated are foundational for Kubernetes certification and practical container management, bridging developer application packaging and operations deployment workflows. This session equips learners with end-to-end Docker usage necessary for container orchestration topics ahead.