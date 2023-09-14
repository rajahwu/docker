# Docker Images Demysified

## What are Docker images?

Container images are prepackaged filesystems for containers.

Every container image starts as a compressed archive.

Within the image are one or more "layers," each
    containing snippents of the container's filesystem

Finally, the image contains some metadata about itself.

## Diving deeper into Docker images

```bash
tar -t -f ./hellow-world.tar.gz

247460e92ef67fdf643394f29fdfdfcec2fde609010ec63e3b7bee779e1a4846/
247460e92ef67fdf643394f29fdfdfcec2fde609010ec63e3b7bee779e1a4846/VERSION
247460e92ef67fdf643394f29fdfdfcec2fde609010ec63e3b7bee779e1a4846/json
247460e92ef67fdf643394f29fdfdfcec2fde609010ec63e3b7bee779e1a4846/layer.tar
7b473dec0fa9e1cd2ffeb04ca39b125972ca0927000ccd033404674671768b8a/
7b473dec0fa9e1cd2ffeb04ca39b125972ca0927000ccd033404674671768b8a/VERSION
7b473dec0fa9e1cd2ffeb04ca39b125972ca0927000ccd033404674671768b8a/json
7b473dec0fa9e1cd2ffeb04ca39b125972ca0927000ccd033404674671768b8a/layer.tar
d68fcc334f453a8c889c682226e6c6dc39694eaa4af54fdc8cc03bba03fdbb1c.json
manifest.json
repositories

tar -x -O -f ./hello-world.tar.gz manifest.json | jq

[
  {
    "Config": "d68fcc334f453a8c889c682226e6c6dc39694eaa4af54fdc8cc03bba03fdbb1c.json",
    "RepoTags": [
      "test:latest"
    ],
    "Layers": [
      "7b473dec0fa9e1cd2ffeb04ca39b125972ca0927000ccd033404674671768b8a/layer.tar",
      "247460e92ef67fdf643394f29fdfdfcec2fde609010ec63e3b7bee779e1a4846/layer.tar"
    ]
  }
]

tar -x -O -f ./hello-world.tar.gz d68fcc334f453a8c889c682226e6c6dc39694eaa4af54fdc8cc03bba03fdbb1c.json| jq
{
  "architecture": "arm64",
  "config": {
    "Hostname": "",
    "Domainname": "",
    "User": "",
    "AttachStdin": false,
    "AttachStdout": false,
    "AttachStderr": false,
    "Tty": false,
    "OpenStdin": false,
    "StdinOnce": false,
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ],
    "Cmd": [
      "/hello"
    ],
    "Image": "sha256:46331d942d6350436f64e614d75725f6de3bb5c63e266e236e04389820a234c4",
    "Volumes": null,
    "WorkingDir": "",
    "Entrypoint": null,
    "OnBuild": null,
    "Labels": null
  },
  "container_config": {
    "Hostname": "",
    "Domainname": "",
    "User": "",
    "AttachStdin": false,
    "AttachStdout": false,
    "AttachStderr": false,
    "Tty": false,
    "OpenStdin": false,
    "StdinOnce": false,
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ],
    "Cmd": [
      "/bin/sh",
      "-c",
      "#(nop) COPY file:5c850056aa49a5a99cda22a7b820dc934348738ac919683db00729574acbeb11 in / "
    ],
    "Image": "sha256:46331d942d6350436f64e614d75725f6de3bb5c63e266e236e04389820a234c4",
    "Volumes": null,
    "WorkingDir": "",
    "Entrypoint": null,
    "OnBuild": null,
    "Labels": null
  },
  "created": "2022-11-29T21:16:36.165868669Z",
  "docker_version": "20.10.3",
  "history": [
    {
      "created": "2022-03-19T16:12:58.834095198Z",
      "created_by": "/bin/sh -c #(nop) COPY file:a79dd5bda1e77203401956a93401d3aef45221fc750295a4291896f3386f4f54 in / "
    },
    {
      "created": "2022-03-19T16:12:58.923371954Z",
      "created_by": "/bin/sh -c #(nop)  CMD [\"/hello\"]",
      "empty_layer": true
    },
    {
      "created": "2022-11-29T21:16:36.165868669Z",
      "created_by": "/bin/sh -c #(nop) COPY file:5c850056aa49a5a99cda22a7b820dc934348738ac919683db00729574acbeb11 in / "
    }
  ],
  "os": "linux",
  "rootfs": {
    "type": "layers",
    "diff_ids": [
      "sha256:efb53921da3394806160641b72a2cbd34ca1a9a8345ac670a85a04ad3d0e3507",
      "sha256:fcf02bf1bcd706a080eb35ba08824d3995112cb3b94c16d3f288a8a5bb958dc3"
    ]
  },
  "variant": "v8"
}
```

## Storage drivers

Storage drivers (also called graph drivers or snapshotters) define how layers are stored
    on disk and represented to containers.

### **overlay2** is the most popular storage driver

* Each layer gets its own top-level folder and is linked with the **l** directory.

* The bottom-most layer (the **lower directory**) has its
    contents in **diff** and its link ID in **link**

* Every child layer also has **lower** pointing back
    to its previous layer, its merged contents in
    **merged**, and a staging area called **work**

* Finally, the storage driver mounts **l** and the
    lower directory as a single directory with a union filesystem

Storage drivers handle how files are copied up from the
    image(**lowerdir**) into the upper directory(**uppperdir**)

### **stoarage driver disadvantages**

* **Copy-ups** can get really slow
* Changes to this filesystem go away after the
    container is removed - **this is why containers
    cannot save data by defautl**
* **Use volumes** if you care about I/O and/or
    **bind mounts** for keeping stuff around

## Decomposing Docker pull

```bash
sudo sh -c 'echo "{\"debug\": true}" > /etc/docker/daemon.json'

stat /etc/docker/daemon.json
  File: /etc/docker/daemon.json
  Size: 16              Blocks: 8          IO Block: 4096   regular file
Device: 820h/2080d      Inode: 912893      Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2023-09-14 12:58:42.362427209 -0400
Modify: 2023-09-14 12:58:42.372427204 -0400
Change: 2023-09-14 12:58:42.372427204 -0400
 Birth: -

sudo service docker restart
* Stopping Docker: docker               [ OK ] 
* Starting Docker: docker              [ OK ] 

docker run hello-world
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

## Configure Docker to enable debug logging

```bash
sudo journalctl -xu docker
sudo journalctl -f -u docker

```
