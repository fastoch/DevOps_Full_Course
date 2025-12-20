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
- `sh` is the shell we'll be using while inside the container


32/35 Day2/40

---

# 5. 

---

# 6. 

---

# 7. 

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

# 9. 

---

# 10. Why should we use the GitOps approach?

## Using Flux

## Using ArgoCD

