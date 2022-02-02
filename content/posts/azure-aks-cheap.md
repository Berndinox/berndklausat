---
slug: azure-aks-cheap
draft: false
title: Azure AKS dirt-cheap
date: 2020-04-10T12:40:32.169Z
tags:
  - Azure
  - Kubernetes
  - AKS
  - Container
---

## Kubernetes

Der trend der letzten Jahre. Immer mehr Firmen springen auf und wollen Ihre Workloads dynamisch skalieren. Es haben sich bereits eine Vielzahl an Anbieter etabliert welche das Service in voll verwalteter Form zum konsum bereitstellen. Die Top-Drei Anbieter im Enterprise Segment:

- Google Cloud
- Amazon Web-Services
- Microsoft Azure

Die kosten für eine derartige Umgebung sind dabei *relativ* ident. Für eine genaue Hochrechnung empfiehlt sich der Kostenrechner des jeweiligen Anbieters. Der folgende Beitrag fokusiert sich dabei auf eine kosteneffiziente Bereitstellung auf der Platform **Microsoft Azure**.  
**Fokus:** Test- und Entwicklungsumgebung  


{{< figure src="/images/azure-aks.jpeg" alt="azure aks" width="300px" >}}


### Erstellen
Entscheident ist bereits die Erzeugung des Clusters, welche unbedingt **mittels Azur CLI** zu erfolgen hat. Dazu verwendet Ihr am besten die im Azure Portal integrierte Powershell *(CloudShell)*. Alternativ können die ntowendigen CMDLETS auch lokal installiert werden.

### Grundsatzentscheidung
Grundsätzlich können AKS CLuster in zwei unterschiedlichen Varianten provisioniert werden. 

- **Virtual Machine Scale Sets** *(Default)*
- **Availability Sets**

Der Support von Virtual Machine Scale Sets *(VMSS)* kann nur einmalig bei Erstellung des Cluster aktiviert, oder auch deaktiviert werden. Eine nachträgliche Änderung ist zum aktuellen Zeitpunkt de facto nicht möglich. Jeder, der plant produktive Workload im Cluster zu fahren sollte unbeindgt VMSS aktivieren, denn unweigerlich bringt diese Variante einige Vorteile mit sich.

**VMSS Vorteile**   
 - mehrere Node-Pools
 - automatische Skalierung der Node-Pools
 - Spot Instanzen **(!)**

an dieser Stelle fragt ihr euch bestimmt:  
> **"Wozu sollte ich nun einen Cluster ohne VMSS erstellen?"**

Das ist einfach beantwortet, durch den Einsatz von VMSS können die von den VMs verwendeten OS-Disken nicht geändert werden. Als Standard werden pro Worker 128 GB große Premium SSD Disken in einsatz gebracht. Allein die Kosten für eine dieser Disken belaufen sich auf **~20 €** pro Monat. Dies fällt bei der Kalulation einer nur kleinen Testumgebung enorm ins Gewicht!

{{< figure src="/images/aks-vmss-disabled.png" alt="aks" width="700px" >}}

Wählt man hingegen die Variante **ohne VMSS** hat man die möglichkeit, wenn auch nur manuel, den Typ der Disk nach Erstellung des Kubernetes-Cluster zu verändern. So kann diese auf Beispielsweise Standard-HDD, welche lediglich Kosten von **~5€** im Monat verursacht, verändert werden.  
Für zwei Nodes können wir so bereits *rund 30€ sparen*! Wow!

**Keine Spot Instanzen**  
Der Einsatz von Spot-Instanzen bietet hingegen nur für größere Umgebungen einen relevanten Vorteil. Das ergibt sich aus den folgenden Merkmalen:

- Spot-Instanzen sind nur in der VMSS Variante verfügbar
- Der Default-Pool darf nicht aus Spotz-Instanzen bestehen. Man benötigt also mindestens eine "normale" Instanz *(MS empfiehlt 3)*.
- Es sind keine B*xx* Instanzen verfügbar, sondern nur teurere A oder DS Instanz-Typen.

Die mit der Sport-Instanz verbundenen Restriktionen rechtfertigen den Einsatz von VMSS *(welcher Aufgrund er Disken teurer ist)* nicht.

##Noch mehr sparen
Azure bietet uns noch eine weitere Möglichkeit die Kosten für eine Test-Kubernetes-Umgebung gering zu halten. Jene des **Load-Balancers**.

Per Default wird der Typ "Standard" in Einsatz gebracht. Die Kosten für einen Standard-LB belaufen sich auf ungefähr 18€ pro Monat. Hingegen ist der Typ "Basic" absolut kostenfrei verfügbar. Der Typ des Loadbalancers kann nur bei der Erzeugung des CLuster gewählt werden, auch nicht via Annotation bei der Installation eines Ingress-Controllers. Für den Einsatz eines herkömmlichen Ingress-Controllers reicht das Feature-Set des Basic-Loadbalancers aus. 

**Bei einem Cluster mit 2 Nodes können wir also: <ins>~50€</ins> monatlich sparen.**  
*Nicht schlecht!*

Zum Abschluss das Command welches eines derartigen Clusters erstellt:
```
az aks create --resource-group K8s-RG --name K8s --network-plugin azure \
 --vnet-subnet-id /subscriptions/#ID#/resourceGroups/K8s-RG/providers/Microsoft.Network/virtualNetworks/vNET-K8s/subnets/Pod-Network \
 --docker-bridge-address 172.17.0.1/16 --dns-service-ip 11.1.0.10 --service-cidr 11.1.0.0/16 --generate-ssh-keys \
 --kubernetes-version 1.16.9 --node-vm-size Standard_B2s --node-count 2 --location northeurope \ 
 --vm-set-type AvailabilitySet --load-balancer-sku basic --load-balancer-managed-outbound-ip-count 1 \
 --max-pods 110 --subscription #ID#
```
*IDs sind entsprechend zu adaptieren.*

## Disclaimer
Wichtig ist es mir zu erwähnen:
- Sämtliche Werte gelten nur als Richtwert und sind je nach Anwendungsfall individuell zu berechnen.
- Microsoft Azrue ändert regelmäßig Ihre Services und Preise, vorliegender Artikel kann nur als Momentaufnahme verstanden werden
- Die Implementierung einer derartigen Umgebung beduetet bewusst auf den verzicht von SLAs und sollte keinesfalls produktiv eingesetzt werden.
- Du bist bei der MS tätig und/oder hast noch Tipps um einen Dev-Cluster noch günstiger zu gestalten. Ich bin für Input offen! Get in touch!
- Irrtümer vorbehalten