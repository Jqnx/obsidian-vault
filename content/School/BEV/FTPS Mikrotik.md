---
title: FTPS Mikrotik
draft: true
tags:
  - school
  - beveiliging
  - filezilla
  - ftp
---

We moeten 2 verschillende NAT regels gebruiken:
- 1 voor de command channel
- 1 voor de data channel

## 1. Command channel nat rule
```shell
/ip firewall nat
add chain=dstnat action=dst-nat to-addresses=192.168.88.254 protocol=tcp in-interface-list=WAN dst-port=21
```

## 2. Data channel nat rule
```shell
/ip firewall nat
add chain=dstnat protocol=tcp dst-port=60000-60100 in-interface-list=WAN action=dst-nat to-addresses=192.168.88.254 to-ports=60000-60100
```

## 3. Passive mode filezilla server
Het is ook belangrijk om gemarkeerde lijn uit te zetten al is de WAN een lokaal netwerk.

![[Pasted image 20240617142536.png]]