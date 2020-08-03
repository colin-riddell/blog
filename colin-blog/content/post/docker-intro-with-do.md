+++
draft = false
date = 2019-09-19T18:19:14+01:00
title = "Deploying to Cloud with DigitalOcean + Docker"
description = ""
slug = ""
tags = ["docker", "DigitalOcean", "cloud", "devops"]
categories = []
externalLink = ""
series = []
author = "Colin Riddell"
+++




Containerization allows development teams to move fast, deploy software efficiently, and operate at an unprecedented scale.

-------

* They are small
* They are fast
* They consume no more engergy than running an application
* They take up 100's of times less space and memory than virtual machines


## Setup

### *Optional* Get a DO account

[https://m.do.co/c/e224d565e5ec](https://m.do.co/c/e224d565e5ec) - this link gives 100 USD credit.

### Install Docker - do before starting

[Docker Download - Mac](https://docs.docker.com/docker-for-mac/install/)

OR

```bash

curl https://download.docker.com/mac/stable/Docker.dmg --output Docker.dmg && open Docker.dmg

```
## Get Started - Docker

### What is Docker

Docker is a container engine application that runs on Mac, Windows and Linux that allows us to run Containers

### What are containers?

Containers are operating systems that run virtually within a host system such as Linux, Mac or Windows. Host systems are typically servers in the cloud or development laptops running locally.

Containers are typically very small taking up just enough memory and disk space to perform their function.

They are ephemeral, meaning they are designed to be short-lived and never permanent.

These small short-lived operating systems are for running processes on the host in such a way that the host doesn't need to know about or provide any of the dependencies of the application directly.

For example if we wish to run an express.js application there are two choices. Without the container (as normal) - where we'd need to install the dependencies on the host machine (server, laptop etc). Or with a container where the dependencies would be installed in the container.

Running an application in a container vs running on the host gives consistency between running all the various places that app might need to run. We might need to run something on several dev' machines, test servers, integration servers and one or more production servers.



Since containers are small environments for our applications to run, we can use containers to scale the application horizontally.

### What are containers: Technically

To understand what containers *are* we need to understand a little Linux. The Linux OS is as simple as:

* A filesystem (files)
* A kernel (core process)
* Processes (running applications)

A container is all of these things running together virtually to give the *illusion* that they are running on a separate intependent operating system.

#### How?

##### chroot
The linux kernel supports mapping a filesystem within a filesystem to a specific process as a particular user.


##### cgroups
cgroups, short for control groups, allow kernel imposed isolation on resources like memory and CPU. After all, what’s the point of isolating processes they can still kill neighbors by hogging RAM.

##### nsenter

We need networking. NSEnter works with the kernel to help create virtual networking namespaces.

## Docker

**Docker** makes use of all these system-level tools to create contained systems that run within the host, but are isolated completely from the host and other containers.

Docker brings together a lot more than `chroot`, `cgroups` and `nsenter` - but these alone are enough to give the basis of running an Operating System within an Operating System.



### Running a container


```
docker run -it ubuntu bash
```

This is the docker run command.
`-it` is `-i` and `-t`.  `-i` asks for network access from the host and `-t` asks for a terminal command to be run.

`ubuntu` is the **image** we wish to use. Docker pulls common images from the online Docker Registry. This is a repository of official images that people contribute docker images of various Operating Systems and applications. Nearly every application you can imagine has a Docker image on there.

`bash` is the command we wish to run. This simply runs the `bash` shell.
**Why not run zsh** - it's common practice to run bash as it's universally available on nearly every Linux distribution.

**Running that command...** we're now in a container. Wasn't that fast? We can do whatever we want here :)


## Node in container

### Running without container

Running an express.js application without docker is as how we would run it normally.


```bash
npm start # in the server

mongod # in a separate tab
npm run serve   #in the client directory

```

This is fine when we're developing locally.. but even in that case we have three tabs to run one application.

Docker containers allow us to do this more easily.


### With a container

```
docker run -it --rm --name auth_app -v "$PWD":/usr/src/auth_app -w /usr/src/auth_app node:8 npm install; npm start
```

In another tab we can do `docker ps` (docker list running containers), to see it running.

This will work, but then fail to connect to the MongoDB.

To solve this we could open another tab then do another long run command to find a mongod image, pull that and run it, but we might still have issues with that and typing these huge run commands isn't nice.

**Ideally, we'd have each thing in its own container. The back-end, the front-end and the mongodb**

## Images

To save us from typing really long commands to use the Docker API, we can use a more script-like code to create containers. To create containers we first need to make a docker **image**


## Images vs Containers


**What's the difference between a container and an image - in docker?**

**Docker Container** is a _running_ docker **image**.

thus

**Docker Image** is like a blueprint for a container. It is literally just a zip (actually `tar.gz`) of the filesystem that the container should use.

### Creating an Image

The **Dockerfile** gives us a way to script the creation of an **image**.

Let's create our first Dockerfile to run the bucket list app.


At the same level as the `package.json` in the bucket list node project:

```Dockerfile
#Dockerfile
FROM node:8

WORKDIR /usr/src/auth-app/

COPY . .

RUN npm install

EXPOSE 3000

CMD [ "npm", "start" ]

```

#### Dockerfile commands broken down

* `FROM node:8` - Docker allows us to create images but use other images as the starting point. You can think of this as being _like_ inheriting between images.  If we use the official node.js as our _base image_, then we will get a container that already has node installed, specifically version **8**
* `WORKDIR` - We need this to specify a default working directory **in the container**, not on the host machine. Once this is set, that means the `.` or the `pwd` in the container is set to this location. When the container starts it will start in this directory. If that directory doesn't exist it will be created.
* `COPY <src> <dest>` - Is used to copy files **from the host to the container**. `<src>` is a location on the host. `<dest>` is a location on the container. We use `.` as we've already set a `WORKDIR`.
* `RUN npm install` - Runs the given command on the container as part of the **build** process for creating that image. This is not to be confused with `CMD` which sets up a command to run when the container is launched. We can have as many `RUN` lines as we require. It's very common to have `RUN` lines that install dependencies.
* `EXPOSE 3000` - By default containers have all their I/O including networking shut off from the host and other containers. Our node app will run on 3000, so we open 3000 in order to see it working.
* `CMD [ "npm", "start" ]` - This is where we specify the container **entrypoint**. This is what's run when the container starts. Remember if a container's `CMD` is simply `ls` it will just run that command and exit. If it's something like a web server it will run that while the container is running. **As soon as the process started by `CMD` exits, the container also exits.**

### Exclude the `node_modules` from being copied
Cretate a `dockerignore` file and add - we don't want

```
node_modules
```

### Buidling + Running the Dockerfile

We can build that Dockerfile into an image called `bucket_list` with the command:

```bash
docker build . -t auth-app
```

We should be able to see the image listed when we do `docker images`.

To run the resulting image:

```bash
docker run -p3000:3000 auth-app
```
**Why did you need to specify the port?**
The port setting we specified in the Dockerfile is just the one we want to ensure the image will allow us to expose. We still need to ask the run command to expose it.

[http://localhost:3000/](http://localhost:3000/) should now have the running app. **But we still need a MongoDB database**. We'll look at doing this in the next section...

## Multiple containers

We _could_ create a MongoD Dockerfile and create a container for it like that, but that requires us to run two containers at the same time in different tabs... so let's instead use `docker-compose`.

**docker-compose** allows us to create a scheme in `yaml` to orchistrate the running of multiple containers and be able to turn them all of, or all on at the same time. This way, we could "spin up" our mongod, client and server in one command.

**docker-compose** lets us build images and start containers through one nice easy to use command line tool and an easy to understand yaml configuration.

We still use Dockerfiles..

Let's build a docker-compose.


```yaml
#docker-compose.yml
version: '3.1'

services:
  auth_server:
    container_name: 'auth_server'
    ports:
      - 3000:3000
    build: ./server/
    links:
       - mongoservice
    restart: on-failure

  mongoservice:
    image: mongo:latest
    container_name: "mongoservice"
    environment:
      - MONGO_DATA_DIR=/data/db
      - MONGO_LOG_DIR=/dev/null
    volumes:
     - ./data/db:/data/db
    ports:
      - 27017:27017
    command: mongod --smallfiles
    restart: on-failure

```



File structure changes to support `docker-compose` microservices.

Let's make a new directory called `services`, `client`, `server` and `docker-compose.yml` into it.


```
.
└── services
    ├── client
    │   ├── README.md
    │   ├── babel.config.js
    │   ├── node_modules
    │   ├── package-lock.json
    │   ├── package.json
    │   ├── postcss.config.js
    │   ├── public
    │   └── src
    ├── docker-compose.yml
    └── server
        ├── Dockerfile
        ├── models
        ├── node_modules
        ├── package-lock.json
        ├── package.json
        └── server.js

```

Keeping the `docker-compose` at a level higher than the services that it uses means it's easier to add more services later on, and have docker-compose orchestrate them all.

### Run it

`docker-compose build` now builds all our images. Looks through any services in the docker-compose file that have a `build` attribute and builds them accordingly.

In our case it just builds the Dockerfile we've pointed out in `./bucket_list_service/` directory as the place to build the image for the bucket_list_service.

For mongodb, we're using the vanilla official mongo docker image which is already built for us. So we can just ask to pull and use that.


`docker-compose up` is the command we use to ask it to run all these containers at once.

**We now find some problems when starting the app.**

These are:

* The bucket_list_service starts really quickly and tries to connect to mongodb before it's even ready!
* The bucket_list_service doesn't find the mongodb service at all.

**We can fix these by doing:**

In `server.js` we need to ask node.js to exit when it doesn't connect to mongodb. Doing this means **the container will fail too.** Then it should immediately restart becuase we have `restart: on-failure` option on that service outline in `docker-compose.yml`.
Doing this it does restart each time, but we still have problems finding it all.

We will fix this in the next section!

## Networking with Docker

When we have multiple containers Docker creates a virual network for them all to find each-other. We do still need to explicitly link them, but the network is created even if there are no links.

### Containers have domain names

Very useful that we don't need to know the ip of a container, instead Docker gives it a domain name available within it's own virtual network. The name is based on the provided `container_name`. In our case `bucket_list_service`.

If we look into `server.js`, the express app is still trying to connect to mongodb locally ( on `localhost`). **localhost** now means that individual container... but our mongod runs in a separate container.

```javascript
// server.js
mongoose.connect('mongodb://localhost:27017/signups')
```
There is no MongoDB server running on localhost within that container, so we need to ask it to connect to the `mongoservice` container instead.

Change this to:

```javascript
// server.js
mongoose.connect('mongodb://mongoservice:27017/signups')

```



Build:

```bash
docker-compose build
```

Launch:

```bash
docker-compose up
```

#### Other Docker Compose options

It's possible to run docker-compose in detached mode:

```bash
docker-compose up -d
```
It's possible to ask it to build before bring-up:

```bash
docker-compose up --build -d
```
Once some containers are running in docker-compose we can see them and check their status:

```bash
docker-compose ps
```
it will show output like:

```
       Name                      Command               State            Ports
---------------------------------------------------------------------------------------
bucket_list_service   npm start                        Up      0.0.0.0:3000->3000/tcp
mongoservice          docker-entrypoint.sh mongo ...   Up      0.0.0.0:27017->27017/tcp
```
To bring containers down:

```bash
docker-compose down

```


Try it:
[http://localhost:3000/](http://localhost:3000/) IT WORKS!


We can still connect to the mongodb with Compass as if it were on `localhost`.


------

## Recap so far

So far we've managed to:

* **Learned that containers are small operating systems running independently within a host.**
* **Create a  docker image for our bucket-list express app.**
* **Create a docker-compose that allows building and running of the bucket-list image along with a pre-built mongod image.**


## Run the front-end server in a container

We need somewhere to serve the front-end from. For this we have a couple of options:

* use express static serve to serve it from where the back-end runs. This is a nice solution, as it doesn't cause any CORS issues, but not increddibly robust.
* Serve it on a totally different server on a different domain. This is common practice these days but will lead to CORS issues that we'll need to configure around in the server.
* Host it in another container on our machine using NGINX

## NGINX

**Nginx** is a hiligly configurable web-server. It's a very robust server for use in production environments for many different things. We will run a Docker container with NGINX running in it. We'll give it the static files from our front-end to serve up from there.

```
#docker-compose.yml
nginx:
    image: nginx:latest
    container_name: nginx
    restart: always
    volumes:
      - ./client/dist/:/usr/share/nginx/html
    ports:
      - 8080:80

```

The location `/usr/share/nginx/html` is where the nginx container will serve its static web content from.

We therefore need to put our built Vue app into `./client/dist` so that it can be served into that container under `/usr/share/nginx/html` - which will be picked up by nginx and hosted from there under port `80`.

**Now** brining the services down, re-building, and **updating** them will launch the nginx container, too.

```
docker-compose down
docker-compose build
docker-compose up -d
```

We should be able to go to `localhost:8081` - but this won't show anything as it's not got anything populated in the host folder.

```
cd client
npm run build
```

This will put the built client code into `dist`. From here it will be picked up by the nginx container so it can get hosted.

**Update the running container** We need to restart **nginx** for this to work.. so now do

```
docker-compose down
docker-compose build
docker-compose up -d
```

Refresh `http://localhost:8081`.



## Update the front-end to point to the containers rather than localhost

-----------

## Configuration Managment + Deployment

We've got a few problems to solve before we think about deploying this to DigitalOcean.

* We need to change the `MongoClient.connect('mongodb://mongoservice:27017')
` line depending on whether we're developing or deploying to production.
* We need to ensure that the webpack bundler has done it's job so the bundle is up to date.
* We need something that will help us actually ask our remote server to build and run the containers remotely.

We can solve these problems with **Configuration Management** We will specifically use **Ansible Playbook**.

Ansible is a tool for doing exactly that. It allows us to run commands remotely on a remote server. Just needs SSH.

First we need somewhere to run this.. lets use Digital Ocean..

### Setting up a new 'Droplet'

**Add our ssh key to DigitalOcean**
* Click avatar top right > Account
* Security > SSH keys > Add SSH Key
* `pbcopy < ~/.ssh/id_rsa.pub` and paste into window on Digital Ocean
* Give name and save.

**Creating the droplet**

In the DO Dashboard do:

* Create > Droplet
* One-click-apps > Docker
* Choose a size > $5/month
* Choose a region > London
* Add SSH Key
* Choose a hostname >  pick something better
* Press Create
* It will take a minute to create.
* Copy the IP Address once it's ready

**Connect remotely and setup a user**

We now need to remote into that droplet and check it has everything we need.

```bash
ssh root@<your droplets IP>
```

Check we have docker installed with `docker` and `docker-compose`. Looks good.

**We will now add a user who has the correct privelages**

* `adduser appuser`
* Put in a password but nothing else, just press enter...
* `usermod -aG sudo appuser`
* `su - appuser`  - become appuser for a minute
* `sudo ls -la /root` - enter the password and should list stuff - all good.
* `exit` - back to root user
* `cp -r /root/.ssh /home/appuser`
* `chown -R appuser:appuser ~appuser/.ssh` - copy the key to the user
* `exit`
* We should now be able to `ssh appuser@<droplet ip>` and get in without any hassle.


**Install ansible**

* `sudo apt update`
* `sudo apt install software-properties-common`
* `sudo apt-add-repository ppa:ansible/ansible` - press enter
* `sudo apt update`
* `sudo apt install ansible`
* `sudo apt intstall python python-pip`

**Enable docker for appuser**

* `sudo usermod -aG docker ${USER}`


**Create the ansible config locally**

Create a directory called `deployment`. We will put all deployment related configs in here.

In deployment create a `bucketlistservice.yml`.


```yaml

- hosts: bucketlistbackend
  tasks:
    - name: Copy app-services files across
      synchronize:
        src: ../services/
        dest: /home/appuser/services
        mode: push
        rsync_opts:
            - "--exclude=node_modules"

    - name: Bring up the rest of the services
      command: docker-compose up -d --build
      args:
        chdir: /home/appuser/services/

```

Create a `hosts` file in the `deployment` directory (same directory as the `bucketlistservice.yml`.

```
[bucketlistbackend]
<your server's ip>
```


**Deploy with Ansible**
```bash
ansible-playbook -i hosts -u appuser bucketlistservice.yml
```

Now check the app is working: `<server ip>:3000` in the browser.
