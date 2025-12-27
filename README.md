# 0. Source material 

- https://youtube.com/playlist?list=PLl4APkPHzsUUOkOv3i62UidrLmSB8DcGC&si=iOFr4S3hK4OpuPtq
- https://kind.sigs.k8s.io/docs/user/quick-start/
- https://kubernetes.io/docs/home/
- https://kubernetes.io/blog/

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

## Creating our first Dockerfile

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

## Explaining the Dockerfile's role

When we run the `docker build` command, we build the Docker image of our app thanks to the Dockerfile.  
Then, once the image has been built, we can use it in a `docker run` command to start the container.  

Once we've written our Dockerfile, the workflow with Docker commands is:
- `docker build` to build the image from this Dockerfile
- `docker push` to push the image to a registry
- `docker pull` for other users to pull our image from the registry
- `docker run` to start the container and run the application it contains

Writing the Dockerfile for an application is what allows us to then containerize (or "dockerize") that application via a `docker build` command.  
The containerized version of our application is the image that results of the `docker build` command.  

But something is missing in our Dockerfile...

## Exposing our application on a port

The execution of `node src/index.js` will render the pages of our web application.  
In order to access our app while it's running inside a container, we need to EXPOSE it on a port.  
Which is why we need to add the following instruction to our Dockerfile:
```yaml
...

CMD ["node", "src/index.js"]

EXPOSE 3000
```
Now we are ready to build the Docker image from our Dockerfile.

## Building our first Docker image

Place yourself in the directory that contains the Dockerfile and the app files, then run:
```bash
docker build -t day02-image .
```
- The `-t` option is to tag the image, to give it a name
- The final dot means "use the Dockerfile inside the current directory"

>[!important]
The above command might fail if the `docker-credential-secretservice` binary is missing.  
In such case, we need to download and install this binary:
- first, install the `libsecret` library: `sudo dnf install libsecret`
- check for the latest version of this binary: https://github.com/docker/docker-credential-helpers/releases
- Then, download them:
```bash
wget https://github.com/docker/docker-credential-helpers/releases/download/v0.9.4/docker-credential-pass-v0.9.4.linux-amd64 -O /tmp/docker-credential-secretservice`
```
- make it executable: `chmod +x /tmp/docker-credential-secretservice`
- move it to where it belongs: `sudo mv /tmp/docker-credential-secretservice /usr/local/bin/`
- test the binary: `docker-credential-secretservice version`
- Finally, you can re-run the `docker build` command

Once the Docker image has been built, you can see it's been added to your local registry by running `docker images`.  

## Pushing the image to an online registry

- go to https://hub.docker.com
- create an account if needed
- log in to your DockerHub account
- create a repository where we'll push our image
- now we can push our image to the DockerHub repo:
```bash
docker tag day02-image:latest fastoch/cka-full-course-day02:latest
docker login -u fastoch
docker push fastoch/cka-full-course-day02:latest
```
The first line creates a local replica of our image. We can see that by running `docker images`.  
The second line is to authenticate to my DockerHub account.  
The last line pushes this replica to our DockerHub registry.  

>[!important]
You might need to install and initialize `pass`, a program that handles authentication for commands such as `docker login` or `nerdctl login`.
For more details: https://docs.rancherdesktop.io/getting-started/installation/#pass-setup
-  once you've installed pass, create a GPG key via `gpg --generate-key`. This will be used by `pass` to secure secrets.
-  save your passphrase for this GPG key in a password manager (other than pass...)
-  The output of `gpg --generate-key` will contain your key ID.
-  You can initialize `pass` by passing this key ID to it: `pass init <GPG_key_ID>`
-  then, you'll be able to run `docker login` commands to authenticate from the CLI

Once you've verified that your Docker image has been successfully pushed to DockerHub, you can delete the original image:
```bash
docker rmi day02-image
```

And now, our image can be pulled from DockerHub by anyone, given we've published it to a public registry.  
If we made our registry private, any member of our team with access to this registry can pull our image.  

>[!note]
Nerdctl is a Docker-compatible command-line interface (CLI) for containerd.
>It provides familiar Docker-like commands while leveraging containerd's runtime for container management.

## Running a container from our image

```bash
docker run -d -p 3000:3000 fastoch/cka-full-course-day02:latest
```
- the `-d` option is for running the container in detached mode, so that we can keep using the same terminal window
- the `-p` option is to map one of the host ports to the container's port which our app is exposed at 

Now, our app can be accessed directly from the host by opening a web browser and going to http://localhost:3000/  
This is a simple Todo app.  

To see our container running: `docker ps`  
Notice that it's been assigned a random name.  
If we wanted to give our container a specific name, we should have use the `--name` option.  

And now, the image we've built is available for other users and can be used to run exactly the same app on any computer.  

---

# 4. Entering a container

To enter the container we've just created:
```bash
docker exec -it <container_name> sh
```
- `-it` is for "interactive terminal"
- `sh` is the shell we'll be using while inside the container, we could use bash depending on the binaries available inside that container.

Once inside the container, we can run `cat /etc/os-release` to show the OS it's running on.  
To exit our container, we just have to run `exit`.  

---

# 5. Multi-stage Docker Build

Multi-stage build is something we use for mutliple reasons:
- to reduce build time and the image size
- to improve the performance of Docker containers
- to enforce best practices

## Cloning the app

Let's create a new directory and clone the application we'll use for this example:
```bash
cd ~/Documents/DevOps
mkdir day03_code
cd day03_code
git clone https://github.com/piyushsachdeva/todoapp-docker
cd todoapp-docker
touch Dockerfile
vi Dockerfile
```

Our Dockerfile will now look like this:
```yaml
FROM node:18-alpine AS installer

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

