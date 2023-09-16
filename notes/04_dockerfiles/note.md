# Exploring Dockerfiles

## Introduction to Dockerfiles

* Learn aobut image manifests and the Dockerfile
* Create our own Dockerfile from scratch!
* Learn the **docker image** set of commands

### Automating Image Creation

* Write a shell script?
  * Complex and error-prone

* Use a configuration management tool?
  * More external dependencies to manage;
    also complex and error-prone

#### **The Dockerfile**

 **Dockerfiles:** The Dockerfile is a language for
    easily building container images.

## The anatomy of a Dockerfile

* Dockerfiles consist of a series of instructions.

* The instruction is the first word of the statement;
    every other word after the instruction is provided to it as arguments

```dockerfile
FROM ubuntu
```

* Docker runs each Dockerfile command in an
    "intermediate container" and saves the results
    as an image layer.

* These layers are joined together to form your image as the Dockerfile is processed.

### common commands

**docker image** and **docker run** commands

## Sourcing other Docker images with **FROM**

**FROM** must be the first command in the Dockerfile!*

```text
*Comments, ARGs, and ENVs can come before the FROM line
```

The **FROM** command describes the "base" image
    that this Dockerfile's image will
    be created from.

Layers from the base image are linked underneath your new image.

```dockerfile
FROM ubuntu
```

The **FROM scratch** command creates your own base image,
    an advanced technique.

### Four Ways to Use FROM

| Form | Description | Example |
|------|-------------|---------|
|FROM $image |Selects the "latest" tag of **$image**|FROM ubuntu|
|FROM $image:$tag |Selects the **$tag** tag of **$image**|FROM ubuntu:kinetic|
|FROM $image AS $name |Selects the "latest" tag of **$image** and gives it an alias **$name** (multi-stage builds only)|FROM ubuntu AS base|
|FROM $image:$tag AS $name |Selects the **$tag** tag of **$image** and gives it an alias **$name** (multi-stage builds only)|FROM ubuntu:kinetic AS base|

## Adding and copying files with **COPY** and **ADD**

COPY add files into Docker images from a provided context.

ADD is the same as COPY but also works with URLs to TAR files.

Use **COPY** instead of ADD wherever possible!

### Three Ways to Use **COPY**

|Form|Description|Example|
|----|-----------|-------|
|COPY $src-file $dest-file|Copies a file **$src-file** into the container as file **$dest-dir**| COPY ./my-file.txt /app/my-file.txt|
|COPY $src-file $dest-dir|Copies a file **$src-file** into the container as directory **$dest-dir**| COPY ./my-file.txt /app/|
|COPY $src-dir $dest-dir|Copies a directory **$src-dir** into the container as directory **$dest-dir**| COPY ./my-dir /app/|

### **COPY** Wildcards

