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

