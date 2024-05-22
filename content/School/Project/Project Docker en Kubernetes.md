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

#todo
- [ ]  Add first page picture
- [ ] Add table of contents when done with entire project
- [x] [[#2.4 VMs configureren]]

# **Project Docker en Kubernetes**



---
## 1. Router
### 1.1 Opzoeken handleiding
De handleiding is terug te vinden op deze link:
https://dl.ubnt.com/guides/edgemax/EdgeRouter_ER-X_QSG.pdf

### 1.2 Instellen statisch IP
We starten met de router te resetten door de reset knop 10 seconden lang ingedrukt te houden tot de `eth4` LED begint te knipperen. We verbinden dan onze ethernet kabel met de `eth0/PoE In` poort op de router. Volgens de handleiding bevind de router zich standaard in het `192.168.1.x` subnet. We zullen dus onze computer een statisch IP-adres instellen in dit subnet.

![[1 - Router - Static IP Windows.png]]
### 1.3 Setup Wizard
#### 1.3.1 WAN configuratie
We stellen hier het statisch IP-adres in dat we gekregen hebben aan het begin van de module. In dit geval is dit: `192.168.25.168/24`

![[2 - Router - WAN Config.png]]

#### 1.3.2 LAN configuratie
Vervolgens stellen we het subnet in dat we gekregen hebben in de opgave: `192.168.10.0/24` , we zetten de DHCP server ook aan.

![[3 - Router - LAN Config.png]]

#### 1.3.3 Instellen DHCP windows
Als we klaar zijn met de setup wizard moeten we onze computer terug zetten op DHCP.

![[4 - Router - DHCP Windows.png]]

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

### 2.3 VMs aanmaken

We geven onze VM een naam en we stellen de OS opties in.

![[ESXI 1 - Template Name.png]]

Dan selecteren we de correcte opslag.

![[ESXI 2 - datastore.png]]

Vervolgens geven we de hardware van onze VM in. We selecteren ook onze ISO hier.

![[ESXI 3 - hardware.png]]
### 2.4 VMs configureren

Zo ziet onze IP configuratie er uit. Alle is hetzelfde voor alle 4 VMs dat we aanmaken buiten het `Address: 192.168.10.20` veld in de IP Configuratie.

![[Ubuntu Setup 1 - IPv4 Config.png]]

We breiden onze standaard partitie uit van `14.996G` naar `29.996G`. Voor de rest passen we hier niets aan.

![[Ubuntu Setup 2 - Disk Partitioning.png]]

Hier geven we onze gebruikers info in.

![[Ubuntu Setup 3 - User Creation.png]]

Aangezien ik persoonlijk liever SSH gebruik dan het ESXi controle paneel voeg ik hier ook mijn SSH keys toe vanaf github.

![[Ubuntu Setup 4 - SSH Install.png]]

---

## 3. Gemeenschappelijke configuratie

### 3.1 Docker

#### 3.1.1 Wat is docker?

Docker is een open-source platform voor het ontwikkelen, verzenden en draaien van applicaties. Dit gebeurd aan de hand van *containers*, *images* en *registries*. 
- Een *container* is een geïsoleerde omgeving om code uit te testen/applicaties te draaien, i.e. een mini VM of een geïsoleerde applicatie.
- Een *image* is een gestandaardiseerd pakket dat alle bestanden, libraries en configuratie bevat. Het is een soort sjabloon met instructies voor het maken van *containers*.
- Een *registry* is waar je deze images kan bewaren en delen. Je hebt publieke en privé registries.

%%*Containers* zijn vaak kleiner en gebruiken minder resources dan virtuele machines, hierdoor zijn ze een efficiënter alternatief. Door de *image* is de inhoud van een *container* ook altijd hetzelfde waardoor je dit in bijna elke omgeving kan gebruiken e.g., op verschillende OS distributies of bij verschillende cloud providers. Dit heeft er ook voor gezorgd dat applicaties die vroeger uit meerdere onderdelen bestonden nu in verschillende kleinere stukken zijn gedeeld waar dat elk deel in zijn eigen *container* draait, het is dus meer dynamisch en modulair.%%

![[Docker-overview.png]]

Docker als een applicatie maakt gebruik van de Docker client (`docker`) en de Docker daemon (`dockerd`).
- De docker client (`docker`) is de hoofd manier waarop je docker zou gebruiken. Dit biedt de commando's aan die je gebruikt, e.g. `docker run`, `docker build`, `docker ps`, `docker logs`, etc.
- De docker daemon (`dockerd`) is backend waar dat de docker client je commando's naar toe stuurt. Dit is het deel van docker dat alles uitvoert en beheerd, e.g. images, containers, netwerken, en volumes.

![[Docker-architecture.png]]

Docker is een zeer krachtige tool voor programmeurs en wordt voornamelijk gebruikt in continuous integration and continuous delivery (CI/CD) pipelines. Dit heeft als doel om het testen en implementeren van code sneller en automatisch te maken.

Bovendien heeft docker ook ingebouwde netwerk capaciteiten.

>[!tip] Docker Desktop
>Er bestaat ook een Docker Desktop applicatie met een GUI.

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
Containerd is eigenlijk de onderliggende laag onder de docker daemon(`dockerd`). Het is de container runtime waar docker gebruik van maakt en waar kubernetes gebruik van kan maken.

Een container runtime maakt de container en zorgt dat het blijft draaien, het beheerd ook de hoeveelheid resources een container gebruikt. Het is een tussenstap tussen de host OS en de container engine, e.g. docker of kubernetes.

![[Docker-stack2.png]]
%%![[docker-stack.png]]%%

> Sources:
> [containerd vs. Docker | Docker](https://www.docker.com/blog/containerd-vs-docker/)
> [What are Container Runtimes? Types and Popular Runtime Tools | Wiz](https://www.wiz.io/academy/container-runtimes)

#### 3.2.2 Waarom containerd over Docker als CRI
Origineel moest je docker gebruiken voor kubernetes maar sinds de release van de CRI (Container Runtime Interface) API is de ondersteuning voor docker stop gezet en moet je hiervoor een 3de partij hun applicatie (cri-dockerd) gebruiken. De CRI ondersteund van nature containerd en CRI-O. En aangezien dat docker al containerd gebruikt heb ik besloten om containerd te gebruiken ipv CRI-O en cri-dockerd.

> Sources:
> [The Differences between docker and containerd](https://vineetcic.medium.com/the-differences-between-docker-containerd-cri-o-and-runc-a93ae4c9fdac#:~:text=containerd%20is%20a%20high%2Dlevel,processes%20we%20call%20'containers'.)

#### 3.2.3 Containerd instellen

Aangezien containerd een onderdeel is van Docker wordt het automatisch mee geinstalleerd als we ook docker installeren. Maar het heeft wat extra aanpassingen nodig om de systemd driver te gebruiken voor kubernetes.

Eerst maken we een tijdelijke standaard config aan en maken we een backup van de oude config:
```shell
containerd config default > /tmp/config.toml
sudo cp /etc/containerd/config.toml /etc/containerd/config.toml.bak
```

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

>Sources:
>[Container Runtimes | Kubernetes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)

### 3.3 Kubernetes

#### 3.3.1 Wat is kubernetes?

Deze docker containers zijn handig maar je kan ze enkel lokaal beheren, dus als je zoals in een productie omgeving meerdere machines hebt die allemaal docker containers draaien kan dit zeer verwarrend worden. Dit is waar *kubernetes* ons kan helpen: "*Kubernetes* is een open-source orkestratieplatform voor het automatiseren van het inzetten, schalen en beheren van software".

*Kubernetes* werkt met clusters, dus een groepering van systemen of nodes. Clusters worden gebruikt om *high-availability* aan te bieden, dit betekent dat je applicatie/service operationeel kan blijven zelfs als er een of meer nodes uitvallen. Hoe bestendig je systeem is is afhankelijk van de hoeveelheid nodes je in je cluster hebt.

#todo 
- [ ] how kubernetes works
- [ ] how kubernetes uses containers
- [ ] a few kubernetes concepts e.g. pods, service, ingress?, deployments,replicas
- [ ] how kubernetes does loadbalancing



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

#### 3.3.2.2 Installeren pakketten

Als we kubeadm gaan gebruiken om onze cluster op te zetten moeten we dit eerst installeren samen met de kubelet en kubectl pakketten.

We starten hiermee door de kubernetes apt repository toe te voegen:
```shell
# Install dependencies
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# Add the kubernetes GPG key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the repository to apt sources
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Dan installeren we de kubelet, kubeadm en kubectl pakketten en houden we de versies bij aangezien deze best hetzelfde is bij alle 3.
```shell
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Als laatste starten we de kubelet service en zorgen ervoor dat deze ook automatisch opstart.
```shell
sudo systemctl enable --now kubelet
```


## 4. Configuratie master node
- [ ] configure kubeadm

```shell
sudo vim /etc/kubernetes/kubeadm-config.yaml
```

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

```shell
sudo kubeadm init --config /etc/kubernetes/kubeadm-config.yaml
```

```shell
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 5. Configuratie worker nodes
- [ ] join nodes to the cluster

```shell
kubeadm join cluster-endpoint.p.kaliki.eu:6443 --token hwlqyf.z8kl8mmido742chb --discovery-token-ca-cert-hash sha256:e00aa409c7d788722ad925112f410840677f2d15d41c6c8522236ca4def18a85
```

## 6. Configuratie kubernetes cluster

```shell
git clone https://<personal-access-token>@github.com/Jqnx/k8s-manifests-priv
```

- [ ] Metallb

```shell
kubectl apply -f k8s-manifests-priv/metallb
```

- [ ] Istio
```shell
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/standard-install.yaml
```

```shell
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.22.0
export PATH=$PWD/bin:$PATH
```

```shell
istioctl install -f samples/bookinfo/demo-profile-no-gateways.yaml -y
kubectl label namespace default istio-injection=enabled
```


- [ ] cert-manager

## 7. Setup website
- [ ] Dockerfile
- [ ] html files
- [ ] docker build
- [ ] github actions
- [ ] git tags

## 8. Sources
### Ubiquiti EdgeRouterX
[EdgeRouterX Manual](https://dl.ubnt.com/guides/edgemax/EdgeRouter_ER-X_QSG.pdf)

### Wireguard op EdgeOS
https://www.hostifi.com/blog/edgerouter-wireguard-remote-access-vpn
https://www.erianna.com/wireguard-ubiquity-edgeos/

### Installatie ESXi
https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.esxi.install.doc/GUID-6FFA928F-7F7D-4B1A-B05C-777279233A77.html

### Uitzetten SWAP
https://askubuntu.com/questions/214805/how-do-i-disable-swap

### Docker
https://docs.docker.com/get-started/overview/
https://www.docker.com/resources/what-container/
https://docs.docker.com/guides/docker-concepts/the-basics/what-is-a-container/
https://docs.docker.com/guides/docker-concepts/the-basics/what-is-an-image/
https://docs.docker.com/guides/docker-concepts/the-basics/what-is-a-registry/
https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
https://docs.docker.com/engine/install/linux-postinstall

### Containerd
[The Differences between docker and containerd](https://vineetcic.medium.com/the-differences-between-docker-containerd-cri-o-and-runc-a93ae4c9fdac#:~:text=containerd%20is%20a%20high%2Dlevel,processes%20we%20call%20'containers'.)
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd

### Kubernetes (kubeadm, kubectl en kubelet)
[Overview | Kubernetes](https://kubernetes.io/docs/concepts/overview)
[Installing kubeadm | Kubernetes](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

### MetalLB
https://metallb.org/installation/
https://metallb.org/configuration/

### Gateway API & Istio
[Use Kubernetes Gateway API instead of Ingress! (TLS - Cert Manager - Istio - Prometheus)](https://www.youtube.com/watch?v=nJUzGJQR3tM)
https://istio.io/latest/docs/setup/additional-setup/getting-started/
https://gateway-api.sigs.k8s.io/
https://gateway-api.sigs.k8s.io/guides/simple-gateway/

### Cert manager
https://cert-manager.io/docs/installation/kubectl/
https://cert-manager.io/docs/usage/gateway/
https://cert-manager.io/docs/configuration/acme/dns01/cloudflare
https://gateway-api.sigs.k8s.io/guides/tls/
https://letsencrypt.org/getting-started/
https://letsencrypt.org/docs/staging-environment/

### Website
https://dev.to/paschalogu/how-i-deployed-my-website-as-a-container-3fje
https://docs.docker.com/reference/dockerfile/
https://hub.docker.com/_/nginx

### Github Actions
[Kubernetes Set Context · Actions · GitHub Marketplace · GitHub](https://github.com/marketplace/actions/kubernetes-set-context)
[Deploy to Kubernetes cluster · Actions · GitHub Marketplace · GitHub](https://github.com/marketplace/actions/deploy-to-kubernetes-cluster)
[Continuous deployment to Kubernetes with GitHub Actions](https://nicwortel.nl/blog/2022/continuous-deployment-to-kubernetes-with-github-actions)
[ssl - How can I add an additional IP / hostname to my Kubernetes certificate? - DevOps Stack Exchange](https://devops.stackexchange.com/questions/9483/how-can-i-add-an-additional-ip-hostname-to-my-kubernetes-certificate)