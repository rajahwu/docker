# Docker Explained

## Docker recap

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
| - Run in container runtimes | - Run on top of ypervisors |
| - Work alongside operating systems | - Need hardware emulation |
| - Do not requrie OS configuration | - Can run many apps at once |
| -Run one app at a time (usually) | - Can run many apps at once |  

## Container runtimes

## OCI and CRI runtimes

## The Docker Engine

## Where are Docker's configuration files?

## Chapter Quiz
