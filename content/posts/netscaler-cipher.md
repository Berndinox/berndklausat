---
template: post
slug: citrix-netscaler-cipher-group-fix
draft: false
title: Netscaler Cipher Issue
date: 2018-01-10T22:40:32.169Z
tags:
  - Citrix
  - Netscaler
  - Proxy
  - SSL
---
## Issue
Erst kürzlich stand ich vor einem verheerenden Problem. Verbindungen auf die Citrix Farm über das Externe Netscaler Gateway brachen nach 10-30 Minuten ab. Je höher die Bandbreite geht, desto schneller geshah dieses Phänomen. Dabei konnten wir einige TCP Retransmission beobachten, beendet wird die Verbindung mit einem RST.

{{< figure src="/images/netscaler.png" alt="iota" width="250px" >}}

Zum Einsatz kommt: **Netscaler MPX v 11.1**
Die Lösung dieses Problem konnte nach gefühlter Ewigkeit wie folgt gefunden werden.
Wireshark-Traces auf einem Span-Port des Externen Interfaces der Netscaler MPX Appliance zeigen den folgenden Eintrag:

![wireshark](/images/wireshark_ns_trace.png "wireshark")

Ausschlaggebend ist hier der Wert **“W=9820”**. Anhand des folgenden Blog Eintrages von Citrix ergibt sich folgende Error-Beschreibung:

**“SSL Card Operation failed”**  
*Quelle: https://www.citrix.com/blogs/2014/05/20/whats-that-netscaler-reset-packet/*

## Cipher Groups
Das ließ uns auf die Cipher-Groups aufmerksam werden, welche jedoch bereits durch den Citrix Support verifiziert und überprüft wurden, anbei die Liste:

```
bind ssl cipher SecureCipherGroup -cipherName TLS1-DHE-DSS-AES-256-CBC-SHA
bind ssl cipher SecureCipherGroup -cipherName TLS1-DHE-DSS-AES-128-CBC-SHA
bind ssl cipher SecureCipherGroup -cipherName TLS1-DHE-RSA-AES-256-CBC-SHA
bind ssl cipher SecureCipherGroup -cipherName TLS1-DHE-RSA-AES-128-CBC-SHA
bind ssl cipher SecureCipherGroup -cipherName TLS1-ECDHE-RSA-DES-CBC3-SHA
bind ssl cipher SecureCipherGroup -cipherName TLS1-ECDHE-RSA-AES128-SHA
bind ssl cipher SecureCipherGroup -cipherName TLS1-ECDHE-RSA-AES256-SHA
bind ssl cipher SecureCipherGroup -cipherName TLS1.2-AES128-GCM-SHA256
bind ssl cipher SecureCipherGroup -cipherName TLS1.2-AES256-GCM-SHA384
bind ssl cipher SecureCipherGroup -cipherName TLS1.2-DHE-RSA-AES128-GCM-SHA256
bind ssl cipher SecureCipherGroup -cipherName TLS1.2-DHE-RSA-AES256-GCM-SHA384
bind ssl cipher SecureCipherGroup -cipherName TLS1.2-ECDHE-RSA-AES128-GCM-SHA256
bind ssl cipher SecureCipherGroup -cipherName TLS1.2-ECDHE-RSA-AES256-GCM-SHA384
bind ssl cipher SecureCipherGroup -cipherName TLS1.2-ECDHE-RSA-AES-128-SHA256
bind ssl cipher SecureCipherGroup -cipherName TLS1.2-ECDHE-RSA-AES-256-SHA384
bind ssl cipher SecureCipherGroup -cipherName TLS1.2-AES-256-SHA256
bind ssl cipher SecureCipherGroup -cipherName TLS1.2-AES-128-SHA256
bind ssl cipher SecureCipherGroup -cipherName TLS1.2-DHE-RSA-AES-128-SHA256
bind ssl cipher SecureCipherGroup -cipherName TLS1.2-DHE-RSA-AES-256-SHA256
bind ssl cipher SecureCipherGroup -cipherName TLS1-AES-256-CBC-SHA
bind ssl cipher SecureCipherGroup -cipherName TLS1-AES-128-CBC-SHA
bind ssl cipher SecureCipherGroup -cipherName SSL3-DES-CBC3-SHA
bind ssl cipher SecureCipherGroup -cipherName SSL2-DES-CBC3-MD5
bind ssl cipher SecureCipherGroup -cipherName SSL3-EDH-DSS-DES-CBC3-SHA
bind ssl cipher SecureCipherGroup -cipherName SSL3-EDH-RSA-DES-CBC3-SHA
```

Mit einem weiteren Trace konnten wir die verwendete Cipher-Group ausfindig machen:

```
Server Hello:
TLS - Transport Layer Security
  TLSv1.2 Record Layer
    Content Type:       22  Handshake Protocol [58]
    Version:            3.3    TLS 1.2 [59-60]
    Length:             2943  bytes  (length is larger than remaining size of packet) [61-62]
    Handshake Protocol
      Handshake Type:   2    Server Hello [63]
        Cipher Suite:   156  TLS_RSA_WITH_AES_128_GCM_SHA256 [134-135]
        Compression Method:0 [136]
FCS - Frame Check Sequence
```
## Solution
Scheinbar ist also die Cipher-Group **`TLS_RSA_WITH_AES_128_GCM_SHA256`** der Übeltäter.
Nachdem wir diese Cipher-Group Netscaler-seitig deaktivierten konnte die Umgebung problemlos betrieben werden.