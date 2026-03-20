# Containerization

## 1. Introduction to Containerization

Containerization is an Operating System (OS) level virtualization technique that leverages multiprogramming features of Unix/Linux systems.

- **Containers vs. Virtual Machines:** Unlike system virtualization, containers do not require a hypervisor or a full replica of a guest operating system. Instead, they share the same host OS kernel and use a container manager to isolate applications, making them much more lightweight and less resource-consuming.
	
	![](../../Pasted%20image%2020260320122059.png)

- **Types of Containers:**
    - **Application Containers:** Designed to distribute and run a single application or microservice, using a layered file system.
    - **System Containers:** Function like lightweight virtual machines containing an entire OS, providing the underlying infrastructure to run auxiliary services (e.g., LXC/LXD).

## 2. Fundamental Linux Technologies Behind Containers

Containers rely on three main Linux kernel features to function, evolving from the older `chroot` mechanism (which isolated a process by changing its root directory).

### A. Namespaces ("What you can see")
Namespaces wrap a global system resource in an abstraction, isolating processes so they believe they have their own dedicated instance of that resource.

- Changes made within a namespace are only visible to the processes that are members of that namespace, isolating them from the rest of the system.
- Namespaces can isolate various resources: process IDs (PID), network devices, mount points, users/groups, and Inter-Process Communication (IPC).
- They are managed using system calls like `clone()` (to create a process in a new or existing namespace), `setns()`, and `unshare()`.

### B. Cgroups (Control Groups) ("How much you can use")
While namespaces control what a process sees, Cgroups control how much of the system's resources a process can consume, preventing a single container from monopolizing the host.

- They allow processes to be organized hierarchically and provide resource limiting, prioritization, accounting, and controlling.
- Cgroups can limit CPU time, memory usage, I/O bandwidth, and PIDs. They are managed through a pseudo-filesystem called `cgroupfs`.

### C. Union File System (UnionFS)
UnionFS allows administrators to logically merge multiple separate physical directories (called branches) into a single unified view.

- **Precedence:** If two files have the same name in different branches, the file in the branch with higher precedence overrides the lower one.
- **Copy-on-Write (CoW):** UnionFS mixes read-only and read-write branches. If a process tries to modify a file from a read-only branch, UnionFS automatically copies the file to a higher-priority read-write branch, modifies the copy, and hides the original. This is the foundational concept behind Docker image layers.

## 3. Docker Architecture

Docker is a Client/Server application designed to create and manage containers.

- **Components:** It consists of a **Client** (Command Line Interface) that interacts with the **Docker Daemon** (server) via a REST API.
- **Registry:** A remote storage location (like Docker Hub) where images can be stored and downloaded by the daemon.
- **Images vs. Containers:**
    - **Image:** A read-only template built from instructions in a Dockerfile. It is made of multiple read-only layers stacked via UnionFS.
    - **Container:** A runnable instance of an image. When an image is run, Docker adds a thin, ephemeral **read/write layer** on top of the image stack.

## 4. Docker Storage Options

By default, data created inside a container is stored in its ephemeral read/write layer. When the container stops, this data is lost. To achieve data persistence, Docker provides three options:

1. **Volumes:** The best and most secure option for persistent data.
    - Created and entirely managed by Docker, stored in a dedicated host directory isolated from other processes.
    - Highly portable, easy to back up, and can be safely shared across multiple containers.
2. **Bind Mounts:** Mounts a specific, user-defined file or directory from the host machine directly into the container.
    - Offers high performance but relies on the host's specific file system structure, making it **less portable**.
    - Often used in development environments to share source code between the host and the container to instantly see changes.
3. **Tmpfs Mount (Temporary File System):** Stores data purely in the host's RAM memory, outside the container's writable layer.
    - Provides extremely high performance but data is completely lost when the container stops.
    - Available only on Linux hosts and cannot be shared among different containers. Useful for caching or securely handling non-persistent sensitive data.

## 4. Docker Network Drivers

Docker provides flexible networking to connect containers to each other or to non-Docker workloads:

- **Bridge (Default):** Used to connect standalone containers running on the same Docker host.
- **Host:** Removes all network isolation, allowing the container to use the host machine's networking stack directly for maximum performance.
- **Overlay:** Connects multiple Docker daemons running on different hosts, enabling distributed containers (e.g., in Docker Swarm) to communicate seamlessly without manual OS-level routing.
- **Macvlan:** Assigns a unique MAC address to a container, making it appear as a physical device on the local network. This is ideal for legacy applications or migrating traditional VMs.
- **None:** Completely disables networking for the container.