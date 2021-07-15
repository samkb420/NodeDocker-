# NodeDocker-
simple node app packaged with docker learnt in gcp 30 days challange


#gCP COMMANDS 

```
gcloud auth list


gcloud config list project




docker run hello-world
```


```
cat > Dockerfile <<EOF
# Use an official Node runtime as the parent image
FROM node:6

# Set the working directory in the container to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Make the container's port 80 available to the outside world
EXPOSE 80

# Run app.js using node when the container launches
CMD ["node", "app.js"]
EOF
```

```
cat > app.js <<EOF
const http = require('http');

const hostname = '0.0.0.0';
const port = 80;

const server = http.createServer((req, res) => {
    res.statusCode = 200;
      res.setHeader('Content-Type', 'text/plain');
        res.end('Hello World\n');
});

server.listen(port, hostname, () => {
    console.log('Server running at http://%s:%s/', hostname, port);
});

process.on('SIGINT', function() {
    console.log('Caught interrupt signal and will exit');
    process.exit();
});
EOF
```


#Build

```
docker build -t node-app:0.1 .
docker run -p 4000:80 --name my-app node-app:0.1

docker images

curl http://localhost:4000

docker run -p 4000:80 --name my-app -d node-app:0.1

docker ps
```

#change code 

```
res.setHeader('Content-Type', 'text/plain');
        res.end('Welcome to Cloud\n');
```
#build another version

```
docker build -t node-app:0.2 .

docker run -p 8080:80 --name my-app-2 -d node-app:0.2
docker ps

curl http://localhost:8080

Welcome to Cloud
```

## Debug 

```
docker logs -f [container_id]


Sometimes you will want to start an interactive Bash session inside the running container. You can use docker exec to do this. Open another terminal (in Cloud Shell, click the + icon) and enter the following command

docker exec -it [container_id] bash


root@xxxxxxxxxxxx:/app#

ls


exit


docker inspect [container_id]

Use --format to inspect specific fields from the returned JSON. For example:

docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' [container_id]


```
## Publish
Now you're going to push your image to the Google Container Registry (gcr). After that you'll remove all containers and images to simulate a fresh environment, and then pull and run your containers. This will demonstrate the portability of Docker containers.

To push images to your private registry hosted by gcr, you need to tag the images with a registry name. The format is [hostname]/[project-id]/[image]:[tag].

For gcr:

[hostname]= gcr.io
[project-id]= your project's ID
[image]= your image name
[tag]= any string tag of your choice. If unspecified, it defaults to "latest".
You can find your project ID by running:


gcloud config list project

```
docker tag node-app:0.2 gcr.io/qwiklabs-gcp-01-823e83c12ea2/node-app:0.

docker images


docker push gcr.io/[project-id]/node-app:0.2
```

Let's test this image. You could start a new VM, ssh into that VM, and install gcloud. For simplicity, we'll just remove all containers and images to simulate a fresh environment.

Stop and remove all containers:

docker stop $(docker ps -q)
docker rm $(docker ps -aq)

You have to remove the child images (of node:6) before you remove the node image. Replace [project-id].

docker rmi node-app:0.2 gcr.io/[project-id]/node-app node-app:0.1
docker rmi node:6
docker rmi $(docker images -aq) # remove remaining images
docker images


docker pull gcr.io/[project-id]/node-app:0.2
docker run -p 4000:80 -d gcr.io/[project-id]/node-app:0.2
curl http://localhost:4000
