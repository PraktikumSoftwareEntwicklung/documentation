---
layout: category
title: Basic System Setup
sidebar_sort_order: 20
---

## Preparation of Docker Partition

Docker requires an XFS or BTRFS file system partition in order to enforce disk quotas (see [Docker Manual about storage driver options](https://docs.docker.com/engine/reference/commandline/run/).
* If not already done, create a partition using `fdisk` (create a new partition table `g`, create a new partition `n`, write changes `w`).
* To format the partition using XFS, you have to install `xfsutils` in order to get the `mkfs.xfs` executable.
* Execute `mkfs.xfs /dev/xxx1` for the device of the created partition.
* Use `blkid /dev/xxx1` to get the UUID of the partition.
* Edit `/etc/fstab` and insert the following line (use the previously determined UUID)
```shell
UUID=f50cc38b-038a-4a00-a73a-c4ab2a73a059       /media/docker   xfs     defaults,pquota 0       1
```
* Mount the partition using `mount -a`

## Docker Installation

* Perform installation according to [setup instructions](https://docs.docker.com/install/)
* Stop docker daemon
* Create the file `/etc/docker/daemon.json` with the following content
```shell
{
	"data-root": "/media/docker"
}
```
* Start docker daemon
* Install Docker Compose according to the [setup instructions](https://docs.docker.com/compose/install/)

## Preparations to Run Docker Compose Files as Services

We assume that systemd is running and operational. To support running docker compose files as services, perform the following steps:

* Create a file `/etc/systemd/system/docker-compose@.service` with the following content

```shell
[Unit]
Description=%i service with docker compose
Requires=docker.service
After=docker.service

[Service]
Restart=always

WorkingDirectory=/etc/docker/compose/%i

# Remove old containers, images and volumes
ExecStartPre=/usr/bin/docker-compose down -v
ExecStartPre=/usr/bin/docker-compose rm -fv
ExecStartPre=-/bin/bash -c 'docker volume ls -qf "name=%i_" | xargs docker volume rm'
ExecStartPre=-/bin/bash -c 'docker network ls -qf "name=%i_" | xargs docker network rm'
ExecStartPre=-/bin/bash -c 'docker ps -aqf "name=%i_*" | xargs docker rm'

# Compose up
ExecStart=/usr/bin/docker-compose up

# Compose down, remove containers and volumes
ExecStop=/usr/bin/docker-compose down -v

[Install]
WantedBy=multi-user.target
```

You can start docker compose files named `docker-compose.yml` in a folder `/etc/docker/compose/x` by using the command `systemctl start docker-compose@x`

## Setup of Reverse Proxy

Basically, you can use any reverse proxy but we decided for [Traefik](https://traefik.io/) because of its native docker support.

* Decide for a folder holding the configuration files. We will use `/media/data/traefik` in this example.
* Create an empty file `acme.json` in the configuration folder and set the chmod to 600
* Create a configuration file `traefik.toml` in the configuration folder containing the following contents (replace domain and email)

```shell
logLevel = "INFO"
defaultEntryPoints = ["https","http"]

[api]
  entryPoint = "traefik"
  dashboard = true
  address = "127.0.0.1:8080"

[entryPoints]
  [entryPoints.http]
  address = ":80"
    [entryPoints.http.redirect]
    entryPoint = "https"
  [entryPoints.https]
  address = ":443"
  [entryPoints.https.tls]

[docker]
endpoint = "unix:///var/run/docker.sock"
domain = "build.mdsd.tools"
watch = true
exposedByDefault = false

[acme]
email = "someone@example.org"
storage = "/acme.json"
entryPoint = "https"
onHostRule = true
onDemand = false

  [acme.httpChallenge]
  entryPoint = "http"
```
* Create a new docker network for web applications by issuing the command `docker network create web`
* Create the file `docker-compose.yml` in the folder `/etc/docker/compose/traefik` with the following contents

```shell
version: '2'

services:
  traefik:
    image: traefik
    restart: always
    ports:
      - 80:80
      - 443:443
      - 127.0.0.1:8080:8080
    networks:
      - web
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /media/data/traefik/traefik.toml:/traefik.toml
      - /media/data/traefik/acme.json:/acme.json
    container_name: traefik

networks:
  web:
    external: true
```
* Issue the command `systemctl enable docker-compose@traefik` to start the container automatically after system reboot

## Setup Jenkings

We have to have a custom version of the jenkins image to use docker from within the jenkins container. A manual to create this image is available at the [Jenkins configuration manual](setupDockercontainerJenkins). We present the basic configuration here.

* Create the folder `/media/data/jenkins` and execute the command `chown 1500:1500 /media/data/jenkins/` to make it writable
* Create the file `docker-compose.yml` in the folder `/etc/docker/compose/jenkins` with the following contents

```shell
version: '2'

services:
  jenkins:
    image: new_jenkins:latest
    restart: always
    user: "1500"
    expose:
      - 8080
    networks:
      - web
    volumes:
      - /media/data/jenkins:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    container_name: jenkins
    labels:
      - "traefik.enable=true"
      - "traefik.backend=jenkins"
      - "traefik.frontend.rule=Host:jenkins.build.mdsd.tools"
      - "traefik.port=8080"
      - "traefik.docker.network=web"

networks:
  web:
    external: true
```
* Issue the command `systemctl enable docker-compose@jenkins` to start the container automatically after system reboot
