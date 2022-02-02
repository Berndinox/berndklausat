---
draft: false
title: Docker Error Logs via Push
date: 2016-09-05T22:40:32.169Z
tags:
  - Docker
  - Error
  - Pushover
  - Container
---
Ihr könnt also jeden einzelnen Container mit “docker logs {ID}” abfragen, oder Ihr verwendet den alt bewährten ELK Stack. Für mich sind beide Lösungen nicht zufrieden stellend. Erstere ist einfach unpraktisch und der ELK Stack benötigt zu viele Resourcen und Wartungsaufwand. Die Lösung ist “Docker Logs via Pushover“!

{{< figure src="/images/docker.png" alt="docker" width="200px" >}}

Erst kürzlich habe ich auf dem DockerHub, für diesen Zweck, folgendes Image veröffentlicht: **[Berndinox/fluentd-pushover](https://hub.docker.com/r/berndinox/fluentd-pushover/)** @ DockerHub

## Das Prinzip ist einfach

Via nativen Docker Logging Treiber werden alle Logs die innerhalb eines Containers generiert werden an die Fluentd-Pushover Container weitergeleitet. Dieser analysiert die Log-Einträge und falls diese ein “Fail” oder “Error” beinhaltet wird dieser Log-Eintrag via Pushover weitergeleitet.

## Klarer Vorteil

Es ist keine Datenbank notwendig, keine komplexe ELK Umgebung, welche Resourcen frisst, außerdem wird nur minimaler Speicherplatz benötigt.

## Howt

Der empfohlene Weg ist der Einsatz via Docker-Compose. Ein beispiel findet Ihr auf meinem Github Repo, oder anbei:

```
version: '2'

services:
  fluentd:
    #build: .
    image: berndinox/fluentd-pushover
    hostname: fluentd
    expose:
      - "24224"
    volumes:
      - ./fluent.conf:/fluentd/etc/fluent.conf
    restart: always
    networks:
      logs:
        ipv4_address: 10.0.0.10

networks:
  logs:
    driver: bridge
    ipam:
      driver: default
      config:
      - subnet: 10.0.0.0/24
        gateway: 10.0.0.1
```

Es wird ein Netzwerk mit Subnet 10.0.0.0/24 erstellt, der Fluentd Container macht sein Service via Port 24224 unter IP 10.0.0.10 für alle anderen Container verfügbar.

Diese müssen nur mehr die Logs an diese Adresse weiterleiten, dies geschieht wie folgt:

```
version: '2'
services:
  ping:
      image: dockercloud/hello-world
      expose:
       - "80/tcp"
      logging:
       driver: "fluentd"
       options:
         fluentd-address: "10.0.0.10:24224"
```

Sprich, bei allen anderen Container-Services muss lediglich der Logging Driver angepasst werden, sowie die IP.

## Achtung

- Ihr müsst auf Pushover.net einen Account erstellen und euren User sowie Account Token in der fluent.conf Dateil hinterlegen.
- Verwendet Ihr meine Compose Vorlage wird die fluentd.conf via Volume in den Container gemappt. Alternativ kann man sein eigenes Image bauen lassen (Dockerfile)  und via “ADD fluentd.conf /etc/fluentd/” die Configuration hinzufügen.