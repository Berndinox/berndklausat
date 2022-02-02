---
slug: docker-secure-cli-bastionhost
draft: false
title: Docker-CLI Bastion
date: 2017-04-01T22:40:32.169Z
tags:
  - Docker
  - CLI
  - Security
  - Container
---
## Achtung
Gemäß Sicherheitsempfehlungen sollte der Docker-Socket oä. nicht mehr in einen Pod zur Verwaltung gemountet werden.
Der Einsatz der unten angeführten Lösung ist also NICHT mehr empfolhen.
Dennoch verbleibt der Eintrag als etwaige Referenz oder Beispiel.

## Bastion Container
Nachfolgender Docker-Container hilft euch euer System zu managen. Er ermöglichst es die Docker Container auf dem darunter liegenden Host zu managen.

{{< figure src="/images/docker.png" alt="docker" width="200px" >}}

Das hat einige Sicherheitsvorteile, denn so ist kein direkter Zugriff auf die Shell des Servers notwendig, sonder man befindet sich in einem abgeschotteten Container, hat jedoch dennoch vollständige root rechte für “docker” Befehle.

Zusätzlich kann man, wenn man am Root-Server mehrere IP Adressen verfügbar hat, die Haupt IP Adresse “verstecken”, des weiteren habe ich Google OTP PAM Modul implementiert, welches per default auch aktiviert ist.

Der Container kann wie folgt gestartet werden:
```
docker run -d -p IP-TO-PUBLISH:22:22 -e PASSWORD=SuperGeheim -v /etc/localtime:/etc/localtime:ro -v /var/run/docker.sock:/var/run/docker.sock berndinox/docker-bastionhost
```
Oder via **Compose**:
```
version: '3.2'

services:
  ssh:
    image: berndinox/docker-bastionhost
    deploy:
      replicas: 1
    ports:
      - "IP:22:22"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock
      - data:/data
    environment:
      - PASSWORD=SECRET

volumes:
  data:
    driver: DRIVER
    driver_opts:
      mountpoint: /data
```

Abschließen, zum generieren der OTP Konfiguration, bzw des QR Codes:
```
docker exec -it DOCKERID google-authenticator
```

## Link
[Berndinox/dockerbastionhost](https://github.com/Berndinox/docker-bastionhost) @ Github

