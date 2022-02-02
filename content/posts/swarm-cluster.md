---
slug: swarm-cluster-scaleway
draft: false
socialImage: /media/docker-swarm.png
title: Swarm Cluster - High-Availability
date: 2017-03-15T12:40:32.169Z
tags:
  - Swarm
  - Docker
  - Scaleway
  - Container
---
## Update 2020
Es hat sich bereits einiges getan, empfehlenswert ist aktuell der wechsel zu: **Kubernetes**

## Setup 
Nichts desto trotz ist nachfolgendes Setup immer noch umsetzbar.

{{< figure src="/images/docker-swarm.png" alt="swarm" width="250px" >}}

**Zuerst** benötigen wir 3 virtuelle Maschinen, nicht möglich ist das Setup mit den VMs “C1″ und VC1S”, da wir eine 2. Disk benötigen. In meinem Setup kommt die VM “VC1M” zum Einsatz. Als Image ist im “ImageHub” die Docker Vorlage zu wählen.

**2)**  Sobald die VMs provisioniert sind beginnen wir mit dem Basis-Setup:

```
apt-get update && apt-get upgrade -y
mkfs -t ext4 /dev/vdb
mkdir -p /mnt/data
echo "/dev/vdb /mnt/data auto  defaults,errors=remount-ro 0 2" >> /etc/fstab
init 6
```
Docker wird auf Version 1.__x__ aktualisiert, die 2. Datendisk wird formatiert und zu den Mountpunkt /mnt/data hinzugefügt.

**3)** Damit die Container auf allen Hosts die gleichen Daten zur Verfügung haben gibt es einige Wege.  Eine Möglichkeit wäre der Einsatz eines Volume-Plugins welches eine Zieldestination automatisch auf dem Host mapped. Hier eine Liste der Verfügbaren Plugins.

In unserem Szenario wollen wir jedoch eine hoch verfügbare Umgebung aufbauen, dh. auch unser Speichersystem muss repliziert werden. Dazu verwenden wir glusterfs.

Glusterfs wird mit 3 Replicas angelegt, sprich pro Host eine Kopie der Daten. Somit entfällt der Schritt, dass Volumes gemappt werden müssen, da die Daten ohnehin auf jedem Host verfügbar sind. Damit jedoch **“Named Volumes”** verwendet werden können muss das Storage Plugin “local-persist” verwendet werden (Siehe Punkt 4).

Voraussetzung für GlusterFS ist das die privaten IPs auf den Hostname verweisen. Meine Hostnames lauten: Node01, Node02, Node03

```
apt-get install -y software-properties-common
add-apt-repository ppa:gluster/glusterfs-3.8
apt-get update && apt-get install -y glusterfs-server
service glusterfs-server start
gluster peer probe node02
gluster peer probe node03
gluster peer status 
mkdir -p /mnt/gluster
gluster volume create gvol replica 3 node01:/mnt/gluster node02:/mnt/gluster node03:/mnt/gluster
gluster volume set gvol performance.cache-size 256MB
```

anschließend jeweils auf allen Nodes das Volume verbinden:anschließend jeweils auf allen Nodes das Volume verbinden:

```
echo "Node1:/gvol0 /mnt/glusterfs glusterfs  defaults,_netdev 0 0" >> /etc/fstab
init 6
```

Auf Node02, den String auch auf Node02 anpassen, sodass das Gluster Volume immer lokal verbunden wird.

**4)** Nun installieren wir das Docker Plugin “local-persist” damit Named-Volumes auf unser Glusterfs gelegt werden können.

```
curl -fsSL https://raw.githubusercontent.com/CWSpear/local-persist/master/scripts/install.sh | sudo bash
```

**5)** Als letzten Schritt initialisieren wir den Docker Swarm Cluster.

```
docker swarm init
docker swarm join-toker manager
```

die Ausgabe des letzen Befehles kopieren und auf den beiden anderen Nodes ausführen. Anschließend kann man wie folgt den Cluster Status abfragen:

### Verify
```
docker node ls
docker info
```

Unser hoch verfügbarer Swarm Cluster im neuen Swarm-Mode ist nun fertig.