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

RUN yarn install --production

CMD ["node", "src/index.js"]
```

Let's break down what it does:
- first, we specify the **base image**, the OS on which we'll be installing our app
  - we need a lightweight Linux OS (such as Alpine) with **Node.js** pre-installed (version 18 in our example)
- the next instruction sets `/app` as the **working directory** inside the image, and therefore inside any containers created from it
- then, we COPY all our **application files** to the container, which means:
  - all the contents of the current directory (host file system) is being copied to the container's current directory (/app)
- next, we RUN a command that installs only the **production dependencies** 
  - in this example, the package manager is **yarn**, but there are many other options such as **npm**, **pnpm**, or **Bun**
- the final CMD instruction specifies the default executable when a container starts from the image
  - once our app files have been copied and the dependencies installed, we are ready to start the container that will run our app
  - this last CMD instruction is the one that starts our container, along with the application that will run inside of it

>[!note]
>it is recommended to leave **blank lines** between instructions (FROM, WORKDIR, COPY, RUN, CMD) for improving parsing and readability of our Dockerfile.

When we run the `docker build` command, we build the Docker image of our app thanks to the Dockerfile.  
Then, once the image has been built, we can use it in a `docker run` command to start the container.  

Once we've written our Dockerfile, the workflow with Docker commands is:
- `docker build` to build the image from this Dockerfile
- `docker push` to push the image to a registry
- `docker pull` for other users to pull our image from the registry
- `docker run` to start the container and run the application it contains

Writing the Dockerfile for an application is what allows us to then containerize (or "dockerize") that application via a `docker build` command.  
The containerized version of our application is the image that results of the `docker build` command.

## Exposing our application on a port

The execution of `node src/index.js` will render the pages of our web application.  
In order to access our app while it's running inside a container, we need to expose it on a port.  
Which is why we need to add the following instruction to our Dockerfile:
```yaml
...

CMD ["node", "src/index.js"]

EXPOSE 3000
```

17min Day2/40

---

# 4. 



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

