Here’s a clean **`README.md`** file for your lab:
# Lab 11: Running Stateful Containers

This lab demonstrates how to run stateful containers with persistent storage using **Podman** (or Docker). You will set up **MySQL** and optionally **PostgreSQL** with volume mounts, verify that data survives container removal, and understand storage concepts for stateful applications.

---

## Objectives
- Use persistent storage for stateful containers  
- Run **MySQL/PostgreSQL** containers with mounted volumes  
- Verify data persistence after container restarts or removal  

---

## Prerequisites
- Podman or Docker installed (**Podman recommended**)  
- Basic command-line skills  
- 2GB+ free disk space  
- Internet connection to pull container images  

---

## Lab Setup

1. Verify Podman installation:
   ```bash
   podman --version
````

**Expected Output:** A version number (e.g., `4.0.0`)

2. Create a working directory:

   ```bash
   mkdir -p ~/stateful-lab && cd ~/stateful-lab
   ```

---

## Task 1: Running MySQL with Persistent Storage

### Step 1.1: Create Persistent Volume

```bash
mkdir -p mysql-data
```

### Step 1.2: Start MySQL Container

```bash
podman run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=redhat123 \
  -e MYSQL_DATABASE=testdb \
  -e MYSQL_USER=testuser \
  -e MYSQL_PASSWORD=user123 \
  -v $(pwd)/mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  docker.io/library/mysql:8.0
```

Verify container:

```bash
podman ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"
```

### Step 1.3: Test Database

```bash
podman exec -it mysql-db mysql -u testuser -puser123 testdb
```

Inside MySQL shell:

```sql
CREATE TABLE lab_data (id INT AUTO_INCREMENT PRIMARY KEY, message VARCHAR(255));
INSERT INTO lab_data (message) VALUES ('Persistent test data');
SELECT * FROM lab_data;
exit;
```

---

## Task 2: Verify Data Persistence

### Step 2.1: Remove Container

```bash
podman stop mysql-db
podman rm mysql-db
```

### Step 2.2: Recreate Container

```bash
podman run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=redhat123 \
  -v $(pwd)/mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  docker.io/library/mysql:8.0
```

### Step 2.3: Verify Data

```bash
podman exec -it mysql-db mysql -u testuser -puser123 testdb -e "SELECT * FROM lab_data;"
```

---

## Task 3: PostgreSQL (Optional)

### Step 3.1: Create Persistent Volume

```bash
mkdir -p pg-data
```

### Step 3.2: Start PostgreSQL Container

```bash
podman run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=redhat123 \
  -e POSTGRES_USER=testuser \
  -e POSTGRES_DB=testdb \
  -v $(pwd)/pg-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  docker.io/library/postgres:13
```

### Step 3.3: Test Persistence

```bash
podman exec -it postgres-db psql -U testuser -d testdb -c "CREATE TABLE lab_data (id SERIAL PRIMARY KEY, message TEXT); INSERT INTO lab_data (message) VALUES ('Postgres persistent data');"
```

Follow the same stop/remove/recreate steps to confirm persistence.

---

## Troubleshooting Tips

* **Permissions:**

  ```bash
  sudo chown -R 1001:1001 mysql-data/
  ```

* **Port Conflicts:**

  ```bash
  ss -tulnp | grep 3306
  ```

* **Logs:**

  ```bash
  podman logs mysql-db
  ```

---

## Conclusion

✅ You have successfully:

* Implemented persistent storage with MySQL and PostgreSQL
* Verified that data survives container removal
* Learned volume mounting for stateful applications

---

## Knowledge Check

1. What happens if you don’t use the `-v` flag when starting a container?
2. How can you move persistent data to another host?
3. What are the security concerns when storing database files on the host?

---

## Further Exploration

* Try **named volumes** (`podman volume create`)
* Set up **multi-container applications** with persistence
* Explore **backup strategies** for containerized databases