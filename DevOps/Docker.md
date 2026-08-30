## 1. Introduction: What is Docker & Why Containers?

- **The Core Problem in Traditional Development:** In traditional setups, developers must install every service (e.g., PostgreSQL, Redis) directly onto their local operating systems. Because different team members use different operating systems and manual configuration involves multiple multi-step commands, the chances of environmental inconsistencies and errors are high. Furthermore, running different versions of the same service on a single host frequently triggers dependency conflicts.
- **The Traditional Deployment Gap:** Developers traditionally produce application artifacts (such as a JAR file) alongside textual configuration manuals for operations teams. Operational handoffs often fail due to misinterpretations or missing instructions, resulting in lengthy debugging cycles between development and operations.
- **The Container Solution:** A **container** is a portable, isolated package containing an application and all its required binaries, libraries, dependencies, and configuration. Because containers encapsulate their own lightweight Linux runtime layer, developers can start services across any host operating system with a single command without installing binaries directly onto their host OS. In deployment, operations simply install the Docker runtime once on the host and run the exact same container images without manual OS-level configuration.

---

## 2. Docker Architecture: Containers vs. Virtual Machines

Operating systems consist of two primary layers: the **kernel layer** (which interfaces directly with hardware like CPU and memory) and the **application layer** (which houses userland tools and distributions such as Ubuntu, Debian, or Alpine).

| Feature                               | Docker Containers                                                                | Virtual Machines (VMs)                                                                     |
| :------------------------------------ | :------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------- |
| **Virtualization Scope**              | Virtualizes only the **application layer**.                                      | Virtualizes the complete operating system, including the **kernel** and application layer. |
| **Kernel Usage**                      | Shares the host machine's operating system kernel.                               | Boots its own dedicated OS kernel on top of a hypervisor.                                  |
| **Image Size**                        | Small (typically megabytes) because base images like Alpine are minimal.         | Large (typically gigabytes) due to full guest OS installations.                            |
| **Startup Speed**                     | Starts almost instantaneously (seconds).                                         | Slower boot times because the full kernel and OS must load.                                |
| **Compatibility & Kernel Dependency** | Requires host kernel compatibility (e.g., Linux containers need a Linux kernel). | Can run any guest OS on any host OS regardless of host kernel.                             |

Virtual Machine gives:
- Hardware level isolation (means stronger isolation)
- Slow
- Heavy

Docker gives:
- Process level isolation with shared host kernel (means weaker isolation)
- Fast
- Lightweight

> **Legacy OS Support (Docker Toolbox):** On legacy platforms that do not support native Docker virtualization (such as Windows versions prior to Windows 10 or older macOS versions), **Docker Toolbox** uses Oracle VirtualBox to run a lightweight Linux VM that hosts the Docker Engine.

---

## 3. Images, Containers, & Registries

### Technical Distinction

- **Docker Image:** An immutable, packaged artifact consisting of stacked, read-only filesystem layers containing the base operating system, runtime libraries, application code, and default configuration.
- **Docker Container:** The running, isolated runtime environment instantiated from an image. A container adds a writable container layer, virtualized networking ports, environment variables, and an isolated file system.

### Layered Architecture

Images are constructed out of stacked layers. When downloading or updating images (e.g., pulling PostgreSQL `9.6` and then `10.10`), Docker checks whether underlying layers already exist locally. Shared layers are reused rather than re-downloaded, drastically cutting download times and storage footprints.

### Registries & Naming Conventions

- **Container Registry:** A storage system for sharing and distributing container images.
- **Public Registry:** **Docker Hub** hosts over 100,000 official and community images. Pulling images like `docker pull mongo:4.2` is a shorthand for `docker pull docker.io/library/mongo:4.2`.
- **Private Registries (e.g., AWS ECR):** Require full domain qualification and explicit authentication before pushing or pulling:
    
    ```
    [registry-domain]/[repository-name]:[tag]
    ```
    
    _(Example: `123456789.dkr.ecr.eu-central-1.amazonaws.com/my-app:1.0`)_.

---

## 4. Essential Docker CLI Commands & Workflow

```
# Image Management
docker pull <image>:<tag>           # Pulls an image from a registry
docker images                       # Lists locally available images
docker rmi <image-id-or-name>       # Removes a local image (container must be removed first)

# Container Lifecycle
docker run [OPTIONS] <image>:<tag>  # Pulls (if missing) and creates/starts a new container
docker start <container-id-or-name> # Restarts an existing stopped container preserving its settings
docker stop <container-id-or-name>  # Stops a running container
docker rm <container-id-or-name>    # Removes a stopped container

# Inspection & Monitoring
docker ps                           # Lists active running containers
docker ps -a                        # Lists all containers (running and stopped)
docker logs <container-name-or-id>  # Fetches console logs of a container
docker logs -f <container-id>       # Streams live container logs

# Execution & Debugging
docker exec -it <container> /bin/sh # Opens an interactive terminal inside the container
```

### Key `docker run` Flags

- `-d`: Runs the container in **detached mode** (in the background, freeing the terminal).
- `--name <name>`: Assigns a custom, human-readable name to the container.
- `-p <host-port>:<container-port>`: Binds a port on the host machine to a port inside the container. This allows multiple containers listening on the same internal port (e.g., Redis on `6379`) to run simultaneously by mapping them to separate host ports (e.g., `6000:6379` and `6001:6379`).
- `-e <KEY>=<VALUE>`: Injects runtime environment variables into the container.
- `--net <network-name>`: Attaches the container to a specific Docker network.
- `-v <volume-name-or-path>:<container-path>`: Mounts persistent storage into the container.

---