FROM nginx:latest AS deployer

COPY --from=installer /app/build /usr/share/nginx/html
```

## Explaining the new version of our Dockerfile

This multi-stage Dockerfile builds a Node.js app in a first stage, and serves the output via Nginx in a second stage, leveraging Docker's layer caching for faster rebuilds.  

Each instruction creates a cacheable layer, and Docker skips re-execution if inputs match previous builds.  

Let's explain these instructions in details:
- first line starts a lightweight Node.js base image and names the stage "installer" for later reference
- second line sets the working directory
- third line copies only package files first; this layer caches if `package.json/package-lock.json` remain unchanged,
  enabling `RUN npm install` (next layer) to reuse the "node_modules" folder without reinstalling dependencies
- fourth line only runs if changes occur in the package files 
- fifth line copies all source code, invalidating prior caches if files change
- `RUN npm run build` then compiles to /app/build/, creating the final artifacts
- the next line begins the "deployer" stage, creating a minimal web server image, and discarding the "installer" stage
- the final line copies the built static files from stage 1 into Nginx's default directory; no further instructions means Nginx serves on port 80

## Leveraging Docker's "layer caching"

Docker images consist of stacked layers, where each instruction (like FROM, RUN, or COPY) generates a new layer on top of the prior ones.
Docker's layer caching speeds up image builds by reusing unchanged layers from previous builds.  

During `docker build`, Docker scans instructions sequentially:
- if a layer matches an existing cached version exactly, it reuses it instantly
- A change in any instruction invalidates that layer and all subsequent ones, forcing rebuilds from that point

>[!tip]
>Order instructions to place rarely changing steps first, like copying dependency files before the application source code.

## Why should we use multi-stage for building our Docker images?

Multi-stage builds separate build-time dependencies (Node/npm) from runtime needs (Nginx), reducing image size, improving security, and speeding up deployments.
What results of the first stage is not included in the final image.  

Combining Docker's layer caching capability and Multi-stage image building allows us to optimize our image size and to deploy our application faster.  

## Rebuilding an image with our new Dockerfile

use the `cd` command to move to the directory that contains the Dockerfile and the Todo app, then run:
```bash
docker build -t multi-stage .
```
You might need to enter the passphrase you've set up for the GPG key you've used to initialize `pass`.  
Reminder: `pass` is the binary required to use `docker login` or `nerdctl login`.  

## Running a container from our new image

- To run a container from our image: `docker run -dp 3000:80 multi-stage:latest`
- to get the container name: `docker ps`
- to enter the container: `docker exec -it <container_name> sh`
- To remove a specific container: `docker rm <container_name>`
- to troubleshoot, we can use commands such as `docker logs` or `docker inspect`

---

# 6. Why do we need K8s?

Managing a lot of containers running across multiple machines is too much of a hassle without Kubernetes.  
Kubernetes handles the following for us:
- Container Networking
- Resource Management
- Security
- High Availability
- Fault Tolerance
- Service Discovery
- Scalability
- Load Balancing

Having to allocate human resources to manually take care of the above would be expensive and much less efficient.  

Of course, in many simple use cases, Kubernetes is not needed.  
For example, **Docker Swarm** suits small-to-medium deployments due to its simplicity and low overhead compared to Kubernetes.  

But if you're running a complex application which is used by many users across multiple countries, then you don't have a choice.  

---

# 7. What is K8s? Kubernetes architecture explained

## Overview

- **Node**: generally a VM, but it can be any computing device
- **Control Plane**: node that provides instructions to worker nodes
- **Worker nodes**: nodes which our containers will run on

## What is a pod?

The Kubernetes pod **abstraction** was invented to group one or more tightly coupled containers into a single, atomic scheduling unit, 
enabling them to share resources like **network namespace**, **storage** volumes, and **lifecycle** while simplifying orchestration.

- A pod is the **smallest deployable unit** in K8s
- Pods enable **localhost** communication between the containers they encapsulate
- Pods are designed to be **ephemeral**

Since each pod has a unique IP address, each container inside of it is assigned a specific port, and containers within a pod can talk via **localhost:<port_number>**.  

If a pod is killed and replaced by a new one, the new pod gets a different IP address, which is why K8s services were invented.   
A K8s service provides a pod with a stable IP address and a DNS name.  

## What is kubectl?

Developers and Kubernetes admins use `kubectl` for deployment, scaling, troubleshooting, and multi-cluster management via **context switching**.  
More on context switching later on...

`kubectl` is the command-line interface (CLI) tool for interacting with Kubernetes clusters.  
It communicates with the Kubernetes API server to perform operations like deploying applications, inspecting resources, and managing cluster state.   
`kubectl` enables CRUD operations (create, read, update, delete) on Kubernetes resources such as pods, deployments, services, and nodes.  

## Control Plane Components

### API server

Also known as `kube-apiserver`.  
This is the **communication center** of your control plane, and therefore the main entry point of your K8s cluster.  
Any incoming request to the control plane goes through the APIserver component.  

### Scheduler 

Also known as `kube-scheduler`.  
This component is responsible for assigning unscheduled pods to the most suitable node.  
It ensures optimal pod placement by evaluating resource availability, constraints, and policies.

### Controller manager

A combination of many different controllers:
- node controller
- namespace controller
- deployment controller
- replication controller
- ...

Also known as `kube-controller-manager`.  
The Controller manager runs core control loops that regulate the cluster's state by reconciling actual conditions with desired specifications.  
This process is called "**reconciliation**".

### etcd

A key-value data store (NoSQL database).  
It stores every single information about the cluster.  

## Worker node components

Every worker node has a kubelet and a kube-proxy component.

### kubelet

The kubelet serves as the primary node agent in Kubernetes, running on each worker node to manage containers and ensure the node's desired state matches the cluster's specifications.  

It acts as a bridge between the Kubernetes control plane and the local node, handling pod lifecycle operations.  

It "receives" instructions from the control plane by actively monitoring the kube-apiserver.  
When it detects updates, it does what the control plane is asking and sends back a response once its job is done.  
The kube-apiserver component will then register corresponding changes in the etcd database.  

### kube-proxy

This commponent enables networking within the worker node.  
It allows pods inside the worker node to communicate with each other by creating iptable rules.  

`kube-proxy` translates Service abstractions into actual network rules, enabling communication between Services and their backend Pods without exposing dynamic Pod IPs directly.  

---

# 8. Local Multi-Node Cluster Setup

There are different ways you can install Kubernetes on your local machine:
- Minikube
- k3s
- k3d
- Kind

We will use Kind, which stands for **K**ubernetes **in** **D**ocker, a tool for running local Kubernetes clusters using Docker containers as nodes.  

## Installing Kind 

**Official instructions**:  
https://kind.sigs.k8s.io/docs/user/quick-start/  

### Installing Kind on my fedora 43 machine

- run `go version` and install the latest version if below 1.16
- run `docker version` and install docker if not installed yet
- Download the latest Kind binary from GitHub releases:
```bash
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.31.0/kind-linux-amd64
```
- make it executable via `chmod +x ./kind`
- and move it to `/usr/local/bin` via `sudo mv ./kind /usr/local/bin/kind`
- run `kind version` to confirm installation

## Creating our first Kubernetes cluster

Before creating our first cluster with kind, we need to install `kubectl`.  
Check the version of `kubectl` currently recommended for preparing the CKA exam.  

To check your `kubectl` version: `kubectl version`  
Ideally, `kubectl` version should match your Kubernetes version.  

Adapt the following command to match your version of `kubectl`:
```bash
kind create cluster --image kindest/node:v1.34.3@sha256:08497ee19eace7b4b5348db5c6a1591d7752b164530a36f855cb0f2bdcbadd48 --name cka-cluster1
```
The releases page for kind is here: https://github.com/kubernetes-sigs/kind/releases  
It will help you find an copy-paste the argument for the `--image` option.  

To create our cluster, we might be prompted for the passphrase we've set up for the GPG key we've used to initialize `pass` (lines 164-171).  
Reminder: `pass` is required to authenticate via `docker login` or `nerdctl login`.  

<img width="1547" height="243" alt="image" src="https://github.com/user-attachments/assets/cec789d8-abba-4e13-8fc9-cde2206955cf" />  

Once our cluster is up, we can run `kubectl get nodes` to see all nodes in the current cluster.  
Currently, we only have one node, which is the control plane.  

And to show the default pods created for our new clutser: `kubectl get pods -A`  

We can also run `kubectl version` to check if our version of `kubectl` (client) matches our control plane node's version (server).  

## Creating a multi-node cluster

The cluster we've created is a single node cluster, with only 1 node for the control plane, and 0 worker nodes.  
We can see it via `kind get clusters`  

In a production environment, we would have one (or several) control plane and many worker nodes.  
And the control plane would live on a different host machine than the worker nodes.  

Now, we will create a multi-node cluster by applying a YAML config file, such as this one:
```yaml
# three node (two workers) cluster config
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

