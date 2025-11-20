

# Docker Image Building & Dockerfile – Detailed Notes


## 1. What is a Docker Image?

A Docker image is a **read-only, layered blueprint** for creating containers.
When you run an image using:

```
docker run <image-name>
```

Docker creates a **container**, which is the runnable instance of that image.

Important points:

* Images are composed of layers.
* Every instruction in a Dockerfile creates a new layer.
* Containers add a thin **writable layer** on top of the image.

---

## 2. What is a Dockerfile?

A **Dockerfile** is a simple text file containing **instructions** that tell Docker how to build an image.

---
# 3. FROM Command

### Purpose

`FROM` defines the **base image** from which your custom image will be built.

### Rules:

* Mandatory. Every valid Dockerfile must start with `FROM`.
* Only exception: `ARG` can appear before it (but rarely needed).
* You can use **multiple FROM** instructions in a single Dockerfile (used in multi-stage builds).

### Example:

```dockerfile
FROM ubuntu:latest
```

### Why base image matters?

Because it gives your image:

* OS environment
* Tools
* Dependencies

### Best practices:

* Use minimal base images (slim, alpine).
* Pin versions (avoid ubuntu:latest).

From PDF reference: 

---

# 4. Building an Image from Dockerfile

### Syntax 1 – When Dockerfile name is custom:

```
docker build -t imagename -f MyDockerfile .
```

### Syntax 2 – When file name is exactly `Dockerfile`:

```
docker build -t imagename .
```

### Important:

* Image name **must be lowercase**.
* `-t` means **tag**, which is the name of your image.
* The last `.` means **build context**.

---

# 5. How Docker Builds Images Internally

When you run:

```
docker build -t myimage .
```

Docker performs these steps:

1. Reads **Dockerfile**.
2. Downloads the required **base image** from Docker Hub (unless already cached).
3. Executes instructions **line by line**.
4. Creates a new **layer** for every instruction.
5. Combines all layers into a final image.

Images are stored under Docker’s internal root directory:

```
/var/lib/docker
```

---

# 6. Docker Commands – Build Time vs Container Run Time

Docker instructions execute in two phases:

## Phase 1: During **image build**

Instructions like:

* FROM
* RUN
* COPY
* ADD
* ENV
* WORKDIR

These define the structure of the image.

## Phase 2: During **container creation / run**

Instructions like:

* CMD
* ENTRYPOINT

These define how the container behaves once started.

Understanding the difference is crucial because **you cannot run runtime commands during build and vice versa**.

---

# 7. RUN Command

**Used during build phase.**

It executes commands inside the image at build time.

Examples:

```dockerfile
RUN apt-get update && apt-get install -y curl
RUN pip install flask
```

Best practices:

* Combine commands to reduce layers.
* Clean cache to reduce image size:

```dockerfile
RUN apt-get update && apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

---

# 8. CMD Command

CMD defines the **default command** that runs when a container starts.

### Examples:

```dockerfile
CMD ["python", "app.py"]
```

### Key rules:

* Only **one CMD** is used — last CMD overrides all previous CMDs.
* CMD **can be overridden** when user runs:

```
docker run myimage ls
```

Here `ls` replaces the CMD.

### FROM PDF: CMD is the default instruction for container startup. 

---

# 9. ENTRYPOINT Command

ENTRYPOINT defines the **main command** that cannot be overridden easily.

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

This will always run, even if user provides arguments.

### Difference vs CMD:

* ENTRYPOINT is fixed.
* CMD is optional and acts as default arguments.

---

# 10. ENTRYPOINT + CMD Together (Very Important Concept)

Example:

```dockerfile
ENTRYPOINT ["echo"]
CMD ["Hello World"]
```

Run container:

```
docker run myimage
```

Output:

```
Hello World
```

Override CMD:

```
docker run myimage Goodbye
```

Output:

```
Goodbye
```

---

# 11. COPY vs ADD Command

| Feature             | COPY             | ADD                |
| ------------------- | ---------------- | ------------------ |
| Copy local files    | Yes              | Yes                |
| Copy from URL       | No               | Yes                |
| Auto-extract tar.gz | No               | Yes                |
| Recommended for     | Normal file copy | Special cases only |

### COPY Example:

```dockerfile
COPY requirements.txt /app/
COPY . /app
```

### ADD Example:

```dockerfile
ADD https://example.com/file.tar.gz /tmp/
ADD app.tar.gz /app/
```

Best practice:

* **Use COPY** unless you specifically need URL download or tar extraction.

From PDF: COPY is predictable, ADD has extra features. 

---

# 12. Example Commands Used in Class

```bash
sudo su
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

git clone https://github.com/vilasvarghesescaler/docker-k8s/

cd docker-k8s/dockerfiles
cat Dockerfile    # Shows: FROM ubuntu:latest

docker build -t myfrom .
docker images
docker run -it myfrom
docker build -t myrun -f 2.Runfile .
docker images
```

---

# 13. Complete Example Dockerfile Breakdown (From PDF)

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Explanation:

* Base image is lightweight.
* Dependencies installed early for cache efficiency.
* ENTRYPOINT always runs python.
* CMD passes "app.py" as default argument.

From PDF reference: 

---

# 14. How to Inspect Images

### View layers:

```
docker history <image>
```

### Inspect metadata:

```
docker inspect <image>
```

### Scan for vulnerabilities:

```
docker scan <image>
```

---





# 15. Final Summary (Core Understanding)

1. **FROM** → Sets base image. Mandatory.
2. **RUN** → Executes commands during build (install packages etc.).
3. **COPY/ADD** → Copies files into image.
4. **CMD** → Default runtime command (can be overridden).
5. **ENTRYPOINT** → Main executable (cannot be overridden easily).
6. **ENTRYPOINT + CMD** → ENTRYPOINT is command, CMD is arguments.
7. **docker build** → Builds the image.
8. **docker run** → Creates and runs container.


