
# Building first images with Docker

Building images is quite simple, especially when using a Dockerfile.

## Creating the project

For this example, we're going to create a _very_ basic Node app. The app is simply going to start an Express server that provides a simple "Hello World" app.

1. All node applications need a `package.json` file, as it outlines all dependencies that our application needs to run. Copy the JSON below into a file name `package.json` 

```json
{
  "name": "first-docker-app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "dependencies": {
    "express": "^4.15.4"
  }
}
```

2. Create a `src` directory
3. In the `src` directory, create a file named `index.js` and paste the following into it:

```javascript
const express = require('express');
const app = express();

app.get('/', function (req, res) {
  const name = req.query.name || "World";
  res.send(`Hello ${name}!`);
});

app.listen(3000, function () {
  console.log('Application is listening on port 3000!')
});
```


## Creating the Dockerfile

A Dockerfile is a simple text-based script that is used to build an image.  Since this application will be running in node, we will use the official [node](https://hub.docker.com/_/node) image on DockerHub.

1. Create a file named `Dockerfile` in the root of the project

2. In the `Dockerfile`, add the `FROM` statement, which indicates the image we are using as a base:
```dockerfile
FROM node:8.4
```

3. The first thing I like to do is set the working directory. In many applications, I simply use `/app` as my working directory. All future commands will then run from this directory.
```dockerfile
WORKDIR /app
```

4. The next thing we need to do is copy in our source! We'll take a naive approach now and just copy everything in one go (we'll talk about more effective images later).
```dockerfile
COPY ./ /app
```

5. In order to run a Node application, we need to install of the dependencies.  In case you're not familiar with how to do that, you use the Node Package Manager (npm).  The command is simply `npm install`. To run a command in the build script, we use the `RUN` instruction:
```dockerfile
RUN npm install
```

6. At this point, we have all of our source and its dependencies in the container. All we have to do is specify the default command that should be used when starting the container. To do this, we use the `CMD` instruction:
```dockerfile
CMD node src/index.js
```

### The final Dockerfile

```dockerfile
FROM node:8.4
WORKDIR /app
COPY ./ /app
RUN npm install
CMD node src/index.js
```


## Build the image!

Now that we have a project and a Dockerfile in place, let's build the image!

```
docker build -t my-first-node-app .
```

In case you're wondering, the final `.` specifies the build context location, which is where the Dockerfile and other files are to be found.


## Run your new image!

Now that the image is built, start it up!  The server listens to port 3000, so we need to expose that to the host. We'll add the `-d` flag to run the container in the background.

```
docker container run -d -p 3000:3000 my-first-node-app
```

Now, if you open to http://localhost:3000/, you should get a "Hello world!".

Try going to http://localhost:3000/?name=Moby and you should get a personalized message!

> **Congratulations! You've just now made a new HWGAAS (Hello World Generator as a Service)!**


## Push to DockerHub

1. Log in to Docker Hub (create an account if you need to).

2. Create a new repository named `my-first-node-app`. You now have a remote repo that you can push your image to!

3. In order to push to the repo, we need to authenticate. The credentials are stored securely in the Mac Keychain or the Windows credential manager.
```
docker login
```

4. All repos in DockerHub are namespaced to your user account. The image name also reflects this namespacing.  In order to push to the repo, we'll need to re-tag our image:
```
docker image tag my-first-node-app [your DockerHub username]/my-first-node-app
```

5. Once it's re-tagged, push the image!
```
docker image push [your DockerHub username]/my-first-node-app
```

6. Once it's pushed you should be able to go into the DockerHub web interface and see an image!


## Get a buddy to pull the image!

Find someone sitting next to you and ask them to pull your image and try it out.  They should be able to simply run:

```
docker container run -d -p 3000:3000 [buddy DockerHub username]/my-first-node-app
```

You should see the image pulled locally and it should then work!


## Challenge Round

Modify the source JavaScript file, rebuild the image, tag, and push. Then, have a buddy pull the image and see the change you made!