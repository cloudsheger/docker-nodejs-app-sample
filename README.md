## Building your first Docker image

It’s time to get our hands dirty and see how Docker build works in a real-life app. We’ll generate a simple Node.js app with an Express app generator. Express generator is a CLI tool used for scaffolding Express applications. After that, we’ll go through the process of using Docker build to create a Docker image from the source code.

We start by installing the express generator as follows:
```
$ npm install express-generator -g
```
Next, we scaffold our application using the following command:
```
$ express docker-app
```
Now we install package dependencies:
```
$ npm install
```
Start the application with the command below:

```
$ npm start
```
If you point your browser to http://localhost:3000, you should see the application default page, with the text “Welcome to Express.”


## Dockerfile
Mind you, the application is still running on your machine, and you don’t have a Docker image yet. Of course, there are no magic wands you can wave at your app and turn it to a Docker container all of a sudden. You’ve got to write a Dockerfile and build an image out of it.

Docker’s official docs define Dockerfile as “a text document that contains all the commands a user could call on the command line to assemble an image.” Now that you know what a Dockerfile is, it’s time to write one.

At the root directory of your application, create a file with the name “Dockerfile.”
```
$ touch Dockerfile
```

#### Dockerignore
There’s an important concept you need to internalize—always keep your Docker image as lean as possible. This means packaging only what your applications need to run. Please don’t do otherwise.

In reality, source code usually contain other files and directories like .git, .idea, .vscode, or travis.yml. Those are essential for our development workflow, but won’t stop our app from running. It’s a best practice not to have them in your image—that’s what .dockerignore is for. We use it to prevent such files and directories from making their way into our build.

Create a file with the name .dockerignore at the root folder with this content:
```
# vi .dockerignore and add the folllowing 
.git
.gitignore
node_modules
npm-debug.log
Dockerfile*
docker-compose*
README.md
LICENSE
.vscode
```
### The base image
Dockerfile usually starts from a base image. As defined in the Docker documentation, a base image or parent image is where your image is based. It’s your starting point. It could be an Ubuntu OS, Redhat, MySQL, Redis, etc.

Base images don’t just fall from the sky. They’re created—and you too can create one from scratch. There are also many base images out there that you can use, so you don’t need to create one in most cases.

We add the base image to Dockerfile using the FROM command, followed by the base image name:
```
# Filename: Dockerfile
FROM node:10-alpine
Copying source code
Let’s instruct Docker to copy our source during Docker build:

# Filename: Dockerfile 
FROM node:10-alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
```
First, we set the working directory using WORKDIR. We then copy files using the COPY command. The first argument is the source path, and the second is the destination path on the image file system. We copy package.jsonand install our project dependencies using npm install. This will create the node_modules directory that we once ignored in .dockerignore.

You might be wondering why we copied package.json before the source code. Docker images are made up of layers. They’re created based on the output generated from each command. Since the file package.json does not change often as our source code, we don’t want to keep rebuilding node_modules each time we run Docker build.

Copying over files that define our app dependencies and install them immediately enables us to take advantage of the Docker cache. The main benefit here is quicker build time. There’s a really nice blog post that explains this concept in detail. 

Want to improve your code? Try Stackify’s free code profiler, Prefix, to write better code on your workstation. Prefix works with .NET, Java, PHP, Node.js, Ruby, and Python.

### Exposing a port
Exposing port 3000 informs Docker which port the container is listening on at runtime. Let’s modify the Docker file and expose the port 3000.
```
# Filename: Dockerfile 
FROM node:10-alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
```
### Docker CMD
The CMD command tells Docker how to run the application we packaged in the image. The CMD follows the format CMD [“command”, “argument1”, “argument2”].
```
# Filename: Dockerfile 
FROM node:10-alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```
### Building Docker images
With Dockerfile written, you can build the image using the following command:
```
$ docker build .
```
docker build
We can see the image we just built using the command docker images.
```
$ docker images
```
```
REPOSITORY TAG IMAGE ID CREATED SIZE
<none> <none> 7b341adb0bf1 2 minutes ago 83.2MB
```
### Tagging a Docker image
When you have many images, it becomes difficult to know which image is what. Docker provides a way to tag your images with friendly names of your choosing. This is known as tagging.
```
$ docker build -t yourusername/repository-name .
```
Let’s proceed to tag the Docker image we just built.
```
$ docker build -t yourusername/example-node-app
```
If you run the command above, you should have your image tagged already.  Running docker images again will show your image with the name you’ve chosen.
```
$ docker images
```
### REPOSITORY TAG IMAGE ID CREATED SIZE
```
cloudsheger/nodejs-app latest be083a8e3159 7 minutes ago 83.2MB
```
### Running a Docker image

You run a Docker image by using the docker run API. The command is as follows:
```
$ docker run -p80:3000 yourusername/example-node-app
```
The command is pretty simple. We supplied -p argument to specify what port on the host machine to map the port the app is listening on in the container. Now you can access your app from your browser on https://localhost.

To run the container in a detached mode, you can supply argument -d:
```
$ docker run -d -p80:3000 yourusername/example-node-app
```
A big congrats to you! You just packaged an application that can run anywhere Docker is installed.

Pushing a Docker image to Docker repository
The Docker image you built still resides on your local machine. This means you can’t run it on any other machine outside your own—not even in production! To make the Docker image available for use elsewhere, you need to push it to a Docker registry.

A Docker registry is where Docker images live. One of the popular Docker registries is Docker Hub. You’ll need an account to push Docker images to Docker Hub, and you can create one here.

With your Docker Hub credentials ready, you need only to log in with your username and password.
```
$ docker login
```
Retag the image with a version number:

```
$ docker tag yourusername/example-node-app yourdockerhubusername/example-node-app:v1
```
Then push with the following:
```
$ docker push cloudsheger/example-node-app:v1
```
If you’re as excited as I am, you’ll probably want to poke your nose into what’s happening in this container, and even do cool stuff with Docker API.

You can list Docker containers:
```
$ docker ps
```
And you can inspect a container:
```
$ docker inspect <container-id>
```
You can view Docker logs in a Docker container:
```
$ docker logs <container-id>
```
And you can stop a running container:
```
$ docker stop <container-id>
```
Logging and monitoring are as important as the app itself. You shouldn’t put an app in production without proper logging and monitoring in place, no matter what the reason.
