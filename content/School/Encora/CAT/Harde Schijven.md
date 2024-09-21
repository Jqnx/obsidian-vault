---
title: Harde Schijven
draft: false
date created: 2024-09-21T00:49:38
date modified: 2024-09-21T17:12:50
tags:
  - encora
  - hardware
  - assemblage
---


# **COMPUTER ASSEMBLAGE**

## Harde Schijven

Harde schijven (ook SSD en flash schijven) zijn bedoeld om data op langere termijn op te slagen, ongeacht of de computer in-of uitgeschakeld is.  
We spreken dan van niet-vluchtig geheugen (non-volatile memory).

---

## Connectoren

### IDE

#### *PATA (Parallel Advanced Technology Attachment)*

 Dient voor Harde schijven en CD/DVD-ROM's aan te sluiten. Heeft 40 pinnen per aansluiting.  
 Kan twee apparaten aansluiten per kabel.  
 Krijgt ook stroom doormiddel van en ***Molex***-connector. Werkt met *master en slave*.  
 Twee masters of slaves aansluiten of dezelfde kabel gaat niet. Daarom dat een van de masters world uitgeschakeld.

#### *SATA (Serial Advanced Technology Attachment)*

 Vormfactor van de kabel is kleiner dan PATA. Bestaat uit 7-pinnen en gebruikt een 15-pin connector om stroom te leveren.  
 Kan harde schijven toe voegen en verwijderen tijdens dat de computer aan staat: ***Hot-Plugging***.  
 Maakt ook gebruik van ***NCQ*** (Native Command Queuing), dit zorgt ervoor dat harde schijven intern de volgorde van lees-en schrijfcommando’s kunnen optimaliseren.  

 Er zijn 3 verschillende versies van SATA:
 > SATA I : 1.5Gbit/s of 150MB/s  
 > SATA II : 3Gbit/s of 300MB/s  
 > SATA III : 6Gbit/s of 600MB/s  
 > Deze interfaces zijn ook backwards compatible.

### SCSI

SCSI gebruikt een controller om data en stroom te verzenden en ook te ontvangen.

#### *Parallel SCSI*

 Dient voor harde schijven, scanners en printers aan te sluiten. Connectoren bestaan uit 50, 68 of 80 pinnen.

#### *Serial SCSI*

Serial SCSI of SAS (Serial Attached SCSI) is de opvolger van Parallel SCSI.  
SAS verzend en ontvangt data zoals harde schijven en tape drives.  
Er zijn verschillende connectoren:  

- SFF-8087 Connector
- SFF-8088 External SAS Connectors
- SFF-8470 Infiniband CX4 Connectors
- SFF-8482 Connectors
- SFF-8484 Connector

Er zijn ook verschillende SAS-interfaces:  
> SAS-1 : 3 Gbit/s  
> SAS-2 : 6 Gbit/s  
> SAS-3 : 12 Gbit/s  
> SAS-4 : 22.5 Gbit/s
---

## SATA Controller Modes

De SATA controller mode bepaald hoe de harde schijf gaat communiceren met de computer.  
De meest voorkomende modes zijn:

### *IDE*

De harde schijf werkt zoals een PATA HDD, zonder de voordelen van SATA:  snelheid, hot plugging en NCQ.

### *AHCI*

De harde schijf zal de SATA features (hot plugging en NCQ) wel gebruiken en zal ook sneller werken.

### *RAID*

Zowel AHCI en RAID features worden gebruikt. Voor RAID zie RAID onder Randapparatuur (RA).

---

## Hard Disk Drives (Mechanische schijven)

5400 RPM
7200 RPM
10000 RPM

---

## Solid State Drives (SSD)

---