---
layout: category
title: Setup Dockercontainer & Jenkins
sidebar_sort_order: 3
---

## 1. Creation of the dockercontainer for Jenkins

### 1.1 Generation of a new dockerimage from a dockerfile

Example dockerfile to create a container, in which Jenkins can run:

```dockerfile
FROM jenkins/jenkins:lts
ARG username=jenkinsbuild
USER root
RUN apt-get update && \
    apt-get -y install apt-transport-https \
        ca-certificates \
        curl \
        gnupg2 \
        software-properties-common && \
    curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey && \
    add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
        $(lsb_release -cs) \
        stable" && \
    apt-get update && \
    apt-get -y install docker-ce=18.06.1~ce~3-0~debian
RUN useradd -M -s /usr/sbin/nologin -u 1500 -G docker $username
```

The first RUN-command installs Docker inside of the container. One must consider that the version of docker (here 18.06.1\~ce\~3-0\~debian) has to match with the version of docker which is installed on the server.

The second RUN-command creates a new user with the ID 1500 and adds this user to the user group "docker". This is needed so Jenkins has the rights to start a new docker container. The user ID is set at the start of a container and should match with the here defined ID. The username can be chosen freely. Since the name is defined via ARG, it is possible to set this name when the image is created without the need to adapt the dockerfile. "jenkinsbuild" is the default value for the username. If there is the need to have a home directory for this user, use the option "-m" instead of "-M".

The created file should only have the name "Dockerfile" and exist in an empty folder (any other file which are located in the same folder are copied as a default setting inside of the crated image). To create the image switch to the containing folder und run following command (the dot an the end is the path to the dockerfile):

```shell
docker build -t new_image .
```


#### 1.1.1 Alternative without a dockerfile

An docker image can be created manually without a dockerfile. In this case create and start a container from an base image (e.g. jenkins/jenkins:lts) and enter this container with the following command:

```shell
docker exec -it -u root jenkins bash
```

Instead of "root" is is possible to log in as an arbitrary user. But to install new software inside the container it is normally needed to have root permissions. After all modifications are done and needed programs are installed, switch back to server layer with "exit". There the modified and still running docker container can be saved as an new image with the following command:

```shell
docker commit eb6323d17d47  new_containername:latest
```

"eb6323d17d47" is the ID of the running container. All changes inside of the container are discarded when stopping the container, if they are not saved in a new image.

### 1.2 Start and configuration of Jenkins

On starting the docker container, which runs Jenkins, a specific configuration is needed, so that docker container, which are created through Jenkins, stay on the same layer, on which the Jenkins container is. If the configuration is inside a file (e.g. /etc/docker/compose/jenkins/docker-compose.yml), is is needed to add the following entry:

```shell
volumes:
    - /var/run/docker.sock:/var/run/docker.sock
```

## 2. Access of folders outside of the container

For saving and loading of e.g. the m2-cache of Maven, it is needed to access folders outside of one's own container. The folders are mapped at the start of the container with the following option:

```shell
-v /home/jenkinsbuild/.m2:/var/maven/
```

If this option is provided at the start of the Maven docker container inside of the Jenkins docker container, then a folder "/car/maven/" exists inside of the Maven docker container which is mapped to the folder /home/jenkinsbuild/.m which exists outside of this container. Attention: since this container exists on the same layer as the Jenkins docker container, this mapped folder is on the server layer and not on the Jenkins layer. With this mapping the permissions of the outside existing folder are inherited. To enable Maven to access this folder, is is needed to create the mapped folder on the server layer and give the needed permissions:

```shell
chown -R 1500 /home/jenkinsbuild/.m2
```

### 2.1 Use of Docker volumes

As an alternative it is possible to map Docker volumes instead of folders inside of a container. To do this, it is first needed to create such a volume:

```shell
docker volume create new_volume
```

With the following two commands it is possible to list all volumes and to inspect volumes to e.g. get the file path of the location of a volume:

```shell
docker volume ls
docker volume inspect new_volume
```

The latter is needed, because with this variant it is also needed to set the permission to be able to write to the volume inside of a container.
The integration of the volume into the container is analogous to the variant with the folder:

```shell
-v new_volume:/home/jenkinsbuild/volume_folder
```

## 3. Generation of the Pipeline

Exemplary Pipeline, which uses a docker volume like described above:

```shell
pipeline {
    agent {
        docker {
            image 'custom_maven:latest'
            args '-v m2-volume:/home/jenkinsbuild/.m2 -m 4G'
        }
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
    }
}
```

Within the arguments for the start of the docker container after mapping the docker volume there is a limitation of the memory. In this case the container can only use 4 gigabyte of memory. A limitation of the CPU is also possible with option '-c xx'. With this option is a number provided, by which the relative usage is distributed. If a container has e.g. the option -c 20 and another has the option -c 80, than (if both container working to capacity) the second container can only use 80% of the CPU resources.

In this example file an option is set, to have a timeout for the whole pipeline of 30 minutes. It is also possible to create separate timeouts for the individual stages.


### 3.1 Limit the used memory (on the hard drive)

One possible attack is to fill the hard drive with data until no space is left. To prohibit this action, it is possible to limit the available space of a docker container in the start configuration. The requirement to do this is, that the filesystem (for the container) has to be a "xfs"-filesystem with the 'pquota' mount option. If the requirement is met, a limitation of e.g. 20 gigabyte looks like this:

```Jenkinsfile
agent {
    docker {
        image 'custom_maven:latest'
        args "... --storage-opt size=20G ..."
    }
}
```

### 3.2 Access buildfiles in the Jenkins container

Some jobs are done in the maven container (e.g. do the actual build) and some jobs are done in the jenkins container (e.g. deployment). It is needed to establish some kind of communication between this container, because for e.g. the deployment jenkins needs the files which are created during the build in the maven container. One solution is to mount a folder in both containers like described in chapter [2](setupDockercontainerJenkins#2.-access-of-folders-outside-of-the-container). For security reasons this folder should be a own partition and/or has a disc quota set to limit the available space.

Jenkins should create a new subfolder with a name which is unique for each build so different builds cannot interfere with each other. This means that the name should contain the project name, branch name and the build id. At the start of the maven container only this subfolder should be mounted in! The only possible interference between two concurrent builds is, when one evil build tries to use all available space and the other (normal) build tries to write something at the same time. Both builds could fail in this case and the normal build has to be restarted again.

## 4. Use of docker network to redirect traffic via a proxy

Since some updatesites (which maven needs, to resolve dependencies) are unreliable, it is useful to use a proxy which redirects specific requests. One solution is "squid", which runs in a separate docker container. This container now must be connected to the maven container which can be done with a docker network. To do this, it is first needed to create a new docker network e.g. with the name "proxy":

```shell
docker network create --driver bridge proxy
```

After that we can connect a container with this network. This must be specified at the startup. If squid is started per manual command, it would look like this:

```shell
docker run -d --name squid --network proxy squid
```

If two running container are connected via a docker network, they can access each other with the specified name of the container. This name is automatically replaced with the corresponding ip address by docker. Now Maven must be configured to use this proxy. One way is to replace the settings.xml when the container is build from the dockerfile. The corresponding command would look like this:

```dockerfile
COPY settings.xml /usr/share/maven/conf/settings.xml
```

And the new settings.xml could look like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <proxies>
    <proxy>
      <id>eclipseProxy</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>squid</host>
      <port>3128</port>
    </proxy>
  </proxies>
  <profiles>
    <profile>
      <id>eclipseProxy-Profile</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <http.proxyHost>squid</http.proxyHost>
        <http.proxyPort>3128</http.proxyPort>
      </properties>
    </profile>
  </profiles>
</settings>
```

Usefull commands to read the logs of the running squid docker container:

```shell
docker exec -it squid tail -f /var/log/squid/access.log
docker exec -it squid cat /var/log/squid/cache.log
```

## 5. Updating a docker image/container

To update for example the Jenkins container from the custom image follow these steps:

1. Stop the running container `systemctl stop docker-compose@jenkins`
2. Get a new base image `docker pull jenkins/jenkins:lts`
3. Build the new custom image `docker build -t new_jenkins .`
4. Start the container `systemctl start docker-compose@jenkins`
