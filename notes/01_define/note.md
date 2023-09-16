# Docker Explained

## Docker recap

```bash
# Docker debug log
sudo tail -f /var/log/docker.log
```

```json
// /etc/docker/daemon.json
{
    "debug": true,
    "log-driver": "local"
}
```

### problem

* different system configuration
* missing files, hardware, or other properties
* hardware dependencies

### previous solutions

* Configuration management tools (Chef, Puppet, Ansible)
  * Configuration as code
  * Require knowledge about hardware and operating systems

* Virtual machines as code (Vagrant)
  * Configure pre-made virtual machines with HashiCorp Congiguration Language (HCL)
  * Heavy, slow-ish, requires inconvenient configuration

### docker

**Docker** uses **images** and **containers**
    to allow apps to run anywhere, consistently

Docker user **control groups**, **namespaces**, and **images**
    to make running apps anywhere easier.

#### containers

1. Configuration through Dockerfiles
2. Share images with other through image registries
3. Command line client and API

Containers are *not* smaller VMs.
|Containers | Virtual Machines |
|-----------|------------------|
| - Run in container runtimes | - Run on top of hypervisors |
| - Work alongside operating systems | - Need hardware emulation |
| - Do not requrie OS configuration | - Can run many apps at once |
| -Run one app at a time (usually) | - Can run many apps at once |  

## Container runtimes

Create, manage, and delete containers

```bash
docker run --rm bash -c 'echo "hello, world!"'
# hello, world!
```

|Can: | Cannot: |
|----|--------|
| - Create namespaces | - Build images |
| - Create and associate cgroups | - Pull images |
| - Map filesystems to containers | - Sever APIs for interacting with containers |
| - Set container capabilities | |
| - Start, stop, and remove individual containers | |

## OCI and CRI runtimes

### types of container runtimes

#### OCI

* Open Container Initiative (OCI)

  * aims to standardize container technology,
    like container images and runtimes

  * The OCI Runtime Specification outlines what
    a container is and how they should be managed

  * **runtime-spec** does **not** dictate **how**
    to do these things

  * Runtime Spec is open source and actively maintained

  * **runc** from Docker is the de facto industry standard container runtime

  * **crun** is the Red Hat defalult container runtime
    and is written in C for performance

  * **youki** is a newer container runtime,
    written in Rust

* Sandboxed OCI Runtimes

  * **gVisor** and **Nabla Containers** use unikernels
    to restrict what container can and cannot do

  * **Kata Containers** is a runtime for
    creating containers inside of virtual machines

#### CRI

* Container Runtime Interface(CRI)

  * CRI provides an API for running containers
    on container runtimes

  * This allows project like Kubernetes to not be
    tied to any specific runtime or runtime standard

  * **containerd** is a popular CRI runtime that
    uses **runc** to create OCI containers

  * **CRI-O** is a lightweight CRI runtime optimized
    for Kubernetes, maintained by Red Hat, Intel, and others

## The Docker Engine

Container engines work alongside container run times to provide components and tools for managing containers

* Docker Engine
  * Most popular container engine
  * Comes with **docker** command-line tool
  * Also comes with a REST-based API for managing containers
    and a DSL for creating container images called **Dockerfile**
  * Uses **containerd** as its runtime by default
* Podman
  * The Red Hat container engine
  * Functionally equivalent to Docker Engine
  * Also comes with a REST-based API for managing
    containers and a DSL for creating container images called
    **Buildah**
  * Uses **crun** as its runtime by default

## Where are Docker's configuration files?

* **/var/lib/docker:** containers, volumes, and metadata
* **/var/lib/docker/overlay:** container volumes
* **/bar/run/docker.sock:** the pipe between the Docker client
    and Docker Engine
* **/etc/docker/daemon.json:** Docker Engine configuration
    (might not exist at first)

## Chapter Quiz
