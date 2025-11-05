#  Microservices and Related Concepts

##  What are Microservices?

**Microservices** is an architectural style where a large application is broken down into **small, independent, and loosely coupled services**.  
Each service focuses on a **specific functionality** and can be **developed, deployed, and scaled independently**.

###  What is a Monolithic Architecture?
- A **monolithic application** is one where the **entire codebase** (frontend, backend, business logic, and database) is written and deployed as **a single unit**.
- If one part fails or needs updating, the entire application has to be redeployed.

###  From Monolithic to Microservices
- In microservices, we **decompose** a large monolithic system into smaller, modular applications (microservices).
- Each service handles **one business capability** and communicates with other services through **APIs** (usually REST or gRPC).

**Example:**  
In an e-commerce app:
- `User-Service` handles user data.  
- `Order-Service` manages orders.  
- `Payment-Service` handles payments.  

Each of these runs independently but communicates using APIs.

---

##  DNS – Domain Name System

###  What is DNS?
**DNS (Domain Name System)** is the **phonebook of the Internet**.  
It translates **domain names** (like `google.com`) into **IP addresses** (like `142.250.183.68`) that computers use to locate each other on a network.

###  Why We Need DNS
Without DNS, we would have to remember and type IP addresses for every website — which is impractical.

**DNS provides:**
- **User-friendly addressing** — Domains instead of numbers.  
- **Scalability** — Supports millions of websites.  
- **Load balancing and redundancy** — Distributes users across multiple servers.  
- **Flexibility** — Servers can be changed without affecting domain names.

**In simple terms:**  
You remember **names**, computers use **numbers**, and **DNS connects the two**.

---

##  Load Balancer

After DNS resolves the domain name to an IP, the request usually goes to a **Load Balancer**.

###  Responsibility
The **Load Balancer** distributes incoming traffic evenly across multiple servers or service instances to ensure:
- High availability
- Better performance
- No single point of failure

###  Types of Load Balancing
1. **Path-based Routing** — Routes requests based on URL paths  
   Example:  
   `/api/users → User-Service`  
   `/api/orders → Order-Service`
2. **Instance-based Routing** — Distributes requests among multiple instances of the same service.

---

##  Authentication vs Authorization

| Concept | Description | Example |
|----------|--------------|----------|
| **Authentication** | Confirms **who you are**. | Logging in with username & password. |
| **Authorization** | Determines **what you can do**. | Accessing your own inbox but not others’. |

**Authentication** verifies identity.  
**Authorization** defines permissions.

Common methods:
- **ACL (Access Control Lists)**
- **RBAC (Role-Based Access Control)**
- **Permission policies**

>  You must be **authenticated** before you can be **authorized**.

---

##  Database per Service Architecture

- Each microservice should have **its own database**.
- One service **can talk to multiple databases**,  
  but **multiple services must not share one database**.

**Reason:**  
This maintains **loose coupling**, **data ownership**, and **independent scaling**.

---

##  Service Discovery

###  What is Service Discovery?
**Service Discovery** is the process by which microservices **automatically find and connect** to each other without hardcoding IP addresses or ports.

###  Why We Need It
In dynamic environments (like containers or Kubernetes):
- Services **start/stop frequently**
- IP addresses **change often**

So, instead of hardcoding, each service **registers itself** with a **Service Registry**.

###  How It Works
1. **Service Registration:**  
   When a service starts, it registers with the registry (e.g., “I’m User-Service on 10.0.2.15:8080”).
2. **Service Lookup:**  
   When another service needs it, it queries the registry.
3. **Communication:**  
   The client connects using the registry’s response.

###  Common Service Discovery Tools
- **Consul (HashiCorp)**
- **Eureka (Netflix)**
- **etcd (CoreOS)**
- **Zookeeper (Apache)**
- **Kubernetes DNS-based Discovery**

---

## Microservice-to-Microservice Communication

Microservices communicate in two ways:

###  Synchronous Communication
- One service calls another and **waits for a response**.
- Works like a **phone call** — you talk and wait for the reply.

**Examples:** REST, gRPC, GraphQL  
**Use case:** Real-time operations like authentication or order confirmation.

**Pros:**
- Simple to implement  
- Immediate response  

**Cons:**
- Tightly coupled  
- If one service fails, others can be affected  

---

###   Asynchronous Communication
- Services **don’t wait** for a response.
- Messages are sent via a **message broker** (like Kafka or RabbitMQ).
- Works like **sending a text message** — you send and move on.

**Examples:** Kafka, RabbitMQ, AWS SQS/SNS  
**Use case:** Notifications, order updates, background jobs.

**Pros:**
- Loose coupling  
- High scalability  
- Failure-tolerant  

**Cons:**
- Complex to manage  
- No immediate feedback  

---

###  Types of Asynchronous Models
1. **Queue Model** — Messages are stored in a queue and processed by one consumer at a time.  
   (Example: RabbitMQ)
2. **Publisher-Subscriber Model** — Messages are broadcast to multiple subscribers.  
   (Example: Kafka, Google Pub/Sub)

> Note: In asynchronous communication, we **don’t need load balancers** — the **message broker handles distribution**.

---

##  Event Sinking (Event Logging / Tracking)

**Event Sinking** refers to **collecting and recording details** about events happening in your system — such as:
- Which service instance handled the request
- How long it took
- Whether it succeeded or failed

These logs help in:
- Monitoring performance
- Debugging issues
- Auditing service activity

Tools: **ELK Stack (Elasticsearch, Logstash, Kibana)**, **Prometheus**, **Grafana**

---

##  The 12-Factor App

The **12-Factor App** is a set of **best practices** for building **cloud-native applications** that are scalable, maintainable, and portable.  
It was created by **Heroku engineers**.

###  The 12 Factors

1. **Codebase** — One codebase tracked in version control (Git), many deploys.  
2. **Dependencies** — Explicitly declare and isolate dependencies.  
3. **Config** — Store configuration in environment variables, not in code.  
4. **Backing Services** — Treat backing services (DB, queues) as attached resources.  
5. **Build, Release, Run** — Separate build, release, and run stages.  
6. **Processes** — Run app as one or more stateless processes.  
7. **Port Binding** — Export services via port binding (e.g., run on `localhost:5000`).  
8. **Concurrency** — Scale out by running multiple processes.  
9. **Disposability** — Fast startup and graceful shutdown for quick scaling.  
10. **Dev/Prod Parity** — Keep development and production environments as similar as possible.  
11. **Logs** — Treat logs as event streams, not files.  
12. **Admin Processes** — Run admin tasks (e.g., migrations) as one-off processes.

>  Following these principles helps create **scalable, maintainable, and cloud-ready microservices.**

---

