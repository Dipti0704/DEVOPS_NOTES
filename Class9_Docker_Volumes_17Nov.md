# 1. Why Do We Need Docker Volumes?

Containers are ephemeral.
By default:

* A container has a **read-only image**
* Plus a **temporary writable layer**
* When a container is stopped and removed, the writable layer is destroyed

This means any data stored inside paths like:

```
/var/lib/mysql
/usr/share/nginx/html
/app/logs
```

is lost after a container is removed.


```
docker run -it ubuntu bash
# install java
# mkdir test

docker stop <id>
docker start <id>
```

If you only stop and start the container, the writable layer is preserved.
But if the container is **removed**, then the writable layer is destroyed.

To persist data beyond the lifecycle of a container, Docker provides **Volumes**.

---

# 2. How Container Storage Actually Works

When you run a container:

* Docker takes the image layers (read-only layers)
* Adds a **writable layer** on top
* Any file changes inside the container are stored in this writable layer

This writable layer is deleted when the container is removed.


---

# 3. Container Data Persistence Misconception


> If we stop and start a container, do we lose data?
> No. The writable layer survives a stop/start.
> We lose data only when the container itself is **removed**.

Example:

```
docker stop <id>
docker start <id>
```

Data remains because the container object still exists.

But if we run:

```
docker rm <id>
```

The entire writable layer is deleted.

---

# 4. What Exactly Is a Docker Volume?

A Docker volume is a **special filesystem location managed by Docker**.
It exists **outside the container writable layer**, which means:

* It persists even if containers are recreated
* It can be shared between multiple containers
* It is stored on the host at `/var/lib/docker/volumes`


---

# 5. Types of Volumes in Docker

 Docker supports **four types**:

1. Anonymous Volumes
2. Named Volumes
3. Bind Mounts
4. tmpfs Mounts (memory-only)

---

# 6. Anonymous Volumes

Created automatically when you mount a directory without specifying a name.

Example:

```
docker run -d -v /data nginx
```

Characteristics:

* Random volume name created (e.g., `d3c9248bc5a8f9301a6e`)
* Exists until the container is removed
* Good for temporary storage


List volumes:

```
docker volume ls
```

---

# 7. Named Volumes

These are explicitly created by the user and managed by Docker.

Create a volume:

```
docker volume create scalervolume
```

Inspect:

```
docker volume inspect scalervolume
```

Use it in a container:

```
docker run -it -v scalervolume:/nov17 ubuntu bash
```

Data survives even if you delete the container:

```
docker rm -f ubuntu_container
docker run -it -v scalervolume:/nov17 ubuntu bash
```

The directory `/nov17` will still contain all previous data.

Named volumes are most commonly used for:

* Databases (MySQL, MongoDB)
* Upload directories
* Persistent application storage


---

# 8. Bind Mounts


> What we saw till now is called bind mounts.
> In this we can attach any folder on the host machine into the container.
> That is not so good because critical folders in Linux can also get attached.

Bind mounts map a **host directory** directly into a container.

Format:

```
-v /host/path:/container/path
```

Example:

```
docker run -d \
  -v $(pwd)/website:/usr/share/nginx/html \
  -p 8080:80 \
  nginx
```

Characteristics:

* Host directory controls the data
* Immediate reflection of changes (useful for development)
* Risky in production because full host access is possible


---

# 9. tmpfs Mounts (RAM Only)

tmpfs mounts store data **only in RAM**.

Example:

```
docker run -d --tmpfs /cache nginx
```

Or:

```
docker run -d --mount type=tmpfs,target=/cache nginx
```

Characteristics:

* Data never hits disk
* Very fast
* Data disappears on container stop
* Used for temporary files, sensitive data, caches

PDF Reference → 

---

# 10. Summary Table (From PDF)

| Type       | Persistent | Stored Where                   | Use Case                      |
| ---------- | ---------- | ------------------------------ | ----------------------------- |
| Anonymous  | Yes        | /var/lib/docker/volumes/...    | Temporary container storage   |
| Named      | Yes        | /var/lib/docker/volumes/<name> | Databases, persistent storage |
| Bind Mount | Host path  | Host filesystem                | Development, logs, configs    |
| tmpfs      | No         | RAM                            | Sensitive temp data, caching  |


---

# 11. Where Docker Stores Volumes

All Docker-managed volumes are stored at:

```
/var/lib/docker/volumes/
```

Bind mounts are stored on the host at the exact directory you specify.


---

# 12. Real World Use Cases

### Named Volumes

* MySQL, PostgreSQL, MongoDB
* Redis
* Persistent upload folders
* Application-level storage

### Bind Mounts

* Development environments (hot reload)
* Host log directories
* Config injection
* SSL certificates

### tmpfs

* Session caches
* Temporary secrets
* High-speed operations



---

# 13. Demonstrating Volume Persistence 

You created a container, created folders, stopped and started it.

Example:

```
docker run -it ubuntu bash
mkdir test
cd test
echo "Hello" > test.txt
```

Stop:

```
docker stop <id>
```

Start:

```
docker start <id>
docker exec -it <id> bash
```

Data persists because the container is not removed.

Now attach a volume:

```
docker run -it -v scalervolume:/nov17 ubuntu bash
```

Anything written inside `/nov17` is persisted even if the container is removed.

---

# 14. Container-to-Container Real-Time Communication

Your handwritten note references the diagram:


This shows two containers reading/writing to the same mounted volume, enabling real-time sharing of files.

This behavior occurs when:

* Two containers mount the **same volume**
* Both use the same mount point

Example:

```
docker run -it -v scalervolume:/data ubuntu bash
docker run -it -v scalervolume:/data ubuntu bash
```

Both containers now see shared data.

---

# 15. Complete Examples Using Each Volume

### 1. Anonymous Volume

```
docker run -d -v /data nginx
```

### 2. Named Volume

```
docker volume create mydb
docker run -d -v mydb:/var/lib/mysql mysql:8
```

### 3. Bind Mount

```
docker run -d -v $(pwd)/website:/usr/share/nginx/html -p 8080:80 nginx
```

### 4. tmpfs Mount

```
docker run -d --tmpfs /secret busybox sh -c "echo hi > /secret/msg && sleep 1000"
```


---

# 16. Final Concept Summary

1. Containers are ephemeral because the writable layer is temporary.
2. Stopping a container does not remove data; removing it does.
3. Volumes store data outside the container lifecycle.
4. There are four volume types: anonymous, named, bind mount, tmpfs.
5. Named volumes are best for production.
6. Bind mounts are best for development.
7. tmpfs is best for sensitive or temporary files.
8. Volumes are stored under `/var/lib/docker/volumes`.

---

