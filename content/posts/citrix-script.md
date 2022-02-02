---
slug: citrix-xendesktop-detail-script
draft: false
title: Citrix User-Detail Script
date: 2017-08-15T22:40:32.169Z
tags:
  - Citrix
  - XenDesktop
  - XenApp
  - PowerShell
---
Es ist nicht immer einfach Benutzer-Eigenschaften via **Powershell** abzufragen. Damit das Rad nicht jedes mal neu erfunden werden muss, habe ich ein Script für Citrix Xendesktop & XenApp 7 geschrieben, welches dies extrem vereinfacht.

{{< figure src="/images/citrix-logo-black.png" alt="azure aks" width="300px" >}}

Unter folgendem Link findet Ihr das Script  
  **[berndinox/Citrix-XD7-UserDetails](https://github.com/Berndinox/Citrix-XD7-UserDetails)** @ Github

Ihr könnte die Funktion in euer Script kopieren und von dieser Gebrauch machen oder das Script einfach erweitern.

Hier einige Beispiele: 

**`Get-CtxUser`**

```
ClientAddress         : 10.0.0.1
ClientName            : LAPTOP
ClientVersion         : 4.6.0.1
ConnectedViaIpAddress : 1.2.3.4
UserName              : USER
PublishedName         : CitrixApp
UserLogonTime         : 2017,2,24
UserSid               : S-1-5-21-123456789
DomainName            : DOMAIN
```

Ok, nicht schlecht! Was geht noch?

`Get-CtxUser -log 1 -CTXItems “ClientAddress”`  
`Get-CtxUser -CTXItems “ClientName”,”UserSid”`

oder:

```
if (Get-CtxUser | Where-Object {$_.Name -contains "10.0.0"})
{Write-Host "Connected from Internal!"}
```

Hier eine Liste der **Eigenschaften**, welche abgefragt werden können:

- SessionGUID
- UserLogonTime
- ClientAddressFamily
- ClientAddress
- INetClient
- EncryptionLevel
- ClientProductId
- SerialNumber
- HRes
- VRes
- ClientBuildNumber
- ClientHardwareId
- RDSCalId
- ClientName
- WorkDirectory
- ClientLicense
- ClientDirectory
- AudioDriverName
- WanScalersPresent
- NegotiatedColorDepth
- ColorDepth
- AppState
- AppStateChangeTime
- DomainName
- UserName
- InitialProgram
- PublishedName
- IsBrokered
- fPublishedApp
- CtxSessionKey
- SessionState
- IsAnonymousSession
- UserSid
- ConnectionMode
- SessionToLingerAfterLastApp
- ProtocolType
- UserShadowSettings
- OEMID
- ClientCodePage
- ClientVersion
- ClientType
- UIModuleVersionH
- UIModuleFlags
- UIFontSmoothing
- UIPreferredIconBPP
- KeyboardLayout
- WDICABUfferLength
- WDModuleVersionH
- WdFlag
- WdInputBufferLength
- WDName
- WdDLL
- PSPath
- PSParentPath
- PSChildName
- PSDrive
- PSProvider

 _**Achtung:**  Getestet auf Windows 2012R2 + Xendesktop 7.6_