|Wildcard|Description|Input|Command|Matches|
|--------|-----------|-----|-------|-------|
|?|Copies a file **src-file** into the container in directory **$dest-dir**|songs/song-1.mp3 songs/song-2.mp3 songs/song-3.mp3 songs/song1.mp3 songs/song2.mp3 songs/song3.mp3|COPY songs/song-?.mp3 /songs| song/song-1.mp3 song/song-2.mp3 song/song-3.mp3|
|*|Matches all characters after **word**(until next non-asterisk(character)|songs/song-1.mp3 songs/song-2.mp3 songs/song-3.mp3 songs/song1.mp3 songs/song2.mp3 songs/song3.mp3|COPY songs/song* /songs|songs/song-1.mp3 songs/song-2.mp3 songs/song-3.mp3 songs/song1.mp3 songs/song2.mp3 songs/song3.mp3|

### Two Userful **COPY** Arguments

|Form|Description|
|----|-----------|
|--chown $user:$group|Assigns **$user** in group **$group** as owner of the file or directory **in the container image**|
|--link|Copies files or directory from context into a blank layer|

## Customizing your Docker image with **RUN**

RUN executes commands within temporary container.

### Two Ways to Use **RUN**

|Form|Description|Example|
|----|-----------|-------|
|RUN command-in-shell-form|**Shell form**: sends everything after **RUN** to /bin/sh or the image's **ENTRYPOINT**| RUN echo "Hello, world!"|
|RUN [ "command", "in", "exec", "form ]|**Exec form**: first word after **RUN** is the programe to be executed. All other words are provided as arguments to the program|RUN ["echo","Hello, world!"]

## Starting your app with **ENTRYPOINT**

The **ENTRYPOINT** Command

ENTRYPOINT configures containers to run stuff

### Two Ways to Use **ENTRYPOINT**

|Form|Description|Example|
|----|-----------|-------|
|ENTRYPOINT command-in-shell-form|**Shell form:** sends everything after **ENDPOINT** to /bin/sh **/bin/sh $command is PID 1.**|ENTRYPOINT echo "Hello, world!"|
|ENTRYPOINT [ "command", "in", "exec", "form" ]|**Exec form:** first word after **ENTRYPOINT** is the program to be executed. All other words are provied as arguments to the program. **Your program is PID 1.**|

#### **Shell Form**

* App runs as a child of **/bin/sh** or **cmd**
* Any signals sent to the app are caught by the shell, not the program
* **Any code in your app that relies on signals will not run!**
* **Arguments sent to containers will get passed into the shell, not your program!**

```dockerfile
# shell form
FROM ubuntu
COPY ./app
RUN apt -y update && apt -y install curl
ENTRYPOINT "SOME VAR=$SOME VALUE_FROM_DOCKER; [ "$SOME_VAR" == "foo" ] && \/app/app.sh --foo || /app/app.sh --bar"
```

#### **entrypoint script pattern**

```dockerfile
# entrypoint script
FROM ubuntu
COPY ./app
RUN apt -y update && apt -y install curl
ENTRYPOINT [ "/app/entrypoint.sh]

```

```bash
# /app/entrypoint.sh
# !#/usr/bin/env bash
SOME VAR=$SOME VALUE_FROM_DOCKER;
[ "$SOME_VAR" == "foo" ] && /app/app.sh --foo || /app/app.sh --bar
```

#### **Exec Form**

* App runs as the top-level process within the container (as PID 1)
* **Any signals sent to the app are sent to your program**
* **Arguments sent to containers will get passed into your program**

## "Starting your app with **CMD**

CMD configures containers to run stuff...sort of.

### CMD's Behavior Depends On

* Whether an **ENTRYPOINT** is provided or not
* Whether **CMD** is written in shell or exec form

#### ...Sort Of?

|Scenario|Example|Result|
|--------|-------------------|------|
|ENTRYPOINT missing, CMD provided| FROM ubuntu COPY ./app CMD ["/app/app.sh"]|**/app/app.sh** is run as PID 1. Arguments provided to it are sent to **/app/app.sh** (remember, shell vs. exec form!).|
|ENTRYPOINT provided, CMD provided |FROM ubuntu COPY ./app ENTRYPOINT ["/app/app.sh"] CMD ["--argument"]|**/app/app.sh** is run as PID 1. **--argument** is provided as argument to **/app/app.sh**. Additional arguments provied to it are sent to /app/app.sh|
| ENTRYPOINT provided, CMD missing | FROM ubuntu COPY ./app ENTRYPOINT ["/app/app.sh] | **/app/app.sh** is run as PID 1. Additional arguments provided to it are sent to **/app/app.sh**.|
|ENTRYPONIT missing, CMD missing|FROM ubuntu COPY ./app|CMD or ENTRYPOINT is inherited from base image, or /**bin/sh 0c** or **cmd/S/C** is used as default ENTRYPOINT|

### What What about SHell Form

| Scenario | Example | Result |
|----------|---------|--------|
|ENTRYPOINT missing, CMD provided | FROM ubuntu COPY ./app CMD "/app/app.sh" | **/app/app.sh** is run through **/bin/sh** or **cmd/S/C**. Arguments provided to it are sent to **sh** and most likely dissolved int the ether.|
|ENTRYPOINT provided, CMD provided | FROM ubuntu COPY ./app ENTRYPOINT "/app/app.sh" CMD ["--argument"] | **/app/app.sh** is run through **bin/sh** or **cmd/S/C**. Arguments provided to it are are sent to **sh** and most likely disolved int the ether. |
| ENTRYPOINT provided, CMD missing | FROM ubuntu COPY ./app ENTRYPOINT "/app/app.sh" | **/app/app.sh** is run through **/bin/sh** or **cmd/S/C**. Arguments provieded to it are sent to **sh** and most likely dissolved into the ether. |
| ENTRYPOINT missing, CMD missing | FROM ubuntu COPY ./app | CMD or ENTRYPOINT is inherited form base image, or / **bin/sh** or **cmd/S/C** is used as default ENTRYPOINT.

## Adding variables with **ENV** adn **ARG**

 Build arguments and environment variables.

 **ARG** allows us to set a variable at build time

 ```dockerfile
 # This is a testing image
FROM ubuntu
ARG curl_bin=curl
COPY . /app
RUN apt -y update && apt -y install "$curl_bin"
CMD [ "--argument" ]
ENTRYPOINT [ "/app/app.sh" ]
 ```

### You set **ARG** variables with **docker build** with teh **--build-arg** flag

**docker build** won't fail if you forget to use **--build-arg,** but your build might break ...

**ENV** configures environment variables for containers started from this image

 ```dockerfile
 # This is a testing image
FROM ubuntu
ENV curl_bin="curl=7.85.0"
COPY . /app
RUN apt -y update && apt -y install "$curl_bin"
CMD [ "--argument" ]
ENTRYPOINT [ "/app/app.sh" ]
 ```

Both **ENV** and **ARG** allow you to set defaults.

**ENV** variables live with every container, **ARG** variable do not

You set **ARGs** at built time: you *override* **ENVs** at run time

Both **ENV** and **ARG** can only be expanded within **RUN** commmands

**ENVs** and **ARGs** used by **RUN** commands *must* precede the **RUN** command that reference them

## Other helpful Dockerfile commands

[dockerfile reference](https://docs.docker.com/engine/reference/builder/)

[buildkit](https://docs.docker.com/build/buildkit/)

### **LABEL**

* Adds documentation for your image
* Accepts a key-value par

```dockerfile
LABEL maintainer="Rajah Wu <dev@rajahwu.me>"
LABEL foo=bar
LABEL baz=quux
```

### **WORKDIR**

* Set a working directory for **RUN** commands
    and/or containers
* Can use **WORKDIR** multiple times to change the
    working directory while building

```dockerfile
WORKDIR /
RUN pwd
WORKDIR /app
ENTRYPOINT [ "./app.sh" ]
```

### **USER**

* Set the **Linux or Windows** user to be used
    for **RUN** commands and/or containers
* Linux users can be numerical as well
* Can use **USER** multiple times to change the
    working directory while building
* The **last** USER will be used by containers
* Useful for preventing containers from running as
    root by default
* Can be overriden with **docker run --user**
* Can create users within the Dockerfile for
    containers to run as later!

```dockerfile
USER root
RUN apt -y update && apt -y install "$curl_bin"
RUN useradd -p supersecret newuser
CMD [ "--argument" ]
USER newuser
ENTRYPOINT [ "/app/app.sh" ]
```

### **EXPOSE**

* Documents ports that containers created from image
    should expose
* **Does not automatically expose ports**
* Useful for documentation; completely optional

```dockerfile
FROM ubuntu
COPY ./app
ENV curl_bin=curl
RUN apt -y update && apt -y install "$curl_bin"
CMD [ "--argument" ]
ENTRYPOINT ["/app/app.sh"]
EXPOSE 8080
```

## Multi-stage Builds

The app in our example is pretty big

```bash
docker images my-image
# REPOSITORY  TAG     IMAGE ID        CREATED         SIZE
# my-image    latest  9b7c1ff6559d    2 minutes ago   113MB
```

```bash
./size_diff.sh
# App size, in bytes: 876
# App image size, in bytes: 118489088
# Size difference: 135261x
```

We could create two Dockerfiles and use the **builder pattern** to
    make our image slimmer and faster ...

    ```bash
    docker create --name get-date-text my-image && 
    docker cp get-dat-text:/app/include/date.txt . &&
    docker build -t my-smaller-image -f smaller.Dockerfile .
    ```

### **Bulider pattern** Disadvantages

* Increases complexity by producing more Dockerfiles
    than needed
* Increases fragiliy by having more commands than needed
* Leaves several unused artifacts behind (**get-date-txt** container and old app image)

Multi-stage builds use intermediate images to produce smaller final images.

```dockerfile
FROM ubuntu
ENV curl_bin="curl"
RUN apt -y update && apt -y install "$curl_bin"
RUN curl -i -sS google.com | \
    grep -E '^Date:' | \
    sed 's/^Date: //' | tr -d $'\r' > '/tmp/date.txt'

FROM bash:alpine3.16
COPY . /app
COPY --from=0 /tmp/date.txt /app/include/data.txt
ENTRYPOINT [ "/app/app.sh" ]
CMD [ "--argument" ]
```

### **Multiple Base Images**

Multi-stage builds can use multipe base images
This produces significanty smaller final images
    and accelerates build time through
    imporved caching

```dockerfile
FROM ubuntu
# ...
FROM bash:alpine3.16
```

### **Stages**

Each group of commands under a base image
    is called a **stage**

#### Stage 0

```dockerfile
FROM ubuntu
ENV curl_bin="curl"
RUN apt -y update && apt -y install "$curl_bin"
RUN curl -i -sS google.com | \
    grep -E '^Date:' | \
    sed 's/^Date: //' | tr -d $'\r' > '/tmp/date.txt'
```

#### Stage 1 (Your Final Image)

```dockerfile
FROM bash:alpine3.16
COPY . /app
COPY --from=0 /tmp/date.txt /app/include/data.txt
ENTRYPOINT [ "/app/app.sh" ]
CMD [ "--argument" ]
```

### **COPY --from**

You can copy files or directories from one temporary image
    to anoter with the **--from** COPY flag

```dockerfile
# ...
COPY --from=0 /tmp/date.txt /app/include/data.txt
# ...
```

### **Aliases**

You can also name your stages, which makes referencing 
    them in later COPY operations easier

```dockerfile
FROM ubuntu AS base
# ...
FROM bash:alpine3.16 AS app
# ...
COPY --from=base /tmp/date.txt /app/include/data.txt
# ...
```

### **Other Features**

You can also reference external images in **COPY --from** commands
    and reuse stages as many times as you'd like.

```dockerfile
FROM ubuntu AS base
# ...
...
FROM base AS second
RUN echo 'Re-using base a second time!'

FROM base AS third
RUN echo 'Re-using base a third time!'

COPY --from=some-other-image /header.txt /app/include/header.txt
```

### Multi-stage Build Advantages

* Produces significantly smaller final images
* Can be much faster than builder pattern while
    being less fragile
* Can produce extremely secure images by discarding
    unnecessary dependencies

```dockerfile
# tiny.Dockerfile
# This image is awesome!
FROM ubuntu AS base
ENV curl_bin="curl"
RUN apt -y update && apt -y install "$curl_bin"
RUN curl -i -sS google.com | \
  grep -E '^Date:' | \
  sed 's/^Date: //' | tr -d $'\r' > '/tmp/date.txt'
RUN curl -Lo /tmp/bash \
  'https://github.com/robxu9/bash-static/releases/download/5.1.016-1.2.3/bash-linux-aarch64' && \
  chmod +x /tmp/bash

FROM busybox:uclibc
COPY . /app
COPY --from=base /tmp/date.txt /app/include/date.txt
COPY --from=base /tmp/bash /usr/local/bin/bash
ENTRYPOINT [ "/usr/local/bin/bash", "/app/app.sh" ]
CMD [ "--argument" ]
```

```bash
# base.Dockerfile
./size_diff.sh
# App size, in bytes: 876
# App image size, in bytes: 118489088
# Size difference: 135261x
```

```bash
# tiny.Dockerfile
./size_diff.sh
# App size, in bytes: 876
# App image size, in bytes: 13212058
# Size difference: 15081x
```
