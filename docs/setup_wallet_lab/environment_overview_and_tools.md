# Environment Overview and Tools

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Docker](#docker)
  - [Docker Compose](#docker-compose)
- [Git and GitHub](#git-and-github)
  - [Cloning the Project](#cloning-the-project)

## Introduction

This page provides an overview of the tools and software used in setting up and
running the DC4EU interoperability lab. The instructions are geared toward
**Ubuntu/Debian** systems and focus on the technologies required to manage
containers, coordinate multi-container applications, and handle version
control.

With these tools in place, you will be ready to proceed with cloning the
repository and setting up the environment as described in subsequent pages.

## Prerequisites

Before starting, ensure that the following software and tools are installed on
your system. All instructions are for Ubuntu/Debian:

- **Docker**: Version 20.10 or later
- **Docker Compose**: (Integrated as `docker compose`) Requires Docker CLI
  version 20.10 or later
- **Git**: Version 2.25 or later

## Docker

Docker is a platform that enables developers to build, run, and manage
applications within containers. A container is a lightweight, standalone package
that includes everything an application needs to run: code, runtime, system
tools, libraries, and settings. Containers are especially useful in creating
consistent test environments because they can be easily shared, replicated, and
deployed across various systems, making it straightforward to ensure that your
application runs the same everywhere.

While Docker containers share the host operating system's kernel, they run a
complete, lightweight operating system inside the container. This OS is
specifically designed to run the application and its dependencies efficiently.
This means that each container has its own file system, processes, and network
interfaces, providing a high level of isolation and security.

Containers are created from Docker images, which act as templates or blueprints.
A Docker image includes all components necessary to run an application the
application code, libraries, dependencies, and default configurations. Each time
you run a Docker image, it creates a new container based on that template,
ensuring consistency across different environments.

For the interoperability lab, all Docker images are created and hosted by SUNET.

Key Docker commands:

- `docker pull <image>`: Downloads a Docker image from a remote repository.
- `docker run <image>`: Runs a container from a specified image.
- `docker ps`: Lists all running containers.
- `docker stop <container>`: Stops a running container.

### Docker Compose

Docker Compose is a tool used to define and manage multi-container Docker
applications. Using a `docker-compose.yaml` file, you can specify the necessary
containers for your application by either building images locally or using
pre-built images. For the interoperability lab, all images are hosted by SUNET,
and the URLs for these images are listed in the `docker-compose.yaml` file.

This setup simplifies launching a complete environment, as Docker Compose
manages configuration details like environment variables, ports, and volume
mappings across multiple containers. With a single command, you can bring up the
entire environment specified in the Compose file, making it easy to set up and
replicate consistent environments.

Key Docker Compose commands:

- `docker-compose up`: Starts the services defined in your
  `docker-compose.yaml`.
- `docker-compose down`: Stops and removes the containers defined in the Compose
  file.
- `docker-compose ps`: Lists all running services defined in the Compose file.

## Git and GitHub

Git is a version control system that allows multiple people to work on a project
without overwriting each other's work. It tracks changes and enables
collaboration by managing branches and repositories. GitHub is a web-based
platform that hosts Git repositories, making it easier to share, review, and
collaborate on code.

Key Git commands:

- `git clone <repository_url>`: Copies a remote repository (like GitHub) to your
  local machine.
- `git status`: Shows the status of your current repository (modified files,
  staged changes, etc.).
- `git add <file>`: Adds a file to the staging area, preparing it to be
  committed.
- `git commit -m "message"`: Saves your staged changes to the repository with a
  message describing the update.

### Cloning the Project

To get started with the interoperability lab, clone the `vc_up_and_running`
project from GitHub

1. Open your terminal.
1. Use `git clone https://github.com/dc4eu/vc_up_and_running.git` to download
   the repository to your local environment.
1. Change into the project directory with `cd vc_up_and_running`.
