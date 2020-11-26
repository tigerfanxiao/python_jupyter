# Container

Containers are isolated execution environment that let us quickly and efficiently deploy exact copies of our desired environments. This is done by virtualizing at the operating system-level and isolating any libraries and applications within the container.

**Bare Metal**: No virtualization - the application or service is deployed directly onto the machine

### Container VS virtual machine

![image-20201114231913447](Docker.assets/image-20201114231913447.png) 

### Container Runtime

* docker (Most Popular)

* rkt

  Created by CoreOS, “designed with composability and security in mind.”

* containerd

  Emphasizes “simplicity, robustness, and portability"

### Container Use Case

* Solving Dev/Prod Parity
* Migrating Infrastructure
* Microservice-Based Architectures
* Elastic Architectures
* Self-Healing Architectures
* CI/CD pipeline

### Self Healing

Self-Healing Applications are applications that are able to automatically detect when something is broken and automatically take steps to correct the problem without the need for human involvement.

Since it is so quick and easy to start up new container instances, when something goes wrong with a container it can often be easily destroyed and replaced within a few seconds.

# Docker

## Concepts

* Docker hub: online official docker image repository	

* Docker is primarily a container runtime.

## Install docker

Remove existing Docker installs. update the uberall

```bash
sudo apt update
```

And then install any prerequisite packages.

```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```

Note that many of these packages are already installed on our Playground server.

We can now add Docker's GPG key.

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

And verify its configuration.

```bash
sudo apt-key fingerprint 0EBFCD88
```

To add the Docker repository, we now just need to run:

