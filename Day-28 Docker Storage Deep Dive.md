# Docker Storage Deep Dive: Layers, Writable Containers, Volumes, and Bind Mounts (CK 2024, Video 28)

## Intro (why this video exists)

# 

- Speaker introduces **video #28** in the CK 2024 series.
- Focus: Docker storage “behind the scene” and how to make container data **persistent**.
- Context: Next video covers Kubernetes storage (persistent volumes, persistent volume claims, etc.), so this is a prerequisite.
- Speaker suggests skipping if you already know Docker storage.
- Like/comment target: **200 likes** and **150 comments** in 24 hours.

## Demo setup (project + image build)

# 
- Speaker clones a GitHub repo (same simple “to-do app” used earlier).
- `cd` into the cloned directory.
- Creates a Dockerfile by copying prepared instructions.
- Builds the image using `docker build`.

### ### Docker (daemon access / permissions)

# 
- Build fails with permission error connecting to Docker daemon (`docker.sock`).
- Speaker tries to fix by changing permissions on `/var/run/docker.sock`.
- Confirms Docker is running.
- Proceeds by running the build with `sudo` to avoid spending time on troubleshooting.

## Docker image layers (what gets stored and why)

# 
- Speaker explains Dockerfile instructions produce layers:
- Main build instructions become **four layers** (one per instruction that stores filesystem changes).
- `EXPOSE` and `CMD` don’t create meaningful storage layers (they don’t store filesystem content the same way).
- Notes layer sizes (base image layer is the main size shown).

### Layer caching benefit (“delta changes”)

# 
- If only one instruction changes, Docker rebuilds only the affected layer and reuses the rest from cache.
- *“Delta changes” = only the change you made gets rebuilt; the rest comes from Docker cache.*

## Read-only image layers + immutable infrastructure

# 
- Speaker emphasizes images are read-only once built:
- *“Right now these are read-only layers… we cannot make the change to the image directly.”*
- Benefit:
- Same image can be promoted across environments without modification.
- This is tied to *“immutable infrastructure”* (promote the same image after testing).

## Writable container layer (what it is + limitation)

# 
- Speaker introduces the need for persistence for stateful apps.
- *A “writeable layer” is also called the “container layer.”*
- Demo flow:
1. Run a container from the built image.
2. Exec into the container.
3. Create a directory (e.g., `test-demo`) inside the container.
- Meaning:
- The change is written into the **container layer**.
- The container layer is a writable copy layered on top of the read-only image layers.
- Limitation:
- *Data lasts only for the lifetime of the container; once the container is removed, the data is deleted.*

## Storage drivers vs volume drivers (two separate roles)

# 
- Speaker says these concepts look similar but serve different responsibilities.

### ### Storage drivers

# 
- *Storage drivers write data in the container layer and also create/store the image layers in the layered architecture.*
- Examples listed:
- overlay2, ZFS, VFS, AUFS, device mapper, etc.
- Notes:
- **overlay2** is common on Ubuntu and other Linux flavors.
- AUFS and device mapper are mentioned as deprecated/out of support.

### ### Volume drivers

# 
- *Volume drivers “make the data persistent,” ensuring data remains even if containers stop/restart, and can be shared across containers.*
- Examples mentioned:
- Default **local** volume driver (stores on host)
- Cloud options like AWS **EBS**, GCP persistent disk, Azure file storage
- Third-party providers like VMware, Portworx

## Demo: data disappears without volumes

# 
1. Exit container
2. Stop container
3. Remove container

- Result: the directory created earlier is gone (no persistence).

### ### Docker volumes (create + where stored + mount into container)

# 
**Create a volume**

- Speaker corrects a mistyped command and uses:
- `docker volume create data-volume`

**List volumes**

- `docker volume ls`
- Shows the volume exists and uses the **local** driver.

**Where Docker stores data on Linux**

- Docker data root: `/var/lib/docker`
- Volumes directory: `/var/lib/docker/volumes`
- Volume data path: `/var/lib/docker/volumes/data-volume/_data`
- Speaker notes you may need `sudo` to inspect.

**Mount volume into a container**

- Speaker runs the container with `-v` to mount the named volume to the container’s app/working directory.
- Inside the container:
- Create `test-demo` again.
- On the host:
- The container’s app directory contents appear under the volume `_data` directory.
- Takeaway:
- Volume-mounted data is persisted outside the container and synced as used.

## Persistence verification (stop/restart/delete container)

# 
- Stop container → data remains in the volume directory.
- Restart/exec → `test-demo` still exists.
- Force remove container (`docker rm -f`) and run a new container using the same volume:
- `test-demo` still exists.
- Key point:
- *Deleting the container does not delete the volume’s data.*

### ### Bind mounts (alternative to named volumes)

# 
- Speaker contrasts volume mounts vs bind mounts using Docker docs.

**Volume mount**

- `--mount` (or `-v`) with:
- type = volume
- source = volume name
- destination = container path
- volume driver = local (in the example)

**Bind mount**

- `--mount` (or `-v`) with:
- type = bind
- source = host directory path
- target = container path
- Speaker’s phrasing:
- *“We are not now binding the volume… we are now binding the local file system storage with the container file system storage.”*

### ### `-v` vs `--mount` (mentioned)

# 
- Speaker notes both work; syntax differs.
- Suggests reading the documentation to understand differences.

## Wrap-up (what the speaker says you should know next)

# 
- Covered in sequence:
1. Layered architecture + caching (*“delta changes”*)
2. Read-only image layers vs writable container layer
3. Why container-layer data is not persistent
4. Storage drivers vs volume drivers
5. Volumes: creation, host location, persistence across lifecycle
6. Bind mounts vs volumes
- Speaker reiterates this is a prerequisite for Kubernetes storage in the next video.

# 

# 

# 

# 

# 

# 

# 

#