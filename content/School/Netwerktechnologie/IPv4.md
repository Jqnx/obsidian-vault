## Wat is IPv4?

IPv4 is de meest gebruikte versie van het **Internet Protocol**. Het bestaat uit **32-bit** of 2<sup>32</sup>.
Het adres bestaat uit decimale getallen, gescheiden door een punt. bv. **192.168.50.55**

***
## Structuur van de IPv4 header

Een IPv4 header bestaat uit 20 bytes.

![[IPv4 Header.png]]
### Version
Dit veld gaat over de versie van het IP protocol. Het veld is **4 bits** groot.

### Header Length of IHL
Dit veld geeft de lengte van de IPv4 header aan. Het is ook **4 bits** groot. Het wordt uitgedrukt in een aantal **32 bit oftewel 4 bytes** dus bv. (*5 * 4 bytes = 20 bytes*).

### ToS of Type of Service
Dit veld dient voor real-time services zoals voip of video conferencing. Het is **8 bits** groot.

### Total Length
Dit veld geeft de lengte weer van het hele IP-Pakket (IP-Header en IP-Payload). Dit veld is **16 bits** groot. De uiterst maximum grootte is 65535 bytes maar de normale maximum grootte is **1500 bytes**.

### Identification
Dit veld dient ervoor zodat fragmenten uiteindelijk correct terug kunnen worden samengevoegd. Dit veld is **16 bits** groot.

### Flags
Dit veld dient voor fragmentatie. Het is **3 bits** groot.
Je hebt de 0 bit, de DF (dont fragment) bit en de MF (more fragments) bit.

0 = Altijd 0
DF = 0 (fragment), 1 (dont fragment)
MF = 1 bij elk fragment buiten het laatste fragment.

### Fragment Offset
Dit veld dient ook voor fragmentatie. Het geeft aan waar in de datagram het fragment thuis hoort. Dit veld is **13 bits** groot. Bij het eerste fragment is dit altijd 0 en bij de anderen moet dit altijd deelbaar zijn door 8.
[[IPv4 Fragmentatie#Fragment Offset berekenen|Fragment Offset Berekenen]]

### TTL of Time to Live
Dit veld geeft aan bij hoeveel routers het voorbij kan passeren (hoppen) voor dat het gedumped wordt. Dit veld is **8 bit** groot.

### Protocol
Dit veld geeft aan of het over TCP/UDP/ICMP gaat. Het is **8 bits** groot.

TCP = **06** in beide decimaal en hexadecimaal
UDP = 17 in decimaal en **11** in hexadecimaal
ICMP = **01** in beide decimaal en hexadecimaal

### Header Checksum
Dit veld gaat over de foutcontrole van de header. Het is **16 bits** groot.
De checksum wordt berekent door alle bytes per 2 op te tellen.
**Je pakt dus paren van 16 bits om op te tellen.**

### Source IP Address
Dit veld is het ip adres van de verzender. Het is **32 bits** groot.

### Destination IP Address
Dit veld is het ip adres van de ontvanger. Het is **32 bits** groot.