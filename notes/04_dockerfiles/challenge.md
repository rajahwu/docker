# Challenge

## Build you own Docker image

* A Go application is included in the [exercise files](https://www.linkedin.com/learning/docker-essential-training/challenge-build-and-run-your-first-image?autoSkip=true&resume=false)
* A Dockerfile to build and package this application
    is included...but it is broken
* **Complete the Dockerfile provided to build an image for it and run it in a container**

## Command Order

* **First, build:** docker build -t test-app .
* **Then run:** docker run --rm test-app

## Tips

* **You do not need to know Go or modify the source code to complete this challenge**
* Remember that **docker build** needs context and that layer order matters!
* Consult the [Dockerfile reference](https://docs.docker.com/engine/reference/builder/) doc if you get stuck

## Extra Credit

* Go applications are completely self-contained and can rund without a root filesystem
* Use a multi-stage build to create your image from scratch with nothing buty the /test-app file
* This is a greate example of building small and highly secure Docker images!