- `cd` into the desired directory and create your config file: `vi cluster2-config.yaml`  
- Copy-paste the above content into it
- Write and quit

Then, to create the cluster: 
```bash
kind create cluster --image kindest/node:v1.34.3@sha256:08497ee19eace7b4b5348db5c6a1591d7752b164530a36f855cb0f2bdcbadd48 --name cka-cluster2 --config cluster2-config.yaml
```

To delete our single node cluster:
```bash
kind delete cluster --name cka-cluster1
```

We can also have a cluster with multiple control-plane nodes, which is advisable in a production environment.  
This is what we call "**High-Availability** mode" or "**HA** mode". The YAML file would be as follows:
```yaml
# a cluster with 3 control-plane nodes and 3 workers
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: control-plane
- role: control-plane
- role: worker
- role: worker
- role: worker
```

## K8s Cluster context

If we had multiple clusters running, we can show them via:
```bash
kubectl config get-contexts
```

To show current context:
```bash
kubectl config current-context
```

In order to switch to a specific cluster:
```bash
kubectl config use-context <context-name>
```

# 9. K8s Pods & YAML config file

Kubernetes is all about encapsulating container workloads into Pods.  

Because Pods address multiple challenges and offer major benefits:
- Resource scheduling and utilization
- Reliability and self-healing
- Declarative, automated lifecycle management
- Horizontal scaling and traffic management
- Consistent environment and portability
- Zero-downtime updates and rollbacks