## 5. Docker Networking

- **Network Isolation:** By default, Docker creates isolated virtual networks. Containers on the same custom network can communicate directly with each other using their **container names** as hostnames without relying on host IP addresses or port mappings.
- **Host Access:** Applications running on the host system outside of Docker communicate with containers via `localhost` and the host-bound port.
- **Network Management Commands:**
    
    ```
    docker network ls                  # Lists all networks
    docker network create mongo-net    # Creates a custom user-defined network
    ```
    

---

## 6. Dockerfile: Building Custom Images

A `Dockerfile` is a text blueprint named exactly `Dockerfile` containing step-by-step instructions to assemble a custom image.

### Core Instructions

- `FROM <base-image>:<tag>`: Defines the parent base image (e.g., `FROM node:13-alpine`).
- `ENV <KEY>=<VALUE>`: Sets persistent environment variables inside the image.
- `RUN <linux-command>`: Executes commands during the build phase _inside_ the container layer to install packages or create directories (e.g., `RUN mkdir -p /home/app`).
- `COPY <host-source> <container-dest>`: Copies files or folders from the host build context into the container filesystem (e.g., `COPY ./app /home/app`).
- `CMD ["executable", "param"]`: Specifies the default container entry point command executed when the container starts (e.g., `CMD ["node", "server.js"]`).

### `RUN` vs. `CMD`

- `RUN` commands execute at **build time** to install dependencies, set up directories, and commit new image layers. Multiple `RUN` commands can exist in a single Dockerfile.
- `CMD` represents the primary **runtime instruction** executed when the container boots. There is only one active `CMD` instruction marking the container process.

### Build & Image Tagging

```
# Build an image with a name tag from the current directory context
docker build -t my-app:1.0 .

# Tag an existing image for a remote private registry
docker tag my-app:1.0 <aws-account-id>.dkr.ecr.<region>.amazonaws.com/my-app:1.0

# Authenticate and push to remote repository
docker push <aws-account-id>.dkr.ecr.<region>.amazonaws.com/my-app:1.0
```

---

## 7. Multi-Container Orchestration with Docker Compose

**Docker Compose** replaces long, complex CLI `docker run` commands with declarative YAML files (`docker-compose.yaml` or `mongo.yaml`), allowing you to manage multi-container applications as a single service stack.

### Key Benefits

- **Automatic Networking:** Docker Compose automatically creates a shared default network for all defined services, enabling inter-container communication using service keys as DNS hostnames.
- **Centralized Configuration:** Ports, environment variables, dependencies, and volumes are maintained in one structured file.

### Example `docker-compose.yaml` (Mongo + Mongo Express + Web App)

```
version: '3' # Compose file format version

services:
  my-app:
    image: my-app:1.0 # Or remote registry image URI
    ports:
      - "3000:3000" # <host-port>:<container-port>
    environment:
      - MONGO_DB_USERNAME=admin # Injected environment settings
      - MONGO_DB_PASSWORD=password

  mongodb:
    image: mongo:latest # Base image from Docker Hub
    ports:
      - "27017:27017" #
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin #
      - MONGO_INITDB_ROOT_PASSWORD=password #
    volumes:
      - mongo-data:/data/db # Mounts named volume to internal storage directory

  mongo-express:
    image: mongo-express # Web-based database management UI
    ports:
      - "8080:8081" # Map host 8080 to container UI port 8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin #
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password #
      - ME_CONFIG_MONGODB_SERVER=mongodb # Connects directly via service name

volumes:
  mongo-data: # Declares named volume at the root level
    driver: local #
```

### Compose CLI Commands

```
# Start all containers in the stack (creates networks and volumes)
docker-compose -f mongo.yaml up

# Stop and remove containers and associated networks
docker-compose -f mongo.yaml down
```

---

## 8. Data Persistence: Docker Volumes

### Why Volumes Are Needed

Containers are stateless by design. The container file system is virtual and ephemeral; when a container is stopped or recreated, any database records or state saved inside its virtual file system are permanently lost. **Volumes** map a directory on the host's physical file system into a folder within the container's virtual file system, replicating writes bidirectionally so data survives container restarts.

### The Three Volume Types

```
1. Host Volumes / Bind Mounts:
   Host: /home/user/db-data  <=======>  Container: /data/db
   (Explicit host folder chosen by user)

2. Anonymous Volumes:
   Host: /var/lib/docker/volumes/<hash>/_data  <=======>  Container: /data/db
   (Automatically generated path; no reference name)

3. Named Volumes (Production Best Practice):
   Host: /var/lib/docker/volumes/<volume-name>/_data  <=======>  Container: /data/db
   (Docker manages physical directory, referenced easily by name)
```

- **Host Volumes (Bind Mounts):** You explicitly specify the exact host directory path to mount into the container using `-v /host/path:/container/path`.
- **Anonymous Volumes:** Created without a reference name (`-v /container/path`). Docker automatically assigns an obscure hash folder inside Docker storage.
- **Named Volumes:** The recommended approach for production. You provide a readable identifier (e.g., `mongo-data`), and Docker manages the underlying storage location. Multiple containers can mount the same named volume to share data.

### Physical Volume Storage Locations

- **Linux:** `/var/lib/docker/volumes/<volume-name>/_data`.
- **Windows:** `C:\ProgramData\Docker\volumes\<volume-name>\_data`.
- **macOS:** Docker Desktop runs inside a lightweight virtual machine; the volumes reside within the VM's `/var/lib/docker/volumes/` path rather than directly on the macOS host filesystem.

---

💡 Would you like to turn these structured notes into an interactive quiz or flashcard study set to test your Docker knowledge?