---
title: Project Docker en Kubernetes
draft: false
tags:
  - school
  - project
  - linux
  - docker
  - k8s
  - containerd
  - github-actions
  - cert-manager
  - metallb
  - istio
  - git
---

![[voorblad v2.jpg]]

<div class="page-break" style="page-break-before: always;"></div>

- [[#Inleiding|Inleiding]]
- [[#Netwerkschema|Netwerkschema]]
- [[#1. Router|1. Router]]
	- [[#1. Router#1.1 Opzoeken handleiding|1.1 Opzoeken handleiding]]
	- [[#1. Router#1.2 Instellen statisch IP|1.2 Instellen statisch IP]]
	- [[#1. Router#1.3 Setup Wizard|1.3 Setup Wizard]]
		- [[#1.3 Setup Wizard#1.3.1 WAN configuratie|1.3.1 WAN configuratie]]
		- [[#1.3 Setup Wizard#1.3.2 LAN configuratie|1.3.2 LAN configuratie]]
		- [[#1.3 Setup Wizard#1.3.3 Instellen DHCP windows|1.3.3 Instellen DHCP windows]]
	- [[#1. Router#1.4 Instellen Wireguard VPN|1.4 Instellen Wireguard VPN]]
		- [[#1.4 Instellen Wireguard VPN#1.4.1 Installeren Wireguard|1.4.1 Installeren Wireguard]]
		- [[#1.4 Instellen Wireguard VPN#1.4.2 Aanmaken Wireguard keys|1.4.2 Aanmaken Wireguard keys]]
		- [[#1.4 Instellen Wireguard VPN#1.4.3 Aanmaken Wireguard interface|1.4.3 Aanmaken Wireguard interface]]
		- [[#1.4 Instellen Wireguard VPN#1.4.3 Instellen client|1.4.3 Instellen client]]
- [[#2. ESXi|2. ESXi]]
	- [[#2. ESXi#2.1 ESXi installeren|2.1 ESXi installeren]]
	- [[#2. ESXi#2.2 ESXi instellingen|2.2 ESXi instellingen]]
	- [[#2. ESXi#2.3 VMs aanmaken|2.3 VMs aanmaken]]
	- [[#2. ESXi#2.4 VMs configureren|2.4 VMs configureren]]
- [[#3. Gemeenschappelijke configuratie|3. Gemeenschappelijke configuratie]]
	- [[#3. Gemeenschappelijke configuratie#3.1 Docker|3.1 Docker]]
		- [[#3.1 Docker#3.1.1 Wat is docker?|3.1.1 Wat is docker?]]
		- [[#3.1 Docker#3.1.2 Docker installeren|3.1.2 Docker installeren]]
	- [[#3. Gemeenschappelijke configuratie#3.2 Containerd|3.2 Containerd]]
		- [[#3.2 Containerd#3.2.1 Wat is containerd?|3.2.1 Wat is containerd?]]
		- [[#3.2 Containerd#3.2.2 Waarom containerd over Docker als CRI|3.2.2 Waarom containerd over Docker als CRI]]
		- [[#3.2 Containerd#3.2.3 Containerd instellen|3.2.3 Containerd instellen]]
	- [[#3. Gemeenschappelijke configuratie#3.3 Kubernetes|3.3 Kubernetes]]
		- [[#3.3 Kubernetes#3.3.1 Wat is kubernetes?|3.3.1 Wat is kubernetes?]]
		- [[#3.3 Kubernetes#3.3.2 Installeren kubeadm, kubelet en kubectl|3.3.2 Installeren kubeadm, kubelet en kubectl]]
		- [[#3.3 Kubernetes#3.3.2.1 Uitzetten SWAP|3.3.2.1 Uitzetten SWAP]]
		- [[#3.3 Kubernetes#3.3.2.2 Installeren pakketten|3.3.2.2 Installeren pakketten]]
- [[#4. Configuratie master node|4. Configuratie master node]]
- [[#5. Configuratie worker nodes|5. Configuratie worker nodes]]
- [[#6. Configuratie kubernetes cluster|6. Configuratie kubernetes cluster]]
	- [[#6. Configuratie kubernetes cluster#6.1 MetalLB|6.1 MetalLB]]
	- [[#6. Configuratie kubernetes cluster#6.2 Cert-manager|6.2 Cert-manager]]
		- [[#6.2 Cert-manager#6.2.1 Installatie cert-manager|6.2.1 Installatie cert-manager]]
		- [[#6.2 Cert-manager#6.2.2 Aanvragen certificaten|6.2.2 Aanvragen certificaten]]
	- [[#6. Configuratie kubernetes cluster#6.3 Istio en GatewayAPI|6.3 Istio en GatewayAPI]]
		- [[#6.3 Istio en GatewayAPI#6.3.1 Installatie Istio en GatewayAPI|6.3.1 Installatie Istio en GatewayAPI]]
		- [[#6.3 Istio en GatewayAPI#6.3.2 Creatie gateway|6.3.2 Creatie gateway]]
- [[#7. Setup website|7. Setup website]]
	- [[#7. Setup website#7.1 Dockerfile|7.1 Dockerfile]]
	- [[#7. Setup website#7.2 Github Actions|7.2 Github Actions]]
		- [[#7.2 Github Actions#7.2.1 Inleiding|7.2.1 Inleiding]]
		- [[#7.2 Github Actions#7.2.2 Github repository|7.2.2 Github repository]]
		- [[#7.2 Github Actions#7.2.3.1 Workflow|7.2.3.1 Workflow]]
		- [[#7.2 Github Actions#7.2.3.2 Build en push docker image|7.2.3.2 Build en push docker image]]
		- [[#7.2 Github Actions#7.2.3.3 Deploy naar kubernetes|7.2.3.3 Deploy naar kubernetes]]
		- [[#7.2 Github Actions#7.2.4 Service account|7.2.4 Service account]]
		- [[#7.2 Github Actions#7.2.5 Portforwarding kube-apiserver|7.2.5 Portforwarding kube-apiserver]]
		- [[#7.2 Github Actions#7.2.6 Updaten website|7.2.6 Updaten website]]
- [[#Troubleshooting|Troubleshooting]]
	- [[#Troubleshooting#1. SWAP|1. SWAP]]
	- [[#Troubleshooting#2. Metallb|2. Metallb]]
	- [[#Troubleshooting#3. Github actions|3. Github actions]]
- [[#Sources|Sources]]
- [[#Agenda|Agenda]]
- [[#Besluit|Besluit]]

<div class="page-break" style="page-break-before: always;"></div>

## Inleiding
Ik heb voor het project *Docker en Kubernetes* gekozen. Vooral omdat ik al wat ervaring met docker had door mijn homelab en al een tijdje kubernetes wou leren, maar hier nooit echt een goede reden voor heb gehad. In dit project ga ik een kubernetes cluster opzetten die een website draaiende houd en extern bereikbaar maakt. Ik ga ook zelf een docker image maken en ervoor zorgen dat deze automatisch wordt uitgerold met gebruik van github actions.

<div class="page-break" style="page-break-before: always;"></div>

## Netwerkschema
![[Netwerkschema_Project_Jan_v3_transparant.png|525]]

<div class="page-break" style="page-break-before: always;"></div>

## 1. Router
### 1.1 Opzoeken handleiding
De handleiding is terug te vinden op deze link:
https://dl.ubnt.com/guides/edgemax/EdgeRouter_ER-X_QSG.pdf

### 1.2 Instellen statisch IP
We starten met de router te resetten door de reset knop 10 seconden lang ingedrukt te houden tot de `eth4` LED begint te knipperen. We verbinden dan onze ethernet kabel met de `eth0/PoE In` poort op de router. Volgens de handleiding bevind de router zich standaard in het `192.168.1.x` subnet. We zullen dus onze computer een statisch IP-adres instellen in dit subnet.

![[1 - Router - Static IP Windows.png]]
<div class="page-break" style="page-break-before: always;"></div>

### 1.3 Setup Wizard
#### 1.3.1 WAN configuratie
We stellen hier het statisch IP-adres in dat we gekregen hebben aan het begin van de module. In dit geval is dit: `192.168.25.168/24`

![[2 - Router - WAN Config.png]]

#### 1.3.2 LAN configuratie
Vervolgens stellen we het subnet in dat we gekregen hebben in de opgave: `192.168.10.0/24` , we zetten de DHCP server ook aan.

![[3 - Router - LAN Config.png]]
<div class="page-break" style="page-break-before: always;"></div>

#### 1.3.3 Instellen DHCP windows
Als we klaar zijn met de setup wizard moeten we onze computer terug zetten op DHCP.

![[4 - Router - DHCP Windows.png]]

### 1.4 Instellen Wireguard VPN
Omdat ik persoonlijk liever met mijn VM's zou verbinden via SSH dan via het controle paneel van ESXI, heb ik een Wireguard VPN opgezet op de router, zodat ik controle kan hebben aan mijn hele netwerk. Dit is ook handig om vanop een afstand aan de router te kunnen, aangezien dit anders niet mogelijk zou zijn.

<div class="page-break" style="page-break-before: always;"></div>

#### 1.4.1 Installeren Wireguard
Eerst openen we de terminal op onze router:

![[router-cli.png]]

Eenmaal als we met onze router verbonden zijn voeren de het volgende uit:
```shell
wget https://github.com/Lochnair/vyatta-wireguard/releases/download/0.0.20191219-2/wireguard-v2.0-e50-0.0.20191219-2.deb
sudo dpkg -i wireguard-v2.0-e50-0.0.20191219-2.deb
```
Het pakket dat we hier downloaden en installeren is afhankelijk van model, je kan hier een lijst vinden met uitleg: [Releases · Lochnair/vyatta-wireguard](https://github.com/Lochnair/vyatta-wireguard/releases). Het is ook afhankelijk van de versie van firmware die je draait, deze kan je terugvinden in de GUI: 

![[Router-firmware-version.png]]

#### 1.4.2 Aanmaken Wireguard keys
We starten door een paar keys aan te maken voor de router:
```shell
mkdir server_keys; cd server_keys
wg genkey | tee privatekey | wg pubkey > publickey
```

<div class="page-break" style="page-break-before: always;"></div>

Je kunt je keys nu raadplegen door hun bestand te lezen:
```shell
cat privatekey
cat publickey
```

We gaan terug naar onze home directory:
```shell
cd ~
```

Dan maken we een paar keys aan voor onze clients, in mijn geval 2 maar het proces is hetzelfde voor alle keys. We maken voor elke user ook een eigen subfolder aan:
```shell
mkdir peer_keys; cd peer_keys
mkdir thuis; cd thuis
wg genkey | tee privatekey | wg pubkey > publickey
```

Opnieuw kan je ze nu raadplegen door hun bestand te lezen:
```shell
cat privatekey
cat publickey
```

Terug naar de home directory:
```shell
cd ~
```

#### 1.4.3 Aanmaken Wireguard interface
We gebruiken de `configure` tool om de interface aan te maken:
```shell
configure

set interfaces wireguard wg0 address 10.4.20.1/24
set interfaces wireguard wg0 listen-port 25039 # Poort aangepast naar een poort uit de range die ik gekregen heb.
set interfaces wireguard wg0 route-allowed-ips true
set interfaces wireguard wg0 private-key /home/ubnt/server_keys/privatekey
```

<div class="page-break" style="page-break-before: always;"></div>

We voegen ook ineens onze peer/client toe:
```shell
set interfaces wireguard wg0 peer <peer-public-key> allowed-ips 10.4.20.11/32
set interfaces wireguard wg0 peer <peer-public-key> description Thuis
```

Als laatste moeten we UDP verkeer voor Wireguard toestaan in onze firewall:
```shell
set firewall name WAN_LOCAL rule 30 action accept  
set firewall name WAN_LOCAL rule 30 protocol udp  
set firewall name WAN_LOCAL rule 30 destination port 25039  
set firewall name WAN_LOCAL rule 30 description 'WireGuard'
```

Alles opslaan:
```shell
commit; save
```

#### 1.4.3 Instellen client
Nu dat onze interface op de router is aangemaakt en ingesteld moeten we onze client instellen. Wireguard heeft een client voor zo goed als elke OS, deze downloaden en installeren we. Dan moeten we onze client configuratie instellen, dit doen we via een .conf bestand dat we dan importeren:

```config
[Interface]
PrivateKey = <private key van de user>
ListenPort = 51820
Address = 10.4.20.11/32 # IP van de peer dat we eerder hebben ingesteld
DNS = 1.1.1.1 # DNS server naar keuze

[Peer]
PublicKey = <public key van de server>
PersistentKeepalive = 25

AllowedIPs = 10.4.20.0/24, 192.168.10.0/24 # Het wireguard subnet en het netwerk van mijn project

Endpoint = <public ip router>:25039 # Publiek ip van de router + de poort die we eerder hebben ingesteld
```
<div class="page-break" style="page-break-before: always;"></div>

>Sources:
>https://www.hostifi.com/blog/edgerouter-wireguard-remote-access-vpn
>https://www.erianna.com/wireguard-ubiquity-edgeos/
>https://github.com/Lochnair/vyatta-wireguard/releases
>https://www.wireguard.com/
---
## 2. ESXi
### 2.1 ESXi installeren

De ESXi installatie is vrij standaard:

1. Opstarten vanaf USB
2. EULA accepteren
3. De correcte schijf selecteren
4. Toetsenbord indeling kiezen
5. Ons root wachtwoord instellen
6. Systeem herstarten wanneer installatie klaar is

> Source: 
> [Install ESXi Interactively](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.esxi.install.doc/GUID-6FFA928F-7F7D-4B1A-B05C-777279233A77.html)

<div class="page-break" style="page-break-before: always;"></div>

### 2.2 ESXi instellingen

We starten met ons systeem een statisch ip adres te geven. 

![[ESXI 1 - Static IP Hypervisor.png]]

Als tweede stellen we een hostname, de dns servers en onze ipv4 gateway in. We stellen ook een domeinnaam en een zoek domein in, voornamelijk omdat deze opties verplicht zijn.

![[ESXI 2 - Aanpassen TCPIP Stack.png]]

Als derde stellen we in dat de NTP client automatisch moet opstarten. Dit zodat we automatisch de correcte tijd hebben op ons systeem.

![[ESXI 3 - NTP Server instellen.png]]

Als laatste uploaden we de ISO die we gaan gebruiken om onze VMs aan te maken.

![[ESXI 4 - Upload Ubuntu Server ISO.png]]

We voegen ook onze eerste port forwarding regel toe zodat we externe toegang hebben aan ons ESXi controle paneel.

![[Router - Portforwarding ESXI.png]]

<div class="page-break" style="page-break-before: always;"></div>

### 2.3 VMs aanmaken

We geven onze VM een naam en we stellen de OS opties in.

![[ESXI 1 - Template Name.png]]

Dan selecteren we de correcte opslag.

![[ESXI 2 - datastore.png]]

Vervolgens geven we de hardware van onze VM in. We selecteren ook onze ISO hier.

![[ESXI 3 - hardware.png]]

<div class="page-break" style="page-break-before: always;"></div>

### 2.4 VMs configureren

Zo ziet onze IP configuratie er uit. Alles is hetzelfde voor alle 4 VMs dat we aanmaken buiten het `Address: 192.168.10.20` veld in de IP Configuratie.

![[Ubuntu Setup 1 - IPv4 Config.png]]

We breiden onze standaard partitie uit van `14.996G` naar `29.996G`. Voor de rest passen we hier niets aan.

![[Ubuntu Setup 2 - Disk Partitioning.png]]

Hier geven we onze gebruikers info en hostname in.

![[Ubuntu Setup 3 - User Creation.png]]

Aangezien ik persoonlijk liever SSH gebruik dan het ESXi controle paneel voeg ik hier ook mijn SSH keys toe vanaf github.

![[Ubuntu Setup 4 - SSH Install.png]]

<div class="page-break" style="page-break-before: always;"></div>

---

## 3. Gemeenschappelijke configuratie

### 3.1 Docker

#### 3.1.1 Wat is docker?

Docker is een open-source platform voor het ontwikkelen, verzenden en draaien van applicaties. Dit gebeurd aan de hand van *containers*, *images* en *registries*. 
- Een *container* is een geïsoleerde omgeving om code uit te testen/applicaties te draaien, i.e. een mini VM of een geïsoleerde applicatie.
- Een *image* is een gestandaardiseerd pakket dat alle bestanden, libraries en configuratie bevat. Het is een soort sjabloon met instructies voor het maken van *containers*.
- Een *registry* is waar je deze images kan bewaren en delen. Er zijn publieke en privé registries.

![[Docker-overview.png]]

<div class="page-break" style="page-break-before: always;"></div>

Docker als een applicatie maakt gebruik van de Docker client (`docker`) en de Docker daemon (`dockerd`).
- De docker client (`docker`) is de hoofd manier waarop je docker zou gebruiken. Dit biedt de commando's aan die je gebruikt, e.g. `docker run`, `docker build`, `docker ps`, `docker logs`, etc.
- De docker daemon (`dockerd`) is backend waar dat de docker client je commando's naar toe stuurt. Dit is het deel van docker dat alles uitvoert en beheerd, e.g. images, containers, netwerken, en volumes.

![[Docker-architecture.png]]

Docker is een zeer krachtige tool voor programmeurs en wordt voornamelijk gebruikt in continuous integration and continuous delivery (CI/CD) pipelines. Dit heeft als doel om het testen en implementeren van code sneller en automatisch te maken.

Bovendien heeft docker ook ingebouwde netwerk capaciteiten.

>[!tip] Docker Desktop
>Er bestaat ook een Docker Desktop applicatie met een GUI.
>[Docker Desktop: The #1 Containerization Tool for Developers | Docker](https://www.docker.com/products/docker-desktop/)

> Sources:
> [Docker overview | Docker Docs](https://docs.docker.com/get-started/overview/)
> [What is a container? | Docker](https://www.docker.com/resources/what-container/)
> [What is a container? | Docker Docs](https://docs.docker.com/guides/docker-concepts/the-basics/what-is-a-container/)
> [What is an image? | Docker Docs](https://docs.docker.com/guides/docker-concepts/the-basics/what-is-an-image/)
> [What is a registry? | Docker Docs](https://docs.docker.com/guides/docker-concepts/the-basics/what-is-a-registry/)

#### 3.1.2 Docker installeren

Om docker te installeren voeren we de volgende commando's uit.

Eerst voegen we de docker apt repository toe:
```shell
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Vervolgens kunnen we de docker pakketten installeren:
```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

We gaan ook instellen dat docker en containerd automatisch opstarten:
```shell
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

Ten slotte gaan we ook onze user toevoegen aan de docker groep:
```shell
sudo usermod -aG docker $USER
```

>Sources: 
>[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
>[Linux post-installation steps for Docker Engine](https://docs.docker.com/engine/install/linux-postinstall)

>[!info]  Convenience Script
>Er is ook een convenience script dat je kan gebruiken, maar dat wordt afgeraden buiten development.

### 3.2 Containerd
#### 3.2.1 Wat is containerd?
Containerd is eigenlijk de onderliggende laag onder de docker daemon(`dockerd`). Het is de container runtime waar Docker gebruik van maakt.

Een container runtime maakt de container en zorgt dat het blijft draaien, het beheerd ook de hoeveelheid resources een container gebruikt. Het is een tussenstap tussen de host OS en de container engine, e.g. Docker of Kubernetes.

![[Docker-stack2.png]]
%%![[docker-stack.png]]%%

> Sources:
> [containerd vs. Docker | Docker](https://www.docker.com/blog/containerd-vs-docker/)
> [What are Container Runtimes? Types and Popular Runtime Tools | Wiz](https://www.wiz.io/academy/container-runtimes)

<div class="page-break" style="page-break-before: always;"></div>

#### 3.2.2 Waarom containerd over Docker als CRI
Origineel moest je Docker gebruiken als container runtime voor kubernetes, maar sinds release v1.24 is dit niet meer mogelijk. Ze hebben deze ondersteuning stop gezet ten gunste van container runtimes die hun eigen CRI technologie ondersteunen. Dit zijn momenteel:
- containerd
- cri-o
- docker (`cri-dockerd`)
- mirantis container runtime (MCR)

Dit wilt **niet** zeggen dat je geen Docker images kan gebruiken met kubernetes, aangezien deze  gemaakt worden volgens een standaard (OCI) die deze container runtimes allemaal ondersteunen.

Sinds dat containerd al wordt meegeleverd met Docker heb ik hiervoor gekozen, CRI-O is een goed alternatief aangezien dit speciaal gemaakt is voor kubernetes.  Voor Docker te gebruiken heb je nu een extra applicatie (`cri-dockerd`) nodig. Mirantis Container Runtime (MCR) is ontwikkeld door Mirantis Inc., dit maakt gebruikt van het `cri-dockerd` component sinds dat dit van dezelfde ontwikkelaars is.

> Sources:
> [The Differences between docker and containerd](https://vineetcic.medium.com/the-differences-between-docker-containerd-cri-o-and-runc-a93ae4c9fdac#:~:text=containerd%20is%20a%20high%2Dlevel,processes%20we%20call%20'containers'.)
> [Container Runtimes | Kubernetes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)
> [Updated: Dockershim Removal FAQ | Kubernetes](https://kubernetes.io/blog/2022/02/17/dockershim-faq/)

#### 3.2.3 Containerd instellen

Aangezien containerd een onderdeel is van Docker wordt het automatisch mee geinstalleerd als we ook docker installeren. Maar het heeft wat extra aanpassingen nodig om de systemd driver te gebruiken voor kubernetes.

Eerst maken we een tijdelijke standaard config aan en maken we een backup van de oude config:
```shell
containerd config default > /tmp/config.toml
sudo cp /etc/containerd/config.toml /etc/containerd/config.toml.bak
```

<div class="page-break" style="page-break-before: always;"></div>

Dan veranderen we `SystemdCgroup = false` naar `SystemdCgroup = true`:
```toml {12} title="etc/containerd/config.toml" showLineNumbers{116}
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    BinaryName = ""
    CriuImagePath = ""
    CriuPath = ""
    CriuWorkPath = ""
    IoGid = 0
    IoUid = 0
    NoNewKeyring = false
    NoPivotRoot = false
    Root = ""
    ShimCgroup = ""
    SystemdCgroup = true
```

Vervolgens kopieren we dit bestand en overschrijven we de bestaande config:
```shell
sudo cp /tmp/config.toml /etc/containerd/config.toml
```

We herstarten het containerd process:
```shell
sudo systemctl restart containerd.service
```

Tenslotte verwijderen we de tijdelijke config die we hebben aangemaakt:
```shell
rm /tmp/config.toml
```

>[!tip] Installatie docker & containerd op master node
>Je hoeft eigenlijk geen docker of containerd installeren op de master node als je hier geen pods gaat draaien. Dit kan echter wel handig zijn als je docker images wilt kunnen maken op de master node.

>Sources:
>[Container Runtimes | Kubernetes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)

<div class="page-break" style="page-break-before: always;"></div>

### 3.3 Kubernetes

#### 3.3.1 Wat is kubernetes?
*Kubernetes* is een open-source orkestratieplatform voor het automatiseren van het inzetten, schalen en beheren van software. Het biedt *High-Availability* aan aan de hand van *clusters*.

*Kubernetes clusters* zijn een groepering van systemen of nodes. Dit zijn:
- `Control planes`, het systeem dat de cluster beheerd. Draait de benodigde controle componenten e.g., `kube-apiserver`, `etcd`, `scheduler` en de `controller-manager`.
	-  De `kube-apiserver` configureert de kubernetes objecten e.g., `pods`, `services`, `deployments`, etc. Dit is de belangrijkste en de enige waar je als user mee in aanraking komt, dit gebeurt met het `kubectl` pakket.
	- `etcd` is een "distributed key-value store" (een simpele database) die al de cluster kritieke data bijhoud.
	- De `scheduler` checked of er nieuwe `pods` zijn zonder toegewezen node, en selecteert er een voor de pods om op te draaien.
	- `controller-manager` draait de `controller` processen e.g., `Node controller`, `Job controller`, `EndpointSlice controller` en `ServiceAccount controller`.
- `Worker nodes`, het systeem dat de verschillende werkladingen gaat draaien e.g., docker containers. De worker nodes worden beheerd aan de hand van de `kubelet`  en `kube-proxy` componenten die communiceren met de *Kubernetes API Server* (`kube-api-server`).

Iedere *Kubernetes cluster* bevat minstens 1 `control plane` en 1 `worker node`.  Dit klinkt allemaal heel ingewikkeld maar we gebruiken `kubeadm` voor onze cluster op te zetten, wat dit hele process veel simpeler maakt.

![[cluster-architecture.png]]

Zoals eerder vernoemd zijn er ook een paar *kubernetes* objecten, de belangrijkste voor ons zijn:
- `Pods`
- `Deployments`
- `ReplicaSets`
- `Services`
- `Namespaces`

Een `pod` is een groep van 1 of meer containers met gedeelde opslag en netwerk configuratie. `Pods` maak je zelden zelfstandig aan, dit wordt meestal gedaan aan de hand van een `Deployment` of een `Job`.

In een `deployment` geef je gewenste staat aan van je applicatie zoals, hoeveel replicas, de `image` om te gebruiken, etc. De `deployment` maakt dan een `ReplicaSet` aan om dit te bereiken.

Een `ReplicaSet` zorgt ervoor dat de gewenste hoeveelheid replica `Pods` draaien. Zoals eerder vermeld worden deze meestal aangemaakt door een `deployment`.

In *kubernetes*, is een `Service` de manier waarop we poorten gaan forwarden van onze containers naar ons *host systeem* of *kubernetes*. Dit gebeurt aan de hand van `Services` omdat ip adressen vaak veranderen in een *kubernetes* omgeving. Dit zorgt er dus voor dat je terecht komt bij de juiste `Pods` ongeacht het ip adres hiervan.

<div class="page-break" style="page-break-before: always;"></div>

Er zijn verschillende types van `Services`:
- `ClusterIP`
- `NodePort`
- `LoadBalancer`
- `ExternalName`

Maar de enige die we gaan gebruiken is het type `LoadBalancer`. Dit gebruikt een externe Load Balancer om je `Service` bereikbaar te maken. We gebruiken hier een externe Load Balancer (`metallb`) omdat *kubernetes* geen ingebouwd component heeft voor Load Balancing.

Ten slotte hebben we de `Namespaces`. Hiermee delen we de cluster op in verschillende geïsoleerde stukken waar onze andere objecten in kunnen leven. Dit wordt voornamelijk gebruikt als je meerdere applicaties of teams hebt die elks een unieke `Namespace` krijgen. Zodat er een kleinere kans is op storingen tussen objecten.

Bijna alle *kubernetes* configuratie waaronder ook deze objecten worden aangemaakt en ingesteld met `yaml` tekst bestanden (`manifests`).

> Sources:
> [Overview | Kubernetes](https://kubernetes.io/docs/concepts/overview/)
> [Cluster Architecture | Kubernetes](https://kubernetes.io/docs/concepts/architecture/)
> [Kubernetes Components | Kubernetes](https://kubernetes.io/docs/concepts/overview/components/)
> [What Are Objects Used for in Kubernetes? 11 Types Explained](https://kodekloud.com/blog/kubernetes-objects/)


#### 3.3.2 Installeren kubeadm, kubelet en kubectl

#### 3.3.2.1 Uitzetten SWAP

Eerst gaan we SWAP uitzetten aangezien kubelet het niet ondersteund.
```shell
sudo swapoff -a
```

En we commenten deze lijn uit in ons /etc/fstab bestand.
```ini title="etc/fstab"
#/swap.img      none    swap    sw      0       0
```

> Sources:
> [How do I disable swap?](https://askubuntu.com/questions/214805/how-do-i-disable-swap)

<div class="page-break" style="page-break-before: always;"></div>

#### 3.3.2.2 Installeren pakketten

Als we `kubeadm` gaan gebruiken om onze cluster op te zetten moeten we dit eerst installeren samen met de `kubelet` en `kubectl` pakketten.

We starten hiermee door de kubernetes apt repository toe te voegen:
```shell
# Install dependencies
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# Add the kubernetes GPG key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the repository to apt sources
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Dan installeren we de `kubelet`, `kubeadm` en `kubectl` pakketten en houden we de versies bij aangezien deze best hetzelfde is bij alle 3.
```shell
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Als laatste starten we de kubelet service en zorgen ervoor dat deze ook automatisch opstart.
```shell
sudo systemctl enable --now kubelet
```

>Sources:
>[Installing kubeadm | Kubernetes](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

---

## 4. Configuratie master node

Je kan `kubeadm` gebruiken met commando opties of met een configuratie bestand. Ik gebruik hiervoor een configuratie bestand aangezien dit meer overzichtelijk is.
```shell
sudo vim /etc/kubernetes/kubeadm-config.yaml
```

<div class="page-break" style="page-break-before: always;"></div>

Dit is de inhoud van het configuratie bestand:
```yaml title="etc/kubernetes/kubeadm-config.yaml"
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: "192.168.10.20"
  bindPort: 6443

---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
networking:
  podSubnet: "10.244.0.0/16"
kubernetesVersion: "v1.30.0"
controlPlaneEndpoint: "cluster-endpoint.p.kaliki.eu"
apiServer:
  certSANs:
  - "p.kaliki.eu"

---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: "systemd"

---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```

>[!hint]
>We voegen ook een extra domeinnaam toe aan het certificaat dat de `api-server` gebruikt. Dit is voor later gebruik van Github actions.
>Source: [https://devops.stackexchange.com/questions/9483/how-can-i-add-an-additional-ip-hostname-to-my-kubernetes-certificate](https://devops.stackexchange.com/questions/9483/how-can-i-add-an-additional-ip-hostname-to-my-kubernetes-certificate)

Ik gebruik hier ook een dns naam voor de instelling `controlPlaneEndpoint`. Dit zorgt ervoor dat je ook meerdere control planes zou kunnen hebben. Dit zou dan naar een load-balancer moeten wijzen maar in mijn geval wijst deze dns naam enkel naar mijn control plane. Dit heb ik ingesteld in de router.

![[router-dnsentry.png]]

We zetten dan de `control plane` op met het volgende commando:
```shell
sudo kubeadm init --config /etc/kubernetes/kubeadm-config.yaml
```

Wanneer onze `control plane` opgezet is moeten we kube config hebben voor `kubectl`. Dit doen we door de volgende commando\'s uit te voeren:
```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Ten slotte moeten we ook een `Pod` network add-on installeren. Dit is hoe onze `Pods` een ip adres gaan krijgen en gaan communiceren. Ik gebruik hier `Flannel`.
```shell
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

> Sources:
> [Creating a cluster with kubeadm | Kubernetes](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
> [Virtual IPs and Service Proxies | Kubernetes](https://kubernetes.io/docs/reference/networking/virtual-ips/#proxy-mode-ipvs)
> [kubeadm Configuration (v1beta3) | Kubernetes](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta3/)
> [ssl - How can I add an additional IP / hostname to my Kubernetes certificate? - DevOps Stack Exchange](https://devops.stackexchange.com/questions/9483/how-can-i-add-an-additional-ip-hostname-to-my-kubernetes-certificate)

---

## 5. Configuratie worker nodes

De configuratie aan de `worker node` kant is heel minimaal, gewoon dit commando uitvoeren om het te verbinden aan de cluster:
```shell
kubeadm join cluster-endpoint.p.kaliki.eu:6443 --token hwlqyf.z8kl8mmido742chb --discovery-token-ca-cert-hash sha256:e00aa409c7d788722ad925112f410840677f2d15d41c6c8522236ca4def18a85
```

> Sources:
> [Creating a cluster with kubeadm | Kubernetes](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

---

## 6. Configuratie kubernetes cluster

### 6.1 MetalLB

We starten met `metallb`, onze load balancer, te installeren op de cluster:
```shell
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml
```

Dan moeten we een `IPAddressPool` aanmaken zodat onze loadbalancer ip adressen kan uitdelen aan de nodige `Services`:
```yaml title="metallb_pool_1.yaml"
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: pool-1
  namespace: metallb-system
spec:
  addresses:
  - 192.168.10.240/28
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: pool-1-l2adv
  namespace: metallb-system
spec:
  ipAddressPools:
  - pool-1
```
Ik geef de loadbalancer hier een range van: `192.168.10.240` tot `192.168.10.254`.

We passen dit dan toe:
```shell
kubectl apply -f metallb_pool_1.yaml
```

>Sources:
>[Installation | MetalLB](https://metallb.org/installation/)
>[Configuration | MetalLB](https://metallb.org/configuration/)

<div class="page-break" style="page-break-before: always;"></div>

### 6.2 Cert-manager

*Cert-manager* is een open-source X.509 certificaat controller voor kubernetes. Het kan certificaten van verschillende Issuers aanvragen, en houdt deze ook up-to-date. Het zal dus automatisch opnieuw een aanvraag indienen wanneer er 1 zou vervallen.

#### 6.2.1 Installatie cert-manager
We starten met het manifest te downloaden:
```shell
wget https://github.com/cert-manager/cert-manager/releases/download/v1.14.5/cert-manager.yaml
```

*Cert-manager* heeft ingebouwde ondersteuning voor `GatewayAPI` maar dit staat niet standaard aan, hiervoor moeten we een lijn tekst toevoegen in het manifest bestand:
```yaml {11} title="1-cert-manager.yaml" showLineNumbers{5622}
containers:
        - name: cert-manager-controller
          image: "quay.io/jetstack/cert-manager-controller:v1.14.5"
          imagePullPolicy: IfNotPresent
          args:
          - --v=2
          - --cluster-resource-namespace=$(POD_NAMESPACE)
          - --leader-election-namespace=kube-system
          - --acme-http01-solver-image=quay.io/jetstack/cert-manager-acmesolver:v1.14.5
          - --max-concurrent-challenges=60
          - --feature-gates=ExperimentalGatewayAPISupport=true
```

Dit installeren we dan in onze cluster:
```shell
kubectl apply -f cert-manager.yaml
```

>Source:
>[kubectl apply - cert-manager Documentation](https://cert-manager.io/docs/installation/kubectl/)
>[Annotated Gateway resource - cert-manager Documentation](https://cert-manager.io/docs/usage/gateway/)

<div class="page-break" style="page-break-before: always;"></div>

#### 6.2.2 Aanvragen certificaten
Ik ga cloudflare gebruiken als mijn ACME Issuer, dit is waar ik mijn DNS records ook beheer. Ik ga deze certificaat ook aanvragen met een DNS challenge aangezien ik geen toegang heb aan de poorten nodig voor een HTTP challenge.

Eerst moeten we een Secret in kubernetes aanmaken met onze cloudflare API token in:
```yaml title="2-cf-api-token.yaml"
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-api-token
  namespace: cert-manager
type: Opaque
stringData:
  api-token: <token here>
```

Vervolgens maken we een staging `ClusterIssuer` aan om te testen of alles correct werkt, aangezien Let's Encrypt hun productie versie rate-limit:
```yaml title="3-cf-issuer-staging.yaml"
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: cf-issuer-staging
spec:
  acme:
    email: <email here>
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: cf-staging-key
    solvers:
    - dns01:
        cloudflare:
          email: <cloudflare email here>
          apiTokenSecretRef:
            name: cloudflare-api-token
            key: api-token
```

<div class="page-break" style="page-break-before: always;"></div>

We maken ook ineens de productie versie van deze `ClusterIssuer` aan voor wanneer we hebben getest dat we een werkend certificaat kunnen aanvragen:
```yaml title="4-cf-issuer-prod.yaml"
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: cf-issuer-prod
spec:
  acme:
    email: <email here>
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: cf-prod-key
    solvers:
    - dns01:
        cloudflare:
          email: <cloudflare email here>
          apiTokenSecretRef:
            name: cloudflare-api-token
            key: api-token
```

We gaan deze `ClusterIssuer`'s later vermelden als een annotation in de `Gateway`. Die gaat dan automatisch een certificaat aanvragen.

We vergeten ook niet om deze toe te passen:
```shell
kubectl apply -f 2-cf-api-token.yaml
kubectl apply -f 3-cf-issuer-staging.yaml
kubectl apply -f 4-cf-issuer-prod.yaml
```
Of als deze allemaal in een map zitten kunnen we deze ook simpeler doen:
```shell
kubectl apply -f cert-manager/
```

> Sources:
>[Cloudflare - cert-manager Documentation](https://cert-manager.io/docs/configuration/acme/dns01/cloudflare)
>[TLS - Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/guides/tls/)
>[Getting Started - Let's Encrypt](https://letsencrypt.org/getting-started/)
>[Staging Environment - Let's Encrypt](https://letsencrypt.org/docs/staging-environment/)

<div class="page-break" style="page-break-before: always;"></div>

### 6.3 Istio en GatewayAPI

#### 6.3.1 Installatie Istio en GatewayAPI
![[simple-gateway.png]]
Voor onze applicatie extern bereikbaar te maken via een domeinnaam moeten we 1 van de volgende 2 opties gebruiken:
- `Ingress`
- `GatewayAPI`

`Ingress` is de meer gebruikte van de 2 maar is al een tijdje "Frozen" (krijgt geen updates meer) en is minder flexibel dan `GatewayAPI`. Dit komt omdat `Ingress` alleen bruikbaar is voor HTTP en HTTPS.

`GatewayAPI` is de vervanging van `Ingress`. Het is onder actieve productie en ondersteund HTTP, HTTPS, TCP en UDP, en GRPC. Het is wel ingewikkelder op te zetten dan `Ingress`.

Beide hebben ook een extra `Ingress` of `Gateway` controller nodig, in dit geval gebruik ik `Istio`.

Aangezien `GatewayAPI` de vervanger is van `Ingress` en het beschikt over de functies die ik nodig heb, heb ik hiervoor gekozen over `Ingress`.

We starten door `GatewayAPI` te installeren:
```shell
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/standard-install.yaml
```

Vervolgend downloaden we de laatste versie van `Istio`, in dit geval `1.22.0` en voegen dit toe aan ons PATH zodat we het kunnen uitvoeren:
```shell
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.22.0
export PATH=$PWD/bin:$PATH
```

We installeren het juiste `Istio` profiel voor gebruik met `GatewayAPI`:
```shell
istioctl install -f samples/bookinfo/demo-profile-no-gateways.yaml -y
```

> Source:
> [Istio / Getting Started with Istio and Kubernetes Gateway API](https://istio.io/latest/docs/setup/additional-setup/getting-started/)
> [Introduction - Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/)

#### 6.3.2 Creatie gateway
Voor onze gateway te creëren maken we eerst een nieuwe namespace aan:
```yaml title="1-namespace.yaml"
---
apiVersion: v1
kind: Namespace
metadata:
	name: gateway
```

Hierna kunnen we onze gateway ook aanmaken:
```yaml title="2-gateway.yaml"
---  
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
	name: istio
	namespace: gateway
	annotations:
		# when ready, switch to production cluster issuer
		cert-manager.io/cluster-issuer: cf-issuer-staging
		#cert-manager.io/cluster-issuer: cf-issuer-prod
spec:
	gatewayClassName: istio
	listeners:
	- name: api-https
	  protocol: HTTPS
	  port: 443
	  hostname: p.kaliki.eu
	  tls:
		mode: Terminate
		certificateRefs:
		- kind: Secret
		  group: ""
		  name: gtw-kaliki-eu-cert
	  allowedRoutes:
		namespaces:
			from: All
```

<div class="page-break" style="page-break-before: always;"></div>

We kunnen hier via de annotations aanpassen welke `ClusterIssuer` er gebruikt wordt. Dan passen we deze toe:
```shell
kubectl apply -f gateway/
```

De gateway maakt ook automatisch een `Service` aan van type `LoadBalancer`. Het extern ip adres hiervan bevind zich dus in de range die we eerder hebben opgegeven voor `metallb`. In mijn geval is dit adres: `192.168.10.240`. Deze `Service` bevind zich ook in dezelfde namespace als de gateway.

Dit is ook waar we onze tweede port forwarding regel aanmaken:

![[Port Forwarding 443.png]]
We forwarden dus het HTTPS verkeer naar het ip dat we hebben gekregen van de `LoadBalancer`.

>Sources:
> [Use Kubernetes Gateway API instead of Ingress! (TLS - Cert Manager - Istio - Prometheus)](https://www.youtube.com/watch?v=nJUzGJQR3tM)
> [Simple Gateway - Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/guides/simple-gateway/)

<div class="page-break" style="page-break-before: always;"></div>

---

## 7. Setup website

### 7.1 Dockerfile
We hebben nu de infrastructuur en de manier waarop we onze applicatie gaan bereiken opgezet. Maar we hebben nog geen applicatie om überhaupt te bereiken. Hiervoor gaan we *NGINX* gebruiken. We gaan hiermee onze eigen website hosten.

Dit creëert echter een probleem, de opslag van onze website i.e. onze `index.html`, `style.css` en nodige `scripts`. Hier zijn een paar oplossingen voor, e.g. `volumes`, `configmaps`, `shared folder` of je eigen `docker image` maken. Ik heb gekozen om mijn eigen docker image te maken, aangezien dit de minste "Points of Failure" heeft en meer uitbreidbaar is dan een `configmap`. Een image zelf maken betekent ook dat je de website zou kunnen bewerken en testen in je eigen programmeer omgeving voor dat het in productie gaat.

Om onze eigen docker image te maken moeten we eerst een `Dockerfile` aanmaken:
```Dockerfile title="Dockerfile"
FROM nginx:stable-alpine

COPY content /usr/share/nginx/html
```
Deze dockerfile is heel minimaal en geeft aan dat we de NGINX docker image gebruiken als basis, plus dat we onze bestanden uit de map `content` kopiëren naar de standaard NGINX html folder (`/usr/share/nginx/html`) in onze container.

>[!hint]
>Ik gebruik hier ook de versie: `stable-alpine`. Dit is gebaseerd op de alpine distro. Deze image is zuiniger met opslag dan de normale image.

Om deze image te creëren gaan we het `docker build` commando gebruiken. Dit doen we zo:
```shell
docker build .
```
Dit commando gaat aan de hand van de `Dockerfile` en de bestanden in onze huidige map een image aanmaken.

<div class="page-break" style="page-break-before: always;"></div>

De mappenstructuur ziet er dus zo uit:
```
./
├── content/
│  └── index.html
└── Dockerfile
```


### 7.2 Github Actions

#### 7.2.1 Inleiding
Het grootste probleem dat we momenteel zouden hebben is dat je manueel de image zou moeten updaten in de kubernetes cluster. Dit gaan we oplossen met gebruik van *Github Actions*.

*Github Actions* is een CI/CD tool gecreëerd door Github. Hiermee kunnen we een aantal taken automatiseren wanneer we naar onze Github repository pushen. Dit gebeurd aan de hand van `workflows`, een yaml bestand met instructies in.

#### 7.2.2 Github repository

Hiervoor moeten we eerst een github repository aanmaken:
![[create-repo.png]]

Dan voeren we de volgende stappen uit om deze toe te voegen op ons systeem.
```shell
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/Jqnx/website-project.git
git push -u origin main
```

*Github actions* heeft ook toegang nodig aan de nodige kubernetes manifests. Dus we zullen deze toevoegen in hun eigen map:
```
./
├── content/
│  └── index.html
├── manifests/
│  ├── deployment.yaml
│  ├── httproute.yaml
│  └── service.yaml
└── Dockerfile
```
Onze git repo ziet er dan zo uit.

<div class="page-break" style="page-break-before: always;"></div>

De `Deployment` stelt in hoeveel replicas we willen, welke image we gebruiken en op welke poort dit gaat gebruiken.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: ghcr.io/jqnx/project-website
        ports:
        - name: http
          containerPort: 80
```
>[!tip]
>Het is belangrijk dat we hier onze eigen image ingeven anders gaat het de foute image gebruiken of geen. De kubernetes deploy stap voegt enkel de tag toe.

<div class="page-break" style="page-break-before: always;"></div>

De `HTTPRoute` is een onderdeel van `GatewayAPI`, het maakt een route aan die luistert naar een hostname en pad, stelt de `Gateway` in om te gebruiken en stelt in naar welke `Service` dit verkeer wordt doorverzonden.
```yaml
---
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: nginx-httproute
  namespace: nginx
spec:
  parentRefs:
  - name: istio
    namespace: gateway
  hostnames: 
  - "p.kaliki.eu"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: nginx-svc
      port: 80
```

De `Service` luistert naar verkeer van de `HTTPRoute` en stuurt dit door naar de `Deployment`.
```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: nginx
  name: nginx-svc
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
```

>Sources:
>[GitHub - Jqnx/project-website](https://github.com/Jqnx/project-website)

<div class="page-break" style="page-break-before: always;"></div>

#### 7.2.3.1 Workflow

De workflow is vrij lang maar we zullen dit in een aantal stukken afbreken:
Eerst stellen we de naam in van de workflow.
```yaml
name: ci/cd
```

Hier geven we aan dat deze workflow enkel wordt uitgevoerd wanneer er naar de repo wordt gepushed met een tag die start met: `v`, e.g. `v1.0.0`
```yaml
on:
  workflow_dispatch:
  push:
    tags:
      - "v*"
```

Hier stellen we een paar environmental variables in voor later hergebruik in het document. We gebruiken github's eigen registry voor onze image to hosten.
```yaml
env:
  # Use docker.io for Docker Hub if empty
  REGISTRY: ghcr.io
  # github.repository as <account>/<repo>
  IMAGE_NAME: ${{ github.repository }}
```

#### 7.2.3.2 Build en push docker image

Vervolgens moeten we onze jobs gaan aangeven, dat zijn de taken die github actions gaat uitvoeren. Onze eerste job is `build`. Het maken, testen en pushen van de docker image. We gebruiken ook cosign om onze docker image te ondertekenen met een digest (`hash`). Ik toon enkel de belangrijkste stappen aangezien het anders te lang zou zijn.

<div class="page-break" style="page-break-before: always;"></div>

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      ...
    outputs:
      tags: ${{ steps.meta.outputs.tags }}

    steps:
      - name: Checkout repository
        ...

      - name: Install cosign
        ...

      - name: Set up Docker Buildx
        ...

      - name: Log into registry ${{ env.REGISTRY }}
        ...

      - name: Extract Docker metadata
        id: meta
        uses: docker/metadata-action@v5.5.1
        with:
          images: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=semver,pattern={{version}}

      - name: Build and push Docker image
        id: build-and-push
        uses: docker/build-push-action@v5.3.0
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Sign the published Docker image
        ...
```

#### 7.2.3.3 Deploy naar kubernetes

Onze tweede job is het deployen van deze image naar onze kubernetes cluster:
```yaml
deploy:
    name: Deploy to kubernetes cluster
    needs: build
    runs-on: ubuntu-latest
    permissions:
      ...
    
    steps:
      - name: Install kubectl
        ...

      - name: Set the kubernetes context
        uses: azure/k8s-set-context@v4
        with:
          method: service-account
          k8s-url: https://p.kaliki.eu:25032
          k8s-secret: ${{ secrets.KUBERNETES_SECRET }}

      - name: Checkout repository
        ...

      - name: Deploy manifests
        uses: azure/k8s-deploy@v5
        with:
          namespace: nginx
          manifests: |
            manifests/deployment.yaml
            manifests/service.yaml
            manifests/httproute.yaml
          images: |
            ${{ needs.build.outputs.tags }}
```

>[!tip] Domeinnaam als k8s-url
>Om een domeinnaam hier te kunnen gebruiken moet deze toegevoegd zijn in het certificaat van de kube-apiserver. Dit hebben we normaalgezien eerder al toegevoegd wanneer we de cluster hadden opgezet.

>Sources:
>[https://github.com/marketplace/actions/kubernetes-set-context](https://github.com/marketplace/actions/kubernetes-set-context)  
>[https://github.com/marketplace/actions/deploy-to-kubernetes-cluster](https://github.com/marketplace/actions/deploy-to-kubernetes-cluster)
>[https://nicwortel.nl/blog/2022/continuous-deployment-to-kubernetes-with-github-actions](https://nicwortel.nl/blog/2022/continuous-deployment-to-kubernetes-with-github-actions)

<div class="page-break" style="page-break-before: always;"></div>

#### 7.2.4 Service account
Deze deployment job moet kunnen communiceren met onze `kube-apiserver` om onze manifests te kunnen toepassen. Hiervoor moeten we een `Service account` aanmaken in onze cluster.

We maken hiervoor dus een `Service account` object aan:
```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: github-actions
  namespace: default
```

<div class="page-break" style="page-break-before: always;"></div>

Dan moeten we een `ClusterRole` aanmaken om de permissies in te stellen die dit account gaat krijgen:
```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: continuous-deployment
  namespace: default
rules:
  - apiGroups:
      - ''
      - apps
      - networking.k8s.io
      - gateway.networking.k8s.io
    resources:
      - namespaces
      - deployments
      - replicasets
      - ingresses
      - services
      - secrets
      - httproutes
    verbs:
      - create
      - delete
      - deletecollection
      - get
      - list
      - patch
      - update
      - watch
```

<div class="page-break" style="page-break-before: always;"></div>

Als 3de zorgen we ervoor dat ons `Service account` wordt toegevoegd aan deze rol:
```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: continuous-deployment
  namespace: default
subjects:
  - kind: ServiceAccount
    name: github-actions
    namespace: default
    apiGroup: ""
roleRef:
  kind: ClusterRole
  name: continuous-deployment
  apiGroup: rbac.authorization.k8s.io
```

Als laatste en belangrijkste moeten we een token aanmaken:
```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: github-actions-token
  namespace: default
  annotations:
    kubernetes.io/service-account.name: github-actions
type: kubernetes.io/service-account-token
```

Deze voegen we toe aan onze github repository:
![[repo-secret.png]]

#### 7.2.5 Portforwarding kube-apiserver
De laatste stap is om ervoor te zorgen dat *Github actions* kan communiceren met onze kubernetes cluster. Hiervoor moeten we onze `kube-apiserver` port forwarden. De poort waar dat de `kube-apiserver` standaard op draait is: **6443**.

![[port forwarding k8s api.png]]

>[!tip] Tailscale
>Ik ben te weten gekomen dat `Tailscale` een *kubernetes operator* en een *github actions* tool hebben die zouden toestaan om veilig de api-server te kunnen gebruiken in je workflow.
>>Sources:  [Tailscale on Kubernetes · Tailscale Docs](https://tailscale.com/kb/1185/kubernetes)
>>[Tailscale GitHub Action · Tailscale Docs](https://tailscale.com/kb/1276/tailscale-github-action)

#### 7.2.6 Updaten website
Zoals vermeld in de workflow wordt deze enkel uitgevoerd wanneer er gepushed wordt met een tag die start met: `v`. Dit is handig voor versie controle te behouden, zo kan je makkelijk een eerdere versie terug gebruiken als er iets mis gaat.

Je kan een tag toevoegen aan je commit door dit commando uit te voeren:
```shell
git tag -a v<version> -m "<tagging message>"
```

Een hele procedure om een keer te updaten ziet er dus bijvoorbeeld zo uit:
```shell
git add .
git tag -a v1.0.0 -m "Version 1.0.0"
git push origin tag v1.0.0
```

>Sources:
>[Git - Tagging](https://git-scm.com/book/en/v2/Git-Basics-Tagging)

---
## Troubleshooting
### 1. SWAP
Het eerste probleem dat ik tegenkwam was dat de kubelet niet wou opstarten. Dit kwam omdat SWAP niet uitstond. Er staat in de documentatie dat SWAP uit moet maar ik had hier per ongeluk over gekeken.
Oplossing: SWAP uitzetten of beter documentatie lezen

### 2. Metallb
Het was even uitzoeken welk ip adres ik hiervoor moest ingeven.
Oplossing: meer documentatie gelezen, voorbeelden bekeken en een beetje experimentatie. (Een ip adres op de LAN dat niet in gebruik is.)

### 3. Github actions
Het was de eerste keer dat ik met github actions werkte dus dit is waar ik de meeste problemen had. Het eerste probleem dat ik hier had was: `Error: signing [ghcr.io/jqnx/project-website:main@sha256:ad889b1031281d98b965ad664101a13c852cc2a87c7d0c754a65d6cd673cd2f6]: getting signer: getting key from Fulcio: getting CTFE public keys: updating local metadata and targets: error updating to TUF remote mirror: invalid key`.
Oplossing: `co-sign` pakket was outdated, geupdate

Het tweede probleem dat ik had was dat ik geen tags had.
Oplossing: Annotated git tags gebruiken en pushen ipv de main branch (git tags met een bericht). Veel experimentatie.

Het derde en laatste probleem dat ik had met github actions was dat het de certificaat van mijn cluster niet kon verifiëren. `error validating "/tmp/deployment.yaml": error validating *** failed to download openapi: Get "[https://p.kaliki.eu:25032/openapi/v2?timeout=32s](https://p.kaliki.eu:25032/openapi/v2?timeout=32s)": tls: failed to verify certificate: x509: certificate is valid for cluster-endpoint.encora.kaliki.eu, kubernetes, kubernetes.default, kubernetes.default.svc, kubernetes.default.svc.cluster.local, master1, not p.kaliki.eu; if you choose to ignore these errors, turn validation off with --validate=false`.
Oplossing: `p.kaliki.eu` toevoegen aan het cluster certificaat. Hiervoor heb ik wel de cluster volledig verwijderd en terug gemaakt.

<div class="page-break" style="page-break-before: always;"></div>

---
## Sources
###### Ubiquiti EdgeRouterX
https://dl.ubnt.com/guides/edgemax/EdgeRouter_ER-X_QSG.pdf

###### Wireguard op EdgeOS
https://www.hostifi.com/blog/edgerouter-wireguard-remote-access-vpn
https://www.erianna.com/wireguard-ubiquity-edgeos/
https://github.com/Lochnair/vyatta-wireguard/releases

###### Installatie ESXi
https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.esxi.install.doc/GUID-6FFA928F-7F7D-4B1A-B05C-777279233A77.html

###### Docker
https://docs.docker.com/get-started/overview/
https://www.docker.com/resources/what-container/
https://docs.docker.com/guides/docker-concepts/the-basics/what-is-a-container/
https://docs.docker.com/guides/docker-concepts/the-basics/what-is-an-image/
https://docs.docker.com/guides/docker-concepts/the-basics/what-is-a-registry/
https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
https://docs.docker.com/engine/install/linux-postinstall

###### Containerd
https://vineetcic.medium.com/the-differences-between-docker-containerd-cri-o-and-runc-a93ae4c9fdac#:~:text=containerd%20is%20a%20high%2Dlevel,processes%20we%20call%20'containers'.
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd

###### Uitzetten SWAP
https://askubuntu.com/questions/214805/how-do-i-disable-swap

###### Kubernetes (kubeadm, kubectl en kubelet)
https://kubernetes.io/docs/concepts/overview
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
https://devops.stackexchange.com/questions/9483/how-can-i-add-an-additional-ip-hostname-to-my-kubernetes-certificate

###### MetalLB
https://metallb.org/installation/
https://metallb.org/configuration/

###### Cert manager
https://cert-manager.io/docs/installation/kubectl/
https://cert-manager.io/docs/usage/gateway/
https://cert-manager.io/docs/configuration/acme/dns01/cloudflare
https://gateway-api.sigs.k8s.io/guides/tls/
https://letsencrypt.org/getting-started/
https://letsencrypt.org/docs/staging-environment/

###### Gateway API & Istio
https://www.youtube.com/watch?v=nJUzGJQR3tM
https://istio.io/latest/docs/setup/additional-setup/getting-started/
https://gateway-api.sigs.k8s.io/
https://gateway-api.sigs.k8s.io/guides/simple-gateway/

###### Website
https://dev.to/paschalogu/how-i-deployed-my-website-as-a-container-3fje
https://docs.docker.com/reference/dockerfile/
https://hub.docker.com/_/nginx

###### Github Actions
https://github.com/marketplace/actions/kubernetes-set-context
https://github.com/marketplace/actions/deploy-to-kubernetes-cluster
https://nicwortel.nl/blog/2022/continuous-deployment-to-kubernetes-with-github-actions
https://devops.stackexchange.com/questions/9483/how-can-i-add-an-additional-ip-hostname-to-my-kubernetes-certificate
https://git-scm.com/book/en/v2/Git-Basics-Tagging
https://github.com/Jqnx/project-website
https://tailscale.com/kb/1185/kubernetes
https://tailscale.com/kb/1276/tailscale-github-action

<div class="page-break" style="page-break-before: always;"></div>

---

## Agenda

###### 6/3
- Installatie router
- Installatie VMWare ESXi
- Portforwarding van ESXi

###### 11/3
- Gestart aan netwerkschema
###### 12 /3
- Netwerkschema v1 afgewerkt

###### 13/3
- Wireguard VPN opgezet op router

###### 14/3
- 4 VMs aangemaakt en geinstalleerd

###### 20/3
- Docker geinstalleerd op worker nodes

###### 20/3 - 17/4
- Kubernetes cluster opzetten research

###### 17/4
- Kubernetes cluster opgezet met 1 master node en 1 worker node

###### 24/4
- Uitgetest hoe services en deployments werken
- Gateway-api research
- Gestart met documentatie schrijven

###### 24/4 - 8/5
- Loadbalancing, gateway-api, nginx en cert-manager experimentatie

###### 8/5
- 2 extra nodes opgezet
- Loadbalancing, gateway-api, nginx en cert-manager opgezet
- Nginx webserver met ssl certificaat opgezet

###### 15/5
- Github actions opgezet
- Verder documentatie geschreven

###### 22/5 - 9/6
- Verder documentatie geschreven
- Powerpoint presentatie gemaakt

---

## Besluit
Ik heb door dit project veel meer bijgeleerd dan ik origineel had verwacht. Docker had ik al wat voorkennis van maar ik heb hier redelijk wat meer over bijgeleerd tijdens dat ik de documentatie aan het schrijven was. Kubernetes wou ik al leren dus ik had een klein idee van wat het was, maar het bleek veel ingewikkelder te zijn dan ik dacht, ook al was veel van de complexiteit verminderd door kube-adm te gebruiken. Github actions had ik nog geen ervaring mee voor dit project en heb hier ook veel meer over te weten gekomen, ook al was het gebruiken van github actions niet nodig. Ik heb dit gebruikt puur omdat ik graag dingen automatiseer. Het leek mij nog al veel moeite om iedere keer een nieuwe image te maken, die te pushen, de deployment te bewerken en die dan terug toe te passen. Zeker als je iets actief zou ontwikkelen.