## Imperative vs. Declarative

There are 2 ways of deploying pods in Kubernetes:
- imperative: by running commands such as `kubectl run nginx` to explicitly tell Kubernetes what to do
- declarative: by writing a YAML (or JSON) config file that describes the desired state and not the steps needed to reach that state

## The imperative way

Let's create an Nginx pod from the terminal:
```bash
kubectl run nginx-pod --image=nginx:latest
```

Now, we can see our pod running via `kubectl get pods`

## The declarative way

Let's create a directory and an example config file:
```bash
cd ~/Documents/DevOps
mkdir day07_code
cd day07_code
vi day07.yml
```

Our YAML file will contain the following:
```yaml
# this is an example pod config file

apiVersion: v1
kind: Pod
metadata: 
  name: nginx-pod2
  labels:
    env: demo
    type: frontend
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80
```
To check the apiVersion, run `kubectl explain pod | head`  

Now, we can create our pod via `kubectl apply -f day07.yml`  
And we can see it getting created then running via `kubectl get pods`  
To delete the pod: `kubectl delete pod nginx-pod2`  

## Useful Pod Commands

- to show all details about a specific pod: `kubectl describe pod <pod_name>`
- to know which node a pod is running on: `kubectl get pods -o wide`
- to edit (via **Vim**) a pod's config while it's running: `kubectl edit pod <pod_name>`
- to open an interactive shell inside the default container of a pod: `kubectl exec -it <pod_name> -- sh`
- to do the same inside a specific container of the pod: `kubectl exec -it <pod_name> -c <container_name> -- sh`
- to exit out of the container and come back to the host machine: `exit` (or maybe `ctrl + D`)

>[!note]
>The `--` separates the kubectl arguments from the command you want to run inside the container; anything after `--` is executed inside the pod’s container.

>[!important]
>You need to be proficient with **Vim** (the text editor), and not only for the CKA exam!

Here's my personal "Vim Motions" tutorial:  
https://github.com/fastoch/Vim-Motions/blob/main/README.md

## Pod Labels

Labels on a pod are key-value pairs used to identify and group pods for management and operations.  
They enable selecting and organizing pods by criteria such as environment, app, version, etc., which is **essential** for applying policies, routing traffic, and scaling.  

- to show labels of a specific pod: `kubectl get pods <pod_name> --show-labels`

## From imperative to declarative

To write a YAML file, we can type an imperative command and output its result to a YAML file:
```bash
<kubectl_command> -o yaml > example-pod-config.yaml
```
or
```bash
<kubectl_command> -o json > example-pod-config.json
```

30/33
video 8/59

---

# 15. Kubernetes Networking

## How do Pods simplify container networking in Kubernetes? 

K8s assigns each pod a unique IP address.  
All containers in a pod share the same network **namespace** and IP address, each container having its own port.  

The K8s networking model requires that every pod can reach every other pod in the cluster without NAT (network address translation), across all nodes.   

This flat, NAT-free network greatly simplifies connectivity, discovery, and debugging because source IPs are preserved and packets do not traverse complex translation layers.  

## What problem do Kubernetes Services solve?

Because pods are **ephemeral**, so are their IP address, which is why Kubernetes introduced **Services**, so that pods can be reached through stable virtual IPs.

---

# 16. Why should we use the GitOps approach?

## Using Flux

## Using ArgoCD

