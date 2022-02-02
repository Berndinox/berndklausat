---
slug: wordpress-serverless-highspeed
draft: false
title: Serverless WordPress - High-Level
date: 2020-06-01T22:40:32.169Z
tags:
  - WordPress
  - Serverless
  - Cloudflare
  - Javascript
---
## Intro
Im weiterem Artikel möchte ich, wie im Teaser erwähnt, die Entwicklung meiner WordPress Seite darstellen. Jedoch kommen für viele andere Anwendungen und Web-Applicationen ähnliche Entwicklungsstufen zum Einsatz. Deshalb eignet sich der folgende Beitrag auch hervorragend um einen Einblick in die unterschiedlichen Technologien und Methoden, wie sie heute zum Einsatz kommen, zu gewinnen.

## Der Beginn
Vor bereits einigen Jahren begann ich mit dem **Hosting auf einem einfachen virtuellen Server**. Dabei werden die notwendigen Abhängigkeiten wie eine WebServer-Software und der passende PHP-Compiler auf diesem Server installiert und konfiguriert. Anschließend wird WordPress entsprechend aufgebracht. Zum Veranschaulichen eine schematische Darstellung:

{{< figure src="/images/wp_simple.png" alt="swwp-simple" width="180px" >}}

Das Setup ist vergleichsweise einfach, jedoch ergeben sich auch einige Nachteile:
- geringe Skalierbarkeit
- keine Versionierung
- Abhängigkeit zu Anbieter
- schlechtere Uptime

## Kubernetes und Docker Swarm
Als *scheinbarer* Retter biete sich **Kubernetes** oder alternativ Docker Swarm an. Tatsächlich lösen diese Technologien die vorher aufgelisteten Nachteile. Beginnen wir mit der grafischen Übersicht.

{{< figure src="/images/wp_kubernetes.png" alt="k8s" width="250px" >}}

Kubernetes abstrahiert eine Anzahl *N* an Servern und erhöht damit die mögliche Uptime zu einem theoretischen Wert von 100%, sowie die Möglichkeit der Skalierung. Auch können Abhängigkeiten zu Anbietern stark minimiert werden, da Kubernetes eine plattformunabhängige Schnittstelle bietet. Durch die Integration eine Entwicklungspipeline ermöglicht man zu guter letzt eine Versionierung, zumindest jene der Konfigurationsitems. *"Cool!"*

Ihr ahnt es vermutlich, denn es gibt immer noch die ein oder anderen  
 **"Pain-Points"**:
- Code *(PHP)* wird bei jeden Besuch ausgeführt
- die Datenbank *(MySQL)* muss stets verfügbar sein
- WordPress ist uU. anfällig auf Attacken: SQL-Injection, XSS-Attcken, usw.
- ReadWriteMany Volumes in der Kubernetes-Umgebung sind *erforderlich*

## Headless CMS & Serverless
*Wie? Ein kopfloses Contant-Management-System und das ohne Server!* **Genau!**  

Der neueste Trend, Serverless oder auch FaaS, kann die zuvor erwähnt Probleme weiter reduzieren bzw. verbessern. Wie das funktioniert lässt sich am einfachsten in folgender Grafik veranschaulichen.

{{< figure src="/images/wp_serverless.png" alt="serverless wordpress" width="490px" >}}

Schnell lässt sich der entscheidende Unterschied eruieren. Der zugreifende Benutzer und WordPress selbst haben nichts mehr miteinander zu tun.  

WordPress dient lediglich als Editor für den Inhalt der Homepage. Via Static-Site-Generator *(SSG)* wird der Inhalt von WordPress via Json-API ausgelesen und als fertige Website via Github weiter verteilt. Als SSG kommt das auf React basierende JS-Framework **[Gatsby](https://www.gatsbyjs.org/Gatsby)** zum Einsatz. Anschließend wird die fertige Seite via **[Github Actions](https://github.com/features/actions)** vollautomatisch auf serverlose Umgebung von **[Cloudflare](https://www.cloudflare.com/de-de/products/cloudflare-workers/)** transferiert. 

Das hört sich kompliziert an, läuft nach einmaliger Einrichtung jedoch ohne irgendein zutun. 

Die daraus resultierenden **Vorteile**:
- Weltweit auf 200 POPs verteilt; *fantastische Zugriffszeiten*.
- Basierend auf "Progressive Web-Applikation" Technologie
- *Theoretisch* unendliche Skalierbarkeit
- Kosteneffizient
- Nicht anfällig auf Sicherheitslücken in WordPress

Ok, lassen wir die **Zahlen** sprechen:
{{< figure src="/images/loadtest-bk.png" alt="swarm" width="720px" >}}

Quelle: [Sucuri Loadtest](https://performance.sucuri.net/domain/berndklaus.at)

Das arithmetische Mittel der Ladezeit beläuft sich auf: <mark>**0,253s**</mark>  
<sub>Anmerkung: die angegebene Zeit bezieht auf die vollständig fertig geladene Seite.</sub>  

Ohne zu übertreiben, das ist: **Fuc*ing Awesome!**
## Resümee
Neue Technologien bieten neue Möglichkeiten, aber auch neue Herausforderungen. Wie man tatsächlich seine WordPress-Seite in eine serverless JS-Seite verwandelt wird im nächsten Artikel, auf technischer Ebene, beschrieben. Stay tuned.
