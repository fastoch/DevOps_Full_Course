# 1. Introduction - What are DevOps technologies for?

## What problem did Docker solve?

Docker primarily solves the "it works on my machine" problem in software development by packaging applications with everything required to run them into portable units called "containers".​  

Containers ensure consistent runtime environments across development, testing, and production, eliminating issues from differing OS versions, libraries, or configurations.  

Containerization made possible to run any application the same way regardless of the host Operating System and of its configuration.

## What problem did Kubernetes solve?

Kubernetes solves the challenges of managing containerized applications at scale across multiple servers.  
Its key benefits are:
- scaling (up or down)
- self-healing
- load balancing
- high-availability

---

# 2. Containerization Workflow

Write a Dockerfile > Build a Docker image > Push image to registry > Pull image from registry to create containers that run our application.  

A **Dockerfile** is a set of **instructions** in YAML format, for example:
- use this OS as the base image
- install the following dependencies on that
- copy these files from my local system to that container
- create those volumes for data persistence
- run this command to build the image

A Docker image is the format we use to ship our application to a registry, so that other users or members of our team can then pull that image to 
create containers that run our application. We call such applications "containerized applications".  

The most popular image registry is **Docker Hub**.

---

# 3. Containerizing our first application

- First, cd into the desired directory (create a dedicated one if needed), and pull an already existing app:
```bash
cd ~/Documents/DevOps
mkdir day02_code
cd day02_code
git clone https://github.com/docker/getting-started-app.git
cd getting-started-app
```
This will download the specified repository so we can then dockerize this sample application.  

- Then, we need to create and edit our Dockerfile, while still being in our getting-started-app directory:
```bash
touch Dockerfile
vi Dockerfile
```

Here's what our Dockerfile looks like:
```yaml
FROM node:18-alpine
WORKDIR /app
COPY . .
```

Let's break down what it does:
- first, we specify the base image, the OS on which we'll be installing our app
  - we need a lightweight Linux OS (such as Alpine) with Node.js pre-installed (version 18 in our example)
- the next instruction sets `/app` as the current working directory inside the image, and therefore inside any containers created from it
- finally, we need to copy all our application files to the container, from the current directory in my local filesystem to the current directory in the container (/app)

---

# 8. Kubernetes Networking

## How do Pods simplify container networking in Kubernetes? 

Kubernetes assigns each pod a unique IP address, so pods talk to each other directly using normal IPs instead of host-level port mapping.  

All containers in a pod share the same network **namespace** and IP address, so they communicate with each other via `localhost:<port>`.  

The Kubernetes networking model requires that every pod can reach every other pod in the cluster without NAT (network address translation), across all nodes.   

This flat, NAT-free network greatly simplifies connectivity, discovery, and debugging because source IPs are preserved and packets do not traverse complex translation layers.  

## What problem do Kubernetes Services solve?

Because pods are **ephemeral**, so are their IP address, which is why Kubernetes introduced **Services**, so that pods can be reached through stable virtual IPs.

---

# 10. Why should we use the GitOps approach?

