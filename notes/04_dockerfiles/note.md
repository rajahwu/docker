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

* **Dockerfiles:** The Dockerfile is a language for
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

## Soring other Docker images with **FROM**

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
|*|Matches all characters after **word**(until next non-asterisk(character))|songs/song-1.mp3 songs/song-2.mp3 songs/song-3.mp3 songs/song1.mp3 songs/song2.mp3 songs/song3.mp3|COPY songs/song* /songs|songs/song-1.mp3 songs/song-2.mp3 songs/song-3.mp3 songs/song1.mp3 songs/song2.mp3 songs/song3.mp3|