---
layout: category
title: Einrichtung Dockercontainer & Jenkins
---
## Erstellen des Dockercontainers für Jenkins

### Erzeugung eines neuen Dockerimage aus einem Dockerfile

Beispielhaftes Dockerfile für einen Container, in dem Jenkins laufen soll:

```Dockerfile
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

Dabei installiert der erste RUN-Befehl Docker innerhalb des Containers. Zu beachten ist dabei, dass die Dockerversion (hier 18.06.1~ce~3-0~debian) mit der Version vom Server übereinstimmt.

Der zweite RUN-Befehl erzeugt einen Benutzereintrag für den Benutzer mit der ID 1500 und fügt diesen der "docker" Benutzergruppe hinzu. Dies ist notwendig, da ansonsten Jenkins nicht die Rechte hat, einen neuen Dockercontainer zu starten. Die Benutzer-ID wird beim starten des Containers festgelegt und sollte mit der hier angegeben ID übereinstimmen. Der Benutzername ist beliebig und kann, da als ARG definiert, beim Erstellen des Images festgelegt werden, ohne das Dockerfile anzupassen. "jenkinsbuild" ist der Default-Wert, wenn dies nicht gemacht wird. Falls ein Home-Verzeichnis für diesen Benutzer erzeugt werden soll, muss die Option "-m" statt "-M" gewählt werden.

Die erstellte Datei sollte nur den Namen "Dockerfile" tragen und sich in einem leeren Ordner befinden (dort befindliche Dateien werden standardmäßig in das Image rein kopiert). Zum erstellen des Images wechselt man in diesen Ordner und gibt folgenden Befehl ein (der Punkt am Ende ist der Pfad zum Dockerfile):

```
docker build -t new_image .
```


#### Alternative ohne Dockerfile

Ein Image lässt sich auch manuell ohne Dockerfile erstellen. In dem Fall erzeugt und startet man einen Container aus einem Basis-Image (z.B. jenkins/jenkins:lts) und wechselt in diesen mit folgendem Befehl:

```
docker exec -it -u root jenkins bash
```

Statt "root" ist es auch möglich sich als beliebiger Nutzer einzuloggen. Zur Installation von neuer Software ist es meistens aber notwendig, root-Rechte zu besitzen. Nachdem alle Modifizierungen vorgenommen wurden und benötigte Programme installiert wurden, wechselt man mit "exit" wieder auf die Server-Ebene. Dort lässt sich der modifizierte und noch laufende Dockercontainer über folgenden Befehl als neues Image speichern:

```
docker commit eb6323d17d47  new_containername:latest
```

Dabei ist "eb6323d17d47" die ID von dem laufenden Container. Speichert man die Änderungen nicht in einem neuen Image, gehen diese bei Beendigung des Containers verloren.

### Start und Konfiguration von Jenkins

Wenn der Dockercontainer, der Jenkins enthält, gestartet werden soll, muss die Konfiguration angepasst werden, damit die durch Jenkins erstellten Dockercontainer auf der selben Ebene wie Jenkins-Dockercontainer liegen. Läuft die Konfiguration über eine Datei (z.B. /etc/docker/compose/jenkins/docker-compose.yml), muss dort folgender Eintrag eingefügt werden:

```
volumes:
    - /var/run/docker.sock:/var/run/docker.sock
```

## Zugriff auf Ordner außerhalb des Containers

Für das Speichern und Laden z.B. des m2-Caches von Maven, ist es notwendig, auf Ordner außerhalb des eigenen Containers zuzugreifen. Gemappt werden diese Ordner beim Start des Containers mit der folgenden Option:

```
-v /home/jenkinsbuild/.m2:/var/maven/
```

Wenn dies beim Start des Maven-Dockercontainers innerhalb des Jenkins-Dockercontainers angegeben wird, dann existiert innerhalb des Maven-Dockercontainers der Ordner "/var/maven/", der mit dem außerhalb liegenden Ordner /home/jenkinsbuild/.m2 gemappt ist. Achtung: da, dieser Container auf der selben Ebene liegt, wie der Jenkins-Dockercontainer selber, ist dieser gemappte Order auf Server-Ebene und nicht auf Jenkins-Ebene. Bei diesem Mapping werden die Zugriffsrechte von dem außerhalb liegenden Ordner übernommen. Damit Maven also darauf zugreifen kann, muss vorher in dem Fall auf Server-Ebene dieser Ordner erstellt werden und die Zugriffsrechte festgelegt werden:

```bash
chown -R 1500 /home/jenkinsbuild/.m2
```

### Nutzung von Docker volumes

Alternativ ist es möglich ein Docker volume statt einem Ordner in den Container zu mappen. Dazu muss als erstes ein solches volume erstellt werden:

```
docker volume create new_volume
```

Mit den folgenden zwei Befehlen lassen sich alle volumes anzeigen und einzelne genauer betrachten, um z.B. den Dateipfad des volumes herauszufinden:

```
docker volume ls
docker volume inspect new_volume
```

Letzteres wird benötigt, da es auch bei dieser Variante von Nöten ist, die Zugriffsrechte festzulegen, damit innerhalb des Containers auf das volume schreibend zugegriffen werden kann.
Das Einbinden des volumes verläuft analog wie bei einem Ordner:

```
-v new_volume:/home/jenkinsbuild/volume_folder
```
