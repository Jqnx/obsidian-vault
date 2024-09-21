---
title: IPv4 Fragmentatie
draft: false
tags:
  - encora
  - netwerktechnologie
  - networking
  - ipv4
  - icmp
---

IP Fragmentatie zorgt ervoor dat je IP-pakket verdeeld wordt in kleinere deeltjes, afhankelijk van de **MTU (Maximum Transmission Unit)**.

Je pakket wordt pas echt gefragmenteerd al voldoet het aan de volgende voorwaarden:
- Het is groter dan de MTU
- De waarde van de [DF bit](IPv4.md#Flags) in de IP header.
De [MF of More Fragment](IPv4.md#Flags) bit is altijd 1 buiten bij het laatste fragment.
 
---
## Data grootte berekenen

1. Start door *$MTU - Header Length$* te doen
2. Zoek naar de grootste waarde die deelbaar is door 8 en kleiner dan de vorige waarde.
3. Dit is de Data size

> [!hint] ICMP Header
> In het geval dat je pingt krijg je een ICMP header en een IP Header in je eerste fragment.
> De ICMP header is 8 bits groot.

## Fragment Offset berekenen

1. Neem de *Data Size*
2. Deel deze door 8
3. Dit de de Fragment Offset

## Total Length berekenen

1. Neem de *Data Size*
2. Voeg de *Header Length* toe
3. Dit is de Total Length van je nieuwe IP-Pakket

## Ethernet Frame berekenen

1. Neem de *Total Length*
2. Voeg **18** (14 van de Mac Header en 3 van de CRC) toe
3. Dit is de totale lengte van de Ethernet Frame


> [!hint] ICMP Header
> In het geval dat je een ICMP header hebt voeg je die ook toe.
> De ICMP header is 8 bits groot.
