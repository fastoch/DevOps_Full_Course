# 1. Introduction - Why were DevOps technologies invented?

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

## What problem do pods solve?

---

# 2. Containerization Workflow

Write a Dockerfile > Build a Docker image > Push image to registry > Pull image from registry to create containers that run our application.  

A dockerfile is a set of instructions in YAML format, for example:
- use this OS as the base image
- install the following dependencies on that
- copy these files from my local system to that container
- create those volumes for data persistence
- run this command to build the image

The image is the format we use to ship our application to a registry, so that other users or members of our team can then pull that image to 
create containers that run our application. We call such applications "containerized applications".

---

# 3. Kubernetes Networking



# Why should we use the GitOps approach?

