# Lab 10: Persisting Data with Volumes

## Objectives
By the end of this lab, you will be able to:

- Configure persistent storage for containers using volumes
- Create and manage named volumes
- Mount volumes in containers
- Use bind mounts to persist data using host directories

## Prerequisites
- Basic understanding of command line operations
- Podman installed on your system (version 3.0+ recommended)
- A Linux-based system (recommended) or WSL2 on Windows
- Rootless Podman configured (optional but recommended for security)

---

## Lab Tasks

### Task 1: Creating Named Volumes
Named volumes are managed by Podman and provide persistent storage that survives container removal.

#### Subtask 1.1: Create a Named Volume
```bash
podman volume create myapp_data
````

**Expected Output:**

```
myapp_data
```

**Verification:**

```bash
podman volume ls
```

You should see `myapp_data` in the list.

#### Subtask 1.2: Inspect the Volume

```bash
podman volume inspect myapp_data
```

**Expected Output:**

```json
[
    {
        "Name": "myapp_data",
        "Driver": "local",
        "Mountpoint": "/var/lib/containers/storage/volumes/myapp_data/_data",
        "CreatedAt": "2023-10-05T12:00:00Z",
        "Labels": {},
        "Scope": "local"
    }
]
```

**Key Concept:**
The **Mountpoint** shows where the volume data is stored on the host.

---

### Task 2: Mounting Volumes in Containers

Volumes can be attached to containers to persist data.

#### Subtask 2.1: Run a Container with a Volume

```bash
podman run -d --name webapp -v myapp_data:/var/www/html docker.io/library/nginx
```

**Explanation:**
`-v myapp_data:/var/www/html` mounts the volume `myapp_data` to `/var/www/html` in the container.

**Verification:**

```bash
podman exec webapp ls /var/www/html
```

(No output expected since the volume is empty.)

#### Subtask 2.2: Persist Data in the Volume

```bash
podman exec webapp sh -c "echo 'Hello, Volume!' > /var/www/html/index.html"
podman exec webapp cat /var/www/html/index.html
```

**Expected Output:**

```
Hello, Volume!
```

#### Subtask 2.3: Verify Data Persistence

Remove the container:

```bash
podman rm -f webapp
```

Recreate the container:

```bash
podman run -d --name webapp_new -v myapp_data:/var/www/html docker.io/library/nginx
```

Check the persisted data:

```bash
podman exec webapp_new cat /var/www/html/index.html
```

**Expected Output:**

```
Hello, Volume!
```

**Key Concept:**
The data persists even after container removal because it is stored in the named volume.

---

### Task 3: Using Bind Mounts with Host Directories

Bind mounts link a directory directly into a container.

#### Subtask 3.1: Create a Host Directory

```bash
mkdir ~/host_data
echo "Hello, Bind Mount!" > ~/host_data/index.html
```

#### Subtask 3.2: Run a Container with a Bind Mount

```bash
podman run -d --name bind_example -v ~/host_data:/usr/share/nginx/html:Z docker.io/library/nginx
```

**Explanation:**

* `-v ~/host_data:/usr/share/nginx/html` binds the host directory to the container.
* `:Z` ensures proper SELinux labeling (required on systems with SELinux enabled).

**Verification:**

```bash
podman exec bind_example cat /usr/share/nginx/html/index.html
```

**Expected Output:**

```
Hello, Bind Mount!
```

#### Subtask 3.3: Modify Host Data and Verify

Edit the host file:

```bash
echo "Updated content!" >> ~/host_data/index.html
```

Check the container:

```bash
podman exec bind_example cat /usr/share/nginx/html/index.html
```

**Expected Output:**

```
Hello, Bind Mount!
Updated content!
```

**Key Concept:**
Changes on the host are immediately reflected in the container.

---

## Troubleshooting Tips

* **Permission Issues:** Use `:Z` or `:z` for SELinux contexts if access is denied.
* **Volume Not Found:** Ensure the volume exists (`podman volume ls`).
* **Bind Mount Paths:** Use absolute paths for host directories.

---

## Conclusion

In this lab, you learned how to:

* Create and manage named volumes for persistent storage.
* Mount volumes in containers to retain data.
* Use bind mounts to share host directories with containers.

These skills are essential for managing stateful applications in containers, ensuring data persists across container lifecycles.

---

## Next Steps

* Explore `podman volume prune` to clean up unused volumes.
* Practice using volumes with databases like PostgreSQL or MySQL.
* Learn about volume drivers for network storage (e.g., NFS).