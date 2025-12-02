# jenkins-pipeline-cxx


## Build a Docker Image

We can build an image from the `Dockerfile` that can be run as a container 
just like an image pulled from Docker Hub.

```bash
# Build the Docker image with the tag "hello-cmake"
$ docker build -t file_service .

# List local images to confirm the image was created
$ docker image ls

# Show the layers and build steps of the hello-cmake image
$ docker image history file_service
```

### Create Docker Container

```bash
# Run the image (container removed after exit with --rm)
$ docker run --rm file_service

# remove the local Docker image
$ docker image rm file_service
```

