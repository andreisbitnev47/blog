---
layout: post
title: "2-MEAN app with Docker"
subtitle: "Dockerizing the MEAN app"
date: 2018-03-09 12:00
author: "Andrei Sbitnev"
header-img: "/img/mean_code.png"
cdn: 'header-off'
tags:
  - Docker
	- MEAN
---
## Introduction

&nbsp; In this tutorial, we will configure Docker containers for the MEAN app, we created in the previous blog post [1-Angular 5 MEAN app](/2018/02/22/1-mean-app/). Using Docker in development as well as in production has a lot of advantages:
<ul>
<li>The project will run in the same environment during development as well as in production. No matter what hardware or operating system is used, Docker containers will behave the same on all machines.</li>
<li>Deployment process will be a lot quicker and less painfull</li>
<li>Setting up the project on the developers machine takes only a few minutes, and as easy as running one command in the terminal</li>
</ul>

## Quick Start

To run any docker container, [Docker](https://docs.docker.com/install/#time-based-release-schedule) must first be installed on your system. Just choose your operating system, and follow the installation instructions. Use the CE version!<br>
In this tutorial we will also use the docker-compose, if you\`re using <b>Docker for windows</b> or <b>Docker for mac</b>, it is already installed with the package. Linux users should install it separately, here\`s a [link](https://docs.docker.com/compose/install/)<br>

As stated above, we will be building Docker containers for the MEAN app created in the previous tutorial. To get the app clone it from the [github repo](https://github.com/andreisbitnev/smart-house/tree/1-mean-app-base) 
```bash
git clone -b 1-mean-app-base https://github.com/andreisbitnev/smart-house.git
```
## Small changes to the code
Before we start creating our docker containers, we need to make some changes to the code. Go to ~/smart-house/be/app.js and modify the mongodb connection url
```javascript
...
var app = express();

mongoose.connect('mongodb://mongodb/test');
...
```
>Tip: The right way to communicate between the containers is by using their names instead of an ip address. Although it is possible for example to send a request from frontend to backend using `localhost:3000/test`, but it is usually not possible to access the database containers in the same way.

We also need to modify the ~/smart-house/fe/package.json file. Change the dev script to

```javascript
...
"dev": "ng serve --host 0.0.0.0",
...
```
The default host for angular cli dev server is `localhost`, here we set it to `0.0.0.0`, for the app to be accessible outside the container.<br>

Let\`s add a dev script to the backend package.json file<br>
~/smart-house/be/package.json
```json
...
"scripts": {
    "start": "node ./bin/www",
    "dev": "nodemon ./bin/www"
  },
...
```

## Dockerfile for the backend app
Docker containers are built from images. You can either use an already existing image, or create your own. To create a new image, Dockerfile is used, which is basically a blueprint for the image. There we specify all the tools needed to be set up for the app to run, as well as copy all the project files.
Let\`s first create a Dockerfile for the backend app. Go to ~/smart-house/be and create a new Docker file<br>
~/smart-house/be/Dockerfile
```Dockerfile
FROM node:9-alpine

WORKDIR /usr/src/project/
COPY . .
RUN rm Dockerfile
RUN npm install -g nodemon \
    && npm install
EXPOSE 9229
CMD ["npm", "run", "dev"]
```
Let\`s go line by line and see what each line do.
```Dockerfile
FROM node:9-alpine
```
Here we specify the base image for the new image. `node` - is the name of the image, and `9-alpine` is the tag, or the version so to say.
Here we are using the alpine distribution of linux, which has node.js version 9.6.1 installed on it. 
>Tip: alpine - is a very lightweight linux distribution, which doesn\`t have a lot of tools installed, like for example ubuntu has. But it has everything needed to run an express or angular app, and the image weighs only <b>68 MB</b>, compared to ubuntu image - <b>679 MB</b>

>Tip: You can check out other available images to build from on [hub.docker.com](https://hub.docker.com/).

```Dockerfile
WORKDIR /usr/src/project/
```
Here we set up the working directory for the project inside the Docker image. It is basically the same as `cd /usr/src/project/`, but the directories will get created if they don\`t exist.
```Dockerfile
COPY . .
```
Here we copy everything from the directory Dockerfile is in (~/smart-house/be) to the working directory in the image (/usr/src/project/)
```Dockerfile
RUN rm Dockerfile
```
Here we just remove the unnecessary files. Another option is not to copy everything in the previous step, but copy only the needed files one by one.
```Dockerfile
RUN npm install -g nodemon \
    && npm install
```
Here we install [nodemon](https://github.com/remy/nodemon) globally to our image, to later use it in development. And we also install the package dependencies.
```Dockerfile
EXPOSE 9229
```
Here we expose the 9229 port, which can then be used to debug the application.
```Dockerfile
CMD ["nodemon", "bin/www"]
```
This is the command, that will be run when a container, built from the image, starts. We want to run the server with nodemon to be able to make changes, without the need to restart the container<br>

## Dockerfile for the frontend app
Let\`s now create a Dockerfile for the frontend app. Go to ~/smart-house/fe and create a new Docker file. It will be basically the same as the previous one, with some small changes.<br>
~/smart-house/fe/Dockerfile
```Dockerfile
FROM node:9-alpine

WORKDIR /usr/src/project/
COPY . .
RUN rm Dockerfile \
    && npm install -g @angular/cli --unsafe \
    && npm install
EXPOSE 4200
CMD ["npm", "run", "dev"]
```
We don\`t need a nodemon for an angular app, but we do need angular cli
```Dockerfile
&& npm install -g @angular/cli --unsafe \
```
>Tip: the --unsafe tag is a workaround, needed to install the angular cli inside the Docker image. There is an open [bug report](https://github.com/angular/angular-cli/issues/7389) on the issue. This workaround is the cleanest from currently suggested.

```Dockerfile
EXPOSE 4200
```
Here we expose the port, that will be used to serve the application.

## Building the images
We can now build our images, and create Docker containers from them. We will also need a mongodb container to store our data. If you have not yet created it in the previous tutorial, do it by running
```bash
sudo docker container run --name mongodb -d mongo
```
>Tip: If you\`re on a windows environment, be sure to run the command in powershell

Here we simply specify the name of the container - <b>mongodb</b>, and the image to build it from - <b>mongo</b>, by default the <b>:latest</b> tag is used, so it\`s the same as mongo:latest. We also specify that the process should run in detached mode <b>-d</b>, and setup port forwarding with <b>-p</b></br>
Now let\`s build a backend image.
```bash
sudo docker image build -t sm-be ./be
```
I\`m running this command from ~/smarthouse directory, so I specify the relative path `./be` to the Dockerfile directory I want to build from. `-t sm-be` - is the Tag, or the name of the image.
>Tip: Now you can run the `sudo docker image ls` to see the list of all available images, and sm-be should be there

Let\`s do the same for the frontend image
```bash
sudo docker image build -t sm-fe ./fe
```

## Creating containers
To create the backend app container run the next command
```bash
sudo docker container run -d --name smart-house-be -p 3000:3000 --link mongodb:mongodb -v "$(pwd)"/be:/usr/src/project -v /usr/src/project/node_modules sm-be
```
`-d` - means detached mode, `--name smart-house-be` - is the name of the container, `-p 3000:3000` - port forwarding, our network port : containers port.<br> 
`-v "$(pwd)"/be:/usr/src/project` - here we map our local volume ~/smart-house/be to containers /usr/src/project. So now whatever changes we make in our project folder will automatically apply to container.<br>
`-v /usr/src/project/node_modules` - here we specify a new volume for node_modules. In the previous step we mapped our local project folder to the project folder inside the container. Since node_modules is also located inside the project folder, the container will look for it in our local environment. This is not the best choise, because our local environment might be different from the one inside the container. So we want node_modules to be maintained inside the container, and we don\`t even need it in our local environment.<br>
`--link mongodb:mongodb` is a link to the mongodb container.
`sm-be` is the name of the image we want to create the container from.

Let\`s now create the frontend app container
```bash
sudo docker container run --name smart-house-fe -p 4200:4200 -d -v "$(pwd)"/fe:/usr/src/project -v /usr/src/project/node_modules sm-fe
```
Now open in the browser [localhost:4200](http://localhost:4200/), you should see the app working.
<img src="/img/finished.png">
You can now make any changes in the angular app, and the dev server will update them automatically. You can also make changes in the backend app, and the nodemon will update them automatically.

>Tip: to start and stop any comtainer just use `sudo docker container start <container_name>` or `sudo docker container stop <container_name>`

## Using docker-compose
We have succesfully built our images and created containers for our MEAN app, but now we need to start and stop them one by one. Also if for example other members of the team were to use our new build, they would have to enter all of those commands listed above manually. It would be great to automate the process. That\`s where <b>docker-compose</b> comes in. Docker-compose simplifies the build process of group of containers, that are used together. Go to ~/smart-house directory and create a docker-compose.yml file<br>
~/smart-house/docker-compose.yml
```yml
version: '2'
services:
  mongodb:
    image: mongo
    volumes:
      - mongo-volume:/home/node
  smart-house-fe:
    build: ./fe
    command: npm run dev
    volumes:
      - ./fe:/usr/src/project
      - /usr/src/project/node_modules
    ports:
      - 4200:4200
  smart-house-be:
    build: ./be
    command: npm run dev
    volumes:
      - ./be:/usr/src/project
      - /usr/src/project/node_modules
    ports:
      - 3000:3000
      - 9229:9229
volumes:
  mongo-volume:
```
From the first look, you can see that the file basically lists everything we specified in the scripts above, but now all the information is structured. Lets go line by line and see what each line does.
`version: '2'` - the version of docker-compose file, the latest is 3.1, 
`services` - contains all the containers, that need to be created,
`mongodb, smart-house-fe, smart-house-be` - the names of containers,
`image` - the base image to use,
`volumes` - a list of volumes for each container. As you may have noticed, we added a volume to our mongodb container called `mongo-volume`. It is always a good idea to add a named volume to the database. If you later on remove the container the volume will still exist, and you can attach it to a new container. This way the data stored in the database will not be lost,
`build` - relative path to the Dockerfile, from which the container should be built,
`command` - although we specify a command to run on container startup, we can rewrite it here.
`volumes` - a list of named volumes to be created.<br>
Let\`s now remove the containers and images we previously created and build everything again using just one command. Remove the smart-house-be container
```bash
sudo docker container rm -v -f smart-house-be
```
>Tip: `-v` - is to remove also the volumes attached to the container, `-f` - is to force it, if container is still running

and the smart-house-fe
```bash
sudo docker container rm -v -f smart-house-fe
```
and the mongodb
```bash
sudo docker container rm -v -f mongodb
```
Now the images<br>
`sudo docker image rm sm-be` and `sudo docker image rm sm-fe`<br>
<!-- Before we build everything once again, let\`s make one small change to che project. You may have noticed, that in the docker-compose.yml we no longer open up the 3000 port on the backend container. It is a better practice to call one container from another using their names, and not exposing ports to the outer network. This way our backend container won\`t be accessible outside the  -->
To build everything again just go to ~/smart-house and run `sudo docker-compose up`. Everything should be working as before

>Tip: You can also run it in detached mode `sudo docker-compose up -d`. To stop all the containers, run `sudo docker-compose down`. To remove all the volumes attached to the containers, run `sudo docker-compose down -v`

## Debugging node in visual studio code.
There is a lot of tools out there for debugging, but since I usually use visual studio code, I also debug the application with it.<br>
First we need to add a debug script to our backend package.json 
```json
...
"scripts": {
    "start": "node ./bin/www",
    "dev": "nodemon ./bin/www",
    "debug": "node --inspect-brk=0.0.0.0:9229 ./bin/www"
  },
...
```

Next, lets change the command that starts the backend app in docker-compose file<br>
~/smart-house/docker-compose.yml
```yml
...
smart-house-be:
    build: ./be
    command: npm run debug
    volumes:
      - ./be:/usr/src/project
...
```
 Finally lets add a configuration file to our visual studio code<br>
 ~/smart-house/.vscode/launch.json
```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "docker",
            "type": "node",
            "request": "attach",
            "port": 9229,
            "address": "127.0.0.1",
            "restart": true,
            "sourceMaps": false,
            "localRoot": "${workspaceRoot}/be",
            "remoteRoot": "/usr/src/project",
            "outFiles": [],
            "protocol": "inspector"
        }
    ]
}
```
Next do the `sudo docker-compose down`, and `sudo docker-compose up`, and start a debugging session in your visual studio code.
<img src="/img/debug.png">

## Git repository
The Dockerized version of the app built in this tutorial is available in a github repo, on the [2-mean-app-docker](https://github.com/andreisbitnev/smart-house/tree/2-mean-app-docker) branch. You can git clone it with
```bash
git clone -b 2-mean-app-docker https://github.com/andreisbitnev/smart-house.git
```