```bash
sudo add-apt-repository  "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

And update our server again.

```bash
sudo apt update
```

We can now install the necessary packages.

```bash
sudo apt install docker-ce docker-ce-cli containerd.io
```

To finish up, we want to add our `cloud_user` user to the `docker` group. Then you can run docker without `sudo`

```bash
sudo usermod cloud_user -aG docker
```

Log out then log back in to refresh the Bash session before running any `docker` commands.

## Run Docker

Now that we have Docker installed, Docker provides a `hello-world` container to test our installation. This container provides us with a high-level overview of what happens when we run a Docker container. Let's check it out by running:

```bash
docker run hello-world
```

Now let's take a look at the output, starting at the steps provided after the `Hello from Docker!` message.

```bash
1. The Docker client contacted the Docker daemon.
2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
3. The Docker daemon created a new container from that image which runs the executable that produces the output you are currently reading.
4. The Docker daemon streamed that output to the Docker client, which sent it to your terminal.
```

To start (`1`), the Docker client takes our command and relays it to the Docker daemon, which then (`2`) looks for the `hello-world` image. An **image** is the base server configuration that our containers are based on. This defines the operating system, any packages, and more. For example, the `hello-world` image is specifically set up to send us the provided message, while the `ubuntu` image is just the stock version of the latest version of Ubuntu.

Since we didn't pull down the `hello-world` image before we ran our `docker run` command, the daemon automatically searched the **Docker Hub**, which is a repository of Docker images both supplied by Docker and the community as a whole. Once the image is found, a copy is pulled down to our server.

The Docker daemon then (`3`) deploys a container based on the image, which in turn provides us with the above output once available. The output goes into further detail (`4`), but all we need to worry about is this:

When we run our Docker containers, these containers are based on an image. We can coordinate commands to run after deploy and can either leave our container running or exit once its task is done.

## Wrap-Up

- Test the Docker install by running `docker run hello-world`
- Run a test container that describes how Docker functions:
  - Client sends tasks to daemon.
  - Daemon looks for the needed image.
    - If not found, it pulls the image from Docker Hub (or another repository).
  - Deploys the container based on image.
  - Container runs any scripted commands to reach its needed end state.

When we ran our initial `docker run hello-world` command, the container was shut down after it completed its run. But this isn't behavior unique to the `hello-world` container, even if we run `docker run` against another image.

```bash
docker run alpine
```

The container does not remain running. We can see this if we list our running containers.

```bash
docker container ls
```

Or:

```bash
docker container list
```

To see a list of all containers, we can use the `-a` flag.

```bash
docker container ls -a
```

Notice that each has a `STATUS` noting when the container was exited. To launch a container and actually be able to work with it, we can use the `-i` or `--interactive` and the `-t` or `--tty` flags. `t` also make sure that the container will have persistent ip address

```bash
docker run -it --name a-container alpine
```

Note that we also assigned the container a name with `--name`.

This drops us into a prompt. From here, we can make any changes or run any tests we desire against the container.

```bash
apk upgrade
```

But when we exit, the container shuts off again:

```bash
exit
docker container ls
```

To run a persistent container, we can use the `-t`flag alongside `-d` to assign a TTY. We can also ensure our containers restart should the host server shut down with the `--restart` flag. 

When this flag is set to `always`, this starts the container after it initially exits and any time afterward. 

We can also use the `unless-stopped` setting so it will restart unless we manually stop it, 

the `on-failure` setting should we want it to restart upon failure, 

or the default `no` which never restarts. We'll be using the command with `-d` or `--detach`, which runs the container in the background instead of dropping it into a prompt.

```bash
docker run -dt --name bg-container --restart always alpine
```

Notice how it outputs its container ID. We can also confirm it's running with our `docker container ls` command.

Alternatively, if we want a truly throwaway test container, we can use the `--rm` flag so it's removed immediately after exiting.

```bash
docker run -it --name rm-test --rm alpine
exit
docker container list -a
```

## Wrap-Up

- ```
  docker container ls
  ```

  \- List running containers

  - `-a` - List **all** containers

- ```
  docker run <image>
  ```

  \- Create and run a container based on the provided image

  - `-i`, `--interactive` - Keep STDIN open

  - `-t`, `--tty` - Locate a psuedo-tty; without this, there's no command prompt

  - `-d`, `--detach` - Run the container in the background

  - ```
    restart <value>
    ```

     

    \- Set restarting parameters

    - `no` - Never restart
    - `always` - Always restart when stopped
    - `unless-stopped` - Restart unless manually stopped
    - `on-failure<:retries>` - Restart when the container fails; we can supply the maximum amount of retries

  - `--rm` - Remove the container upon exit

  - `--name <name>` - Provide a nickname for the container

At this point, we have four containers, but only one is up and actively running.

```bash
docker container list -a
```

To **start** one of our exited containers, we can use the `docker start` command. Let's work with our `a-container` container.

```bash
docker container start a-container
```

From here, if we want to work with our container, we can use the `docker exec` command. For example, let's add `nginx` to our container.

```bash
docker exec a-container apk add nginx
```

Alternatively, we can drop directly into our container's shell by pairing the `exec` command with `-it`, the same as when we were working with `docker run`. We'll want to specify the name of the shell as the command (ie. `docker exec -it <container> <shell>`):

```bash
docker exec -it a-container ash
```

From here, we can run commands as we would normally.

```bash
cd /etc/nginx
ls
```

Log out when finished.

```bash
exit
```

We can also upload files as needed. Let's fill out our `a-container` web server by updating the default `nginx` server config. We can do this with `docker cp <source> <dest>`. The source and destination can be either the localhost or a container. Let's first copy down our `default.conf` file.

```bash
docker cp a-container:/etc/nginx/conf.d/default.conf .
```

This copies it directly to our current directory under its current name, `default.conf`.

Open the file and update the contents.

```bash
vim default.conf
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/;
}
```

We can now replace it by pushing it back to the same location.

```bash
docker cp default.conf a-container:/etc/nginx/conf.d/default.conf
```

We can confirm with:

```bash
docker exec a-container cat /etc/nginx/conf.d/default.conf
```

## Wrap-Up

```shell
# Stop and restart a container
docker start <container>
docker restart <container>
docker stop <container>
```

- `-i`: Attach to STDOUT

- ```
  docker exec <container> <command>
  ```

  : Run a command against the container

  - `-it <container> <shell>`: Drop into a shell prompt

- ```
  docker cp <source> <container>:<destination>
  ```

  : Copy a file to the container

  - `<container>:<source> <destination>`: Copy **from** a container to localhost

At this point, we have both our `bg-container` and `a-container` running, but we're really only going to work on our `a-container` from here on out. We also want to give it a name and perform some other general management and cleanup tasks on our server.

First, let's consider stopping and starting our server. We know `start` starts our container. Other commands are similarly self-explanatory. `docker stop` will shut down a container, while `restart` will `stop` and then `start` it again. Let's just shut down our `bg-container` with a `docker stop`.

```bash
docker stop bg-container
```

And since we no longer need this container, we can remove it.

```bash
docker rm bg-container
```

But what about our other unneeded containers? We could remove these either by looking up their names or IDs and removing them via the `rm` command. Alternatively, we can remove **all** our stopped containers in one go, which we'll do via `docker container prune`.

```bash
docker container prune
```

Now if we list all our containers, we only have the single, running container left.

```bash
docker container ls -a
```

Finally, let's work with our remaining container a little. We've installed nginx on it, so it's safe to assume this is actually a web server. Let's rename it to reflect this.

```bash
docker rename a-container web01
```

Finally, we can perform a simple performance check on our container by running `docker stats` against it, outputting our CPU, memory, and process information.

```bash
docker stats
```

This will output the information for **all** containers. We only have the one right now, but if we needed to specify, we could add the container name at the end of the command.

## Wrap-Up

- `docker stop <container>`: Stop a container
- `docker restart <container>`: Restart a container
- `docker rm <container>`: Remove a container (more than one container can be specified)
- `docker prune`: Remove all stopped containers
- `docker rename <container> <new-name>`: Rename a container, even when it is running
- `docker stats`: List resource stats for all containers
- `docker stats <container>`: List resource stats for the specified container

## References

- [Docker Cheat Sheet](https://www.docker.com/sites/default/files/d8/2019-09/docker-cheat-sheet.pdf)

We know how to access our container through some `docker` commands, but what if we want to share our container with the greater world? Docker provides us with the option to publish our container to a port on our host, letting us share or otherwise test our applications and services.

Before we start, however, let's make sure we have something we can actually check and create an index page for our web server.

```bash
vim index.html
Hello, World!
```

Save and exit the file, then push it to the container.

```bash
docker cp index.html web01:/var/www/
```

Start nginx.

```bash
docker exec -dt web01 nginx -g 'pid /tmp/nginx.pid; daemon off;'
```

Now let's see if this works. Right now, our container can only be accessed from our host. Specifically there's a private network we can use to access the host. So to see if our web configuration is working, we can `curl` the container's private IP. But first, we need to locate the private IP. We can find this with `docker inspect`.

```bash
docker inspect web01 | grep -i ip
```

This outputs a variety of information about our container, including container state, overall configuration, and networking. Look for the `IPAddress` value, then use `curl` to see if everything is working.

```bash
curl <IPAddress>
```

Our web container is working! But we've now encountered a problem: we can't publish an existing container to a port on our host, at least not through Docker. Instead, we'll need to work with a copy of this container.

First, we'll need to create an image based on our container. We can do this with a:

```bash
docker commit web01 web-base
```

We can then deploy our container with `docker run`. Only this time, we specifically want to use the `--port` or `-p` flag, to map the host port to the container.

```bash
docker run -p 80:80 -dt --name web02 web-base
```

We may also need to restart `nginx`.

```bash
docker exec web02 rc-service nginx restart
```

We can now navigate to the **public IP** of our localhost and see our `index.html` file.

## Wrap-Up

```shell
# View container information
docker inspect <container>
# Create an image of an existing container
docker commit <container> <image-name>
# Publish to host port
docker run -p <host_port>:<container_port> --dt --name <container> <image-name>
```

We wouldn't have been able to complete any of our previous lessons if not for the `alpine` Docker image, and Docker cannot function without a base image for the container to base itself on. But what **is** an image? Those familiar with virtual machine images probably already have some idea, and they'd be mostly right save for a few differences to increase reusability, decrease disk space, and ensure the `docker build` runs quickly. To achieve this, images are comprised of intermediate layers that are then cached. An image should contain everything it needs to complete its intended job, whether that be run an application, microserver, or otherwise. This includes code, runtime, libraries, environmental variables, and configuration files.

We can see the steps taken to build an image by viewing the `Dockerfile` of an image. This can be found in the image repository itself, which in most cases will be the **Docker Hub**. Let's look at the `hello-world` image, found [here](https://hub.docker.com/_/hello-world).

Under **Shared Tags**, click on the **latest**: **linux** build. Notice the simple Dockerfile:

```bash
FROM scratch
COPY hello /
CMD ["/hello"]
```

The `hello-world` image is a **base image**. This means it's not based on any other image, but instead uses `FROM scratch` to signify the next command will be the filesystem layer. Images can also be based on other images. For example, the [`nginx`](https://hub.docker.com/_/nginx) image we used in the lab is based on Debian Buster (`FROM debian:buster-slim`). This is the image's **parent image**.

The next line copies the `hello` binary to the root of the filesystem, which can be found [here](https://github.com/docker-library/hello-world/blob/master/hello.c). This is the output message we saw when we ran the `hello-world` container.

The final line runs the binary, outputting the `DOCKER_GREETING` message. There can only be one `CMD` line, and it's the command that's run after the container is launched. If not persistent, the container will automatically exit once the command has finished.

Each line works as its own layer and is cached separately, as indicated earlier.

Now, let's get hands-on and create our own Dockerfile in the next lesson.

## Wrap-Up

- Images are based on a Dockerfile
  - **Base Image**: An image based on the `scratch` template; the first layer defines the file system.
  - **Parent Image**: The image from which the current Docker image is based.
- Images are comprised of layers.
  - Each layer is cached.
  - Increases reusability.
  - Decreases disk space.
  - Ensures speed during deploy.

Now that we know what images are made of, let's make one ourselves via Dockerfile. For this, we're going to dockerize a Node.js application.

Let's grab our app:

```bash
cd
wget --content-disposition 'https://github.com/linuxacademy/content-Introduction-to-Containers-and-Docker/raw/master/lessonfiles/demo-app.tar'
tar -xf demo-app.tar
cd app/
```

We need our Dockerfile to have the following layers:

1. Based on the Node.JS 10 Alpine image (`node:10-alpine`)
2. Creates the `/home/node/app/node_modules` directory and sets `node` as the owner and group
3. Sets the working directory as `/home/node/app`
4. Copies app `package*` files to the working directory
5. Sets the NPM registry
6. The installs are prerequisite NPM packages
7. Copies the rest of the `app/` data to the working directory and sets the owner and group of the working directory to `node`
8. Switches to the `node` user
9. Opens port 8080 to be used
10. Runs the application

So let's create our `Dockerfile` and start.

```bash
vim Dockerfile
```

Dockerfiles are built using **instructions** which are just set keywords that perform certain actions. We saw three in the previous lesson: `FROM`, `COPY`, and `CMD`. Instruction keywords do **not** have to be written in all caps, but it is preferred.

We first need to define our parent image with `FROM`; all valid Dockerfiles must begin with `FROM`.

```
FROM node:10-alpine
```

We now want to create a directory and set its owner. For this, we can use the `RUN` command, which lets us execute any commands against the shell. Our image is based on Alpine, which uses the `ash` shell, so our command will be `mkdir -p /home/node/app/node_modules && chown -R node:node /home/node/app`. All we have to do is feed that into the `RUN` instruction.

```
RUN mkdir -p /home/node/app/node_modules && chown -R node:node /home/node/app
```

Next, we need to move into the `/home/node/app` directory. We do this by setting a **working directory** with `WORKDIR`. Any `RUN`, `CMD`, `COPY`, `ADD`, and `ENTRYPOINT` instructions that follow will be performed in this directory:

```
WORKDIR /home/node/app
```

We now want to add our files. We have two options for this: `COPY` and `ADD`. `ADD` can pull files from outside URLs, and thus utilizes additional functionality. In other words, it's best to use `COPY` if you can, and since our `package*` files are all local, we can do so:

```
COPY package*.json ./
```

With the first argument being our source file(s) and the second our destination.

We now need to ensure the prerequisite packages for our application are installed from NPM. These are just more `RUN` commands.

```
RUN npm config set registry http://registry.npmjs.org/
RUN npm install
```

We can now copy over the rest of our application files using `COPY`. We'll also want to set the owner of our working directory to the `node` user and group. Since we're on Linux, this can **also** be achieved using some special `COPY` functionality.

```dockerfile
# copy the all the local files to the docker working directory and set the owner to node:node
COPY --chown=node:node . .
```

From here, we want to switch users. This works similarly to the previous `WORKDIR` command, but now we're switching users not directories for any following `RUN`, `CMD`, and `ENTRYPOINT` commands.

```
USER node
```

Our application is hosted on port 8080, so we also want to make sure that port is available on our container. For this, we'll use the `EXPOSE` keyword.

```
EXPOSE 8080
```

And now, finally, we want to provide the command (`CMD`) run as soon as the container is launched. Unlike `RUN`, `CMD`'s preferred format doesn't take this as a shell command. Instead, the executable name should be supplied in an array, followed by any parameters: `CMD ["executable","param1","param2"]`. In our case, this will be:

```
CMD [ "node", "index.js" ]
```

Our final Dockerfile should look like:

```
FROM node:10-alpine
RUN mkdir -p /home/node/app/node_modules && chown -R node:node /home/node/app
WORKDIR /home/node/app
COPY package*.json ./
RUN npm config set registry http://registry.npmjs.org/
RUN npm install
COPY --chown=node:node . .
USER node
EXPOSE 8080
CMD [ "node", "index.js" ]
```

Save and exit (`:wq`).

We can now build our container using the `docker build` command and specifying the location of our Dockerfile. Since we're in the same directory, we can just denote this with a `.`. That said, we'll want to provide a name for our output image, so we'll want to use the `-t` flag to set a flag. If we're planning on adding this to the Docker Hub, we'll want to follow a `<username>:<imagename>` format for the tag. Since we aren't, we'll just give it the simple name of `appimage`.

```shell
docker build . -t appimage
# show docker images
docker images 
```

We can now run our container.

```
docker run -dt --name app01 appimage
```

And see that our app is working.

```
docker inspect app01 | grep IPAddress
curl 172.17.0.2:8080
```

## Wrap-Up

- Dockerfiles are built using a series of instructions, dictated by set keywords:

  - All Dockerfiles start with `FROM`.

  - `RUN` runs commands on the shell.

  - `WORKDIR` sets the working directory.

  - ```
    COPY
    ```

    copies files 

  * `ADD` also copies files, but can copy files from outside URLs.

  - `ADD` uses more resources.
  - We should use `COPY` when possible.
  - `USER` sets the working user.
  - `EXPOSE` exposes the container port.
  - `CMD` is the final command in the Dockerfile and is the command run after the container is launched.

- [See a full list of instructions.](https://docs.docker.com/engine/reference/builder/)

- ```
  docker build <dockerfile>
  ```

   

  builds the image.

  - `-t`: Set a name; we should use `<username>:<imagename>` if using the Docker Hub.

Now that we've pulled down, run, and otherwise created a couple of images, we should take the time to learn how to manage them. Luckily for us, many of the commands for image management are similar to the commands we used to manage our containers themselves.

Let's start with considering how images are used: images are pulled from their host repository and then cached on the local machine. When we perform a `docker run`, Docker will first look locally, then search the Docker Hub if it can't find an image. If we want to pull down an image from the Docker Hub **before** we run a `docker run`, we can use the `docker pull` command.

```bash
docker pull alpine
```

Notice how it tells us that our image is already up to date; we had pulled down the `alpine` image earlier in this course, after all.

Should we want to see which images we have stored locally, we can run:

```bash
docker image ls
```

Notice we have our `appimage`, as well as the `alpine`, `web-base`, and `hello-world` images.

Should we desire to remove any of these images, we can use `docker image rm <container>`, or, alternatively, pair `docker image prune` with the `-a` flag to remove all images not associated with a container. Lets do this:

```bash
docker image prune -a
```

Finally, we can review information about any of our images using `docker image inspect`.

```bash
docker image inspect appimage
```

## Wrap-Up

- Images are managed similarly to containers (add `image`).

- `docker pull <image>`: Pulls down an image; this happens automatically when no image is available during `docker run`.

- ```
  docker image ls
  ```

  : Lists all Docker images.

  - `--no-trunc` shows the full ID.

- `docker image rm <image>`: Removes an image.

- ```
  docker image prune
  ```

  : Removes all dangling images.

  - `-a`: Removes all images not being used by a container.

- `docker image inspect <image>`: Displays image data.

If we want to share our Docker images with the world (or just our coworkers), we can create a Docker repository on the Docker Hub found at [hub.docker.com](https://hub.docker.com/). Log in or create an account with a username, password, and associated email.

Once logged in, select **Repositories**, then click the **Create Repository** button. Give your repo a name; I'll be using the name `demo-app`. Leave the repo public, and provide no build settings. **Create** the repo.

From here, we'll want to switch to the command line. To upload our image to the Docker Hub, we'll need to log in from the command line.

```bash
docker login --username=<username>
```

Input the password when prompted.

Before we can push directly to our repo, however, we need to tag our image with the name of our repository in a `<username>/<reponame>` format. This is done via `docker tag`.

```bash
docker tag appimage <username>/demo-app:latest
```

We can now use the `docker push` command to share it.

```bash
docker push <username>/demo-app
```

If we return to our repo in our browser and refresh, then select **Tags**, we can see the history of our image and share various versions as needed.

## Wrap-Up

- Use the Docker Hub to host images.
- `docker login --username=<username>`: Log in to your Docker account on the CLI.
- `docker tag <image> <username>/<reponame>`: Tag the image with the repo name.
- `docker push <username>/<reponame>`: Push the image to Docker Hub.

If we want to share our Docker images with the world (or just our coworkers), we can create a Docker repository on the Docker Hub found at [hub.docker.com](https://hub.docker.com/). Log in or create an account with a username, password, and associated email.

Once logged in, select **Repositories**, then click the **Create Repository** button. Give your repo a name; I'll be using the name `demo-app`. Leave the repo public, and provide no build settings. **Create** the repo.

From here, we'll want to switch to the command line. To upload our image to the Docker Hub, we'll need to log in from the command line.

```bash
docker login --username=<username>
```

Input the password when prompted.

Before we can push directly to our repo, however, we need to tag our image with the name of our repository in a `<username>/<reponame>` format. This is done via `docker tag`.

```bash
docker tag appimage <username>/demo-app:latest
```

We can now use the `docker push` command to share it.

```bash
docker push <username>/demo-app
```

If we return to our repo in our browser and refresh, then select **Tags**, we can see the history of our image and share various versions as needed.

## Wrap-Up

- Use the Docker Hub to host images.
- `docker login --username=<username>`: Log in to your Docker account on the CLI.
- `docker tag <image> <username>/<reponame>`: Tag the image with the repo name.
- `docker push <username>/<reponame>`: Push the image to Docker Hub.

# Orchestration 

Kubernetes is Orchestration tool to manage container

### AWS EKS

Amazon Elastic Kubernetes Service (EKS) is a managed service that makes it easy for you to run Kubernetes on AWS without needing to install and operate your own Kubernetes control plane or worker nodes.

### Other Orchestration

* Marathon 

  Based on Apache Mesos, offers APIs for integrating with other tools.

* Docker Swarm

   Docker Swarm Docker’s native container orchestration solution

* Nomad

  Open source and built by HashiCorp, designed to be simple and lightweight.

# Scenario

Zero-Downtime Deployments

A zero-downtime deployment (with containers) goes like this: 

1. Spin up containers running the new code. 
2. When they are fully up, direct user traffic to the new containers. 
3. Remove the old containers running the old code. No downtime for users!

Containers are great at accomplishing things like: 

* Software Portability – Running software consistently on different machines. 
* Isolation – Keeping individual pieces of software separate from one another. 
* Scaling – Increasing or decreasing resources allocated to software as needed. 
* Automation – Automating processes to save time and money. 
* Efficient Resource Usage – Containers use resources efficiently, which saves money

# Microservice

### Monolith VS Microservices

![image-20201115005928019](Docker.assets/image-20201115005928019.png)

Containers excel when it comes to managing a large number of small, independent workloads.

Containers and orchestration make it easier to manage and automate the process of deploying, scaling, and connecting lots of microservice instances. 

For example, I may have one microservice that needs additional resources. With containers, all I need to do is create more containers for that service to handle the load. With orchestration, that can even be done automatically and in real time!

# Cloud Transformation

Containers can help you move into the cloud. It is relatively easy to wrap existing software in containers. While containers may not be the answer for every type of existing software, they are a powerful tool.

Automated Scaling refers to automatically provisioning resources in response to real-time data metrics.

# CI/CD

Continuous Deployment is the practice of deploying new code automatically and frequently.

Instead of writing new code for months and doing a big deployment, continuous deployment means constantly doing many small deployments. Some companies even do multiple deployments a day!

Containers work very well in the context of continuous deployment. They make it easy to test code in an environment that is the same as production, because the code can be automatically tested inside the container itself.

An automation pipeline for continuous delivery can automatically build a container image with the new code, test it, then automatically ship that same container image production.

# Developer Visibility

With Containers, the container is the production environment. This means that anyone can spin up an environment that is exactly like production, even on their own laptop. Developers (and others) have the ability to test their code and see exactly how it will behave in production. 

The additional visibility offered by containers can help your organization develop and troubleshoot code much more efficiently!

# cloud and container

Why do Containers in the Cloud? 

Containers are just one way to run software in the cloud. 

Some of the benefits of using containers in the cloud are: 

• Regular Benefits of Containers You get all the normal benefits of containers in terms of portability, speed, and ease of automation. 

• Save Money Since containers are lightweight and have little overhead, they can help save on cloud costs. With cloud providers, you only pay for the resources you use. Use less resources, pay less 

• Cloud Platforms Offer Container Support out of the Box Most cloud platforms provide services built specifically for containers

# Next Steps

## Course on Linux Academy

| Course                    | Level    | Topic                     | Dependency                | Status   |
| ------------------------- | -------- | ------------------------- | ------------------------- | -------- |
| Container & Orchestration | Beginner | Container & Orchestration | No                        | Finished |
| Docker Quick Start        | Beginner | Docker                    | Container & Orchestration |          |
| Kubernetes Quick Start    | Beginner | Kubernetes                | Docker Quick Start        |          |

Containers 

•  LXC/LXD Deep Dive

 • CoreOS Essentials 

Docker 

• Docker Deep Dive 

• Docker Certified Associate Prep Course

Kubernetes 

• Certified Kubernetes Administrator (CKA)

• Kubernetes the Hard Way 

DevOps 

• DevOps Essentials 

• Implementing a Full CI/CD Pipeline

