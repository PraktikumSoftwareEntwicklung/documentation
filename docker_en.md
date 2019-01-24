---
layout: category
title: Setup Dockercontainer & Jenkins
---
Eine deutsche Version ist [hier](docker_de "Einrichtung Dockercontainer & Jenkins").

## Creation of the dockercontainer for Jenkins

### Generation of a new dockerimage from a dockerfile

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

The second RUN-command creates a new user with the ID 1500 and adds this user to the user group "docker". This is needed so Jenkins has the rights to start a new docker container. The user ID is set at the start of a container and should match with the here defined ID. The username can be chosen freely. Since the name defined via ARG, it is possible to set this name when the image is created without the need to adapt the dockerfile. "jenkinsbuild" is the default value for the username. If there is the need to have a home directory for this user, use the option "-m" instead of "-M".

The created file should only have the name "Dockerfile" and exist in an empty folder (any other file which are located in the same folder are copied as a default setting inside of the crated image). To create the image switch to the containing folder und run following command (the dot an the end is the path to the dockerfile):

```shell
docker build -t new_image .
```


#### Alternative without a dockerfile

An docker image can be created manually without a dockerfile. In this case create and start a container from an base image (e.g. jenkins/jenkins:lts) and enter this container with the following command:

```shell
docker exec -it -u root jenkins bash
```

Instead of "root" is is possible to log in as an arbitrary user. But to install new software inside the container it is normally needed to have root permissions. After all modifications are done and needed programs are installed, switch back to server layer with "exit". There the modified and still running docker container can be saved as an new image with the following command:

```shell
docker commit eb6323d17d47  new_containername:latest
```

"eb6323d17d47" is the ID of the running container. All changes inside of the container are discarded when stopping the container, if they are not saved in a new image.

### Start and configuration of Jenkins

On starting the docker container, which runs Jenkins, a specific configuration is needed, so that docker container, which are created through Jenkins, stay on the same layer, on which the Jenkins container is. If the configuration is inside a file (e.g. /etc/docker/compose/jenkins/docker-compose.yml), is is needed to add the following entry:

```shell
volumes:
    - /var/run/docker.sock:/var/run/docker.sock
```

## Access of folders outside of the container

For saving and loading of e.g. the m2-cache of Maven, it is needed to access folders outside of one's own container. The folders are mapped at the start of the container with the following option:

```shell
-v /home/jenkinsbuild/.m2:/var/maven/
```

If this option is provided at the start of the Maven docker container inside of the Jenkins docker container, then a folder "/car/maven/" exists inside of the Maven docker container which is mapped to the folder /home/jenkinsbuild/.m which exists outside of this container. Attention: since this container exists on the same layer as the Jenkins docker container, this mapped folder is on the server layer and not on the Jenkins layer. With this mapping the permissions of the outside existing folder are inherited. To enable Maven to access this folder, is is needed to create the mapped folder on the server layer and give the needed permissions:

```shell
chown -R 1500 /home/jenkinsbuild/.m2
```

### Use of Docker volumes

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

## Generation of the Jenkinsfile

Exemplary Jenkinsfile, which uses a docker volume like described above:

```Jenkinsfile
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

Within the arguments for the start of the docker container after mapping the docker volume there is a limitation of the memory set. In this case the container can only use 4 gigabyte of memory. A limitation of the CPU is also possible with option '-c xx'. With this option is a number provided, by which the relative usage is distributed. If a container has e.g. the option -c 20 and another has the option -c 80, than (if both container working to capacity) the second container can only use 80% of the CPU resources.

In this example file an option is set, to have a timeout for the whole pipeline of 30 minutes. It is also possible to create separate timeouts for the individual stages.
