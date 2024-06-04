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

Hier geven we onze gebruikers info en hostname in.

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

#### 3.2.2 Waarom containerd over Docker als CRI
Origineel moest je Docker gebruiken als container runtime voor kubernetes, maar sinds release v1.24 is dit niet meer mogelijk. Ze hebben deze ondersteuning stop gezet ten gunste van container runtimes die hun eigen CRI technologie ondersteunen. Dit zijn momenteel:
- containerd
- cri-o
- docker (`cri-dockerd`)
- mirantis container runtime (MCR)

Dit wilt **niet** zeggen dat je geen Docker images kan gebruiken met kubernetes, aangezien deze  gemaakt worden volgens een standaard (OCI) die deze container runtimes allemaal ondersteund.

Sinds dat containerd al wordt meegeleverd met Docker heb ik hiervoor gekozen, CRI-O is een goed alternatief aangezien dit speciaal gemaakt is voor kubernetes.  Voor Docker te gebruiken heb je nu een extra applicatie (`cri-dockerd`) nodig. Mirantis Container Runtime (MCR) is ontwikkeld door Mirantis Inc., dit maakt gebruikt van het `cri-dockerd` component sinds dat dit van dezelfde ontwikkelaars is.

%%Origineel moest je docker gebruiken voor kubernetes maar sinds release v1.24 van de CRI (Container Runtime Interface) API is de ondersteuning voor docker stop gezet en moet je hiervoor een 3de partij hun applicatie (cri-dockerd) gebruiken. De CRI ondersteund van nature containerd en CRI-O. En aangezien dat docker al containerd gebruikt heb ik besloten om containerd te gebruiken ipv CRI-O en cri-dockerd.%%

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

>[!example]-
>```yaml title="nginx-deployment.yaml"
>apiVersion: apps/v1
>kind: Deployment
>metadata:
>  name: nginx-deployment
>  labels:
>   app: nginx
>spec:
> replicas: 3
>selector:
>    matchLabels:
>     app: nginx
>  template:
>    metadata:
>     labels:
>        app: nginx
>    spec:
>     containers:
>      - name: nginx
>        image: nginx:1.14.2
>       ports:
>        - containerPort: 80
>```

Een `ReplicaSet` zorgt ervoor dat de gewenste hoeveelheid replica `Pods` draaien. Zoals eerder vermeld worden deze meestal aangemaakt door een `deployment`.

In *kubernetes*, is een `Service` de manier waarop we poorten gaan forwarden van onze containers naar ons *host systeem* of *kubernetes*. Dit gebeurt aan de hand van `Services` omdat ip adressen vaak veranderen in een *kubernetes* omgeving. Dit zorgt er dus voor dat je terecht komt bij de juiste `Pods` ongeacht het ip adres hiervan.

>[!example]-
>```yaml title="sample-service.yaml"
>apiVersion: v1
>kind: Service
>metadata:
>   name: my-service
>spec:
>   selector:
>     app.kubernetes.io/name: MyApp
>  ports:
>	- protocol: TCP
>	  port: 80
>	  targetPort: 9376
>```

Er zijn verschillende types van `Services`:
- `ClusterIP`
- `NodePort`
- `LoadBalancer`
- `ExternalName`

Maar de enige die we gaan gebruiken is het type `LoadBalancer`. Dit gebruikt een externe Load Balancer om je `Service` bereikbaar te maken. We gebruiken hier een externe Load Balancer (`metallb` of een cloud provider's) omdat *kubernetes* geen ingebouwd component heeft voor Load Balancing.

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
We voegen ook een extra domeinnaam toe aan het certificaat dat de `api-server` gebruikt. Dit is voor later gebruik van github actions.

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

### 6.2 Cert-manager

*Cert-manager* is een open-source X.509 certificaat controller voor kubernetes. Het kan certificaten van verschillende Issuers aanvragen, en houdt deze ook up-to-date. Het zal dus automatisch opnieuw een aanvraag indienen wanneer er 1 zou vervallen.

#### 6.2.1 Installatie cert-manager
We starten met het manifest te downloaden:
```shell
wget https://github.com/cert-manager/cert-manager/releases/download/v1.14.5/cert-manager.yaml
```

*Cert-manager* heeft ingebouwde ondersteuning voor `GatewayAPI` maar dit staat niet standaard aan, hiervoor moeten we een lijk tekst toevoegen in het manifest bestand:
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

#### 6.2.2 Aanvragen certificaten
Ik ga cloudflare gebruiken als mijn ACME Issuer, dit is waar ik mij DNS records ook beheer. Ik ga deze certificaat ook aanvragen met een DNS challenge aangezien ik geen toegang heb aan de poorten nodig voor een HTTP challenge.

Eerst moet ik een Secret in kubernetes aanmaken met mijn cloudflare API token in:
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

Vervolgens ga ik een staging `ClusterIssuer` aanmaken om te testen of alles correct werkt, aangezien Let's Encrypt hun productie versie rate-limit:
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
> [kubectl apply - cert-manager Documentation](https://cert-manager.io/docs/installation/kubectl/)
>[Annotated Gateway resource - cert-manager Documentation](https://cert-manager.io/docs/usage/gateway/)
>[Cloudflare - cert-manager Documentation](https://cert-manager.io/docs/configuration/acme/dns01/cloudflare)
>[TLS - Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/guides/tls/)
>[Getting Started - Let's Encrypt](https://letsencrypt.org/getting-started/)
>[Staging Environment - Let's Encrypt](https://letsencrypt.org/docs/staging-environment/)
 
### 6.3 Istio en GatewayAPI

#### 6.3.1 Installatie Istio en GatewayAPI
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
> [Istio / Getting Started with Istio and Kubernetes Gateway API](https://istio.io/latest/docs/setup/additional-setup/getting-started/)
> [Introduction - Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/)
> [Simple Gateway - Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/guides/simple-gateway/)

---

## 7. Setup website
#todo 
- [ ] html files
- [ ] github actions
- [ ] git tags

### 7.1 Dockerfile
We hebben nu de infrastructuur en de manier waarop we onze applicatie gaan bereiken opgezet. Maar we hebben nog geen applicatie om überhaupt te bereiken. Hiervoor gaan we *NGINX* gebruiken. We gaan hiermee onze eigen website hosten.

Dit creëert echter een probleem, de opslag van onze website i.e. onze `index.html`, `style.css` en nodige `scripts`. Hier zijn een paar oplossingen voor, e.g. `volumes`, `configmaps`, `shared folder` of je eigen `docker image` maken. Ik heb gekozen om mijn eigen docker image te maken, aangezien dit de minste "Points of Failure" heeft en meer uitbreidbaar is dan een `configmap`. Een image zelf maken betekent ook dat je de website zou kunnen bewerken en testen in je eigen programmeer omgeving voor dat het in productie gaat.

Om onze eigen docker image te maken moeten we eerst een `Dockerfile` aanmaken:
```Dockerfile title="Dockerfile"
FROM nginx:stable-alpine

COPY content /usr/share/nginx/html
```
Deze dockerfile is heel minimaal en geeft aan dat we de NGINX docker image gebruiken als basis, plus dat we onze bestanden uit de map `content` kopiëren naar de standaard NGINX html folder (`/usr/share/nginx/html`) in onze container.

#todo 
- [ ] docker build

### 7.2 NGINX files

#todo 
- [ ] github repository
- [ ] manifest files
- [ ] html files

### 7.3 Github Actions

#### 7.3.1 Inleiding
Het grootste probleem dat we momenteel zouden hebben is dat je manueel de image zou moeten updaten in de kubernetes cluster. Dit gaan we oplossen met gebruik van *Github Actions*.

*Github Actions* is een CI/CD tool gecreëerd door Github. Hiermee kunnen we een aantal taken automatiseren wanneer we naar onze Github repository pushen. Dit gebeurd aan de hand van `workflows`, een yaml bestand met instructies in.

Hier is de `workflow` die ik hiervoor gebruik:
```yaml title=".github/workflows/docker-publish.yaml"
name: ci/cd

# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.

on:
  workflow_dispatch:
  push:
    tags:
      - "v*"

env:
  # Use docker.io for Docker Hub if empty
  REGISTRY: ghcr.io
  # github.repository as <account>/<repo>
  IMAGE_NAME: ${{ github.repository }}


jobs:

  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      # This is used to complete the identity challenge
      # with sigstore/fulcio when running outside of PRs.
      id-token: write
    outputs:
      tags: ${{ steps.meta.outputs.tags }}

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      # Install the cosign tool
      # https://github.com/sigstore/cosign-installer
      - name: Install cosign
        uses: sigstore/cosign-installer@v3.5.0
        with:
          cosign-release: 'v2.2.4'

      # Set up BuildKit Docker container builder to be able to build
      # multi-platform images and export cache
      # https://github.com/docker/setup-buildx-action
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3.3.0

      # Login against a Docker registry
      # https://github.com/docker/login-action
      - name: Log into registry ${{ env.REGISTRY }}
        if: github.event_name != 'pull_request'
        uses: docker/login-action@v3.1.0
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      # Extract metadata (tags, labels) for Docker
      # https://github.com/docker/metadata-action
      - name: Extract Docker metadata
        id: meta
        uses: docker/metadata-action@v5.5.1
        with:
          images: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=semver,pattern={{version}}

      # Build and push Docker image with Buildx
      # https://github.com/docker/build-push-action
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

      # Sign the resulting Docker image digest.
      # This will only write to the public Rekor transparency log when the Docker
      # repository is public to avoid leaking data.  If you would like to publish
      # transparency data even for private images, pass --force to cosign below.
      # https://github.com/sigstore/cosign
      - name: Sign the published Docker image
        env:
          # https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions#using-an-intermediate-environment-variable
          TAGS: ${{ steps.meta.outputs.tags }}
          DIGEST: ${{ steps.build-and-push.outputs.digest }}
        # This step uses the identity token to provision an ephemeral certificate
        # against the sigstore community Fulcio instance.
        run: echo "${TAGS}" | xargs -I {} cosign sign --yes {}@${DIGEST}
  
  deploy:
    name: Deploy to kubernetes cluster
    needs: build
    runs-on: ubuntu-latest

    permissions:
      id-token: write
      contents: read
      actions: read
    
    steps:
      - name: Install kubectl
        uses: azure/setup-kubectl@v4
        with:
          version: 'v1.29.4'
        id: install

      - name: Set the kubernetes context
        uses: azure/k8s-set-context@v4
        with:
          method: service-account
          k8s-url: https://p.kaliki.eu:25032
          k8s-secret: ${{ secrets.KUBERNETES_SECRET }}

      - name: Checkout repository
        uses: actions/checkout@v4

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

#### 7.3.2 Eerste instellingen en env variables

Dit is vrij lang maar we zullen dit in een aantal stukken afbreken:
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

#### 7.3.3 Build en push docker image

Vervolgens moeten we onze jobs gaan aangeven, dat zijn de taken die github actions gaat uitvoeren. Onze eerste job is `build`. Het maken, testen en pushen van de docker image. We gebruiken ook cosign om onze docker image te ondertekenen met een digest (`hash`).
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      # This is used to complete the identity challenge
      # with sigstore/fulcio when running outside of PRs.
      id-token: write
    outputs:
      tags: ${{ steps.meta.outputs.tags }}

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      # Install the cosign tool
      # https://github.com/sigstore/cosign-installer
      - name: Install cosign
        uses: sigstore/cosign-installer@v3.5.0
        with:
          cosign-release: 'v2.2.4'

      # Set up BuildKit Docker container builder to be able to build
      # multi-platform images and export cache
      # https://github.com/docker/setup-buildx-action
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3.3.0

      # Login against a Docker registry
      # https://github.com/docker/login-action
      - name: Log into registry ${{ env.REGISTRY }}
        if: github.event_name != 'pull_request'
        uses: docker/login-action@v3.1.0
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      # Extract metadata (tags, labels) for Docker
      # https://github.com/docker/metadata-action
      - name: Extract Docker metadata
        id: meta
        uses: docker/metadata-action@v5.5.1
        with:
          images: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=semver,pattern={{version}}

      # Build and push Docker image with Buildx
      # https://github.com/docker/build-push-action
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

      # Sign the resulting Docker image digest.
      # This will only write to the public Rekor transparency log when the Docker
      # repository is public to avoid leaking data.  If you would like to publish
      # transparency data even for private images, pass --force to cosign below.
      # https://github.com/sigstore/cosign
      - name: Sign the published Docker image
        env:
          # https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions#using-an-intermediate-environment-variable
          TAGS: ${{ steps.meta.outputs.tags }}
          DIGEST: ${{ steps.build-and-push.outputs.digest }}
        # This step uses the identity token to provision an ephemeral certificate
        # against the sigstore community Fulcio instance.
        run: echo "${TAGS}" | xargs -I {} cosign sign --yes {}@${DIGEST}
```

#### 7.3.3 Deploy naar kubernetes

Onze tweede job is het deployen van deze image naar onze kubernetes cluster:
```yaml
deploy:
    name: Deploy to kubernetes cluster
    needs: build
    runs-on: ubuntu-latest

    permissions:
      id-token: write
      contents: read
      actions: read
    
    steps:
      - name: Install kubectl
        uses: azure/setup-kubectl@v4
        with:
          version: 'v1.30.0'
        id: install

      - name: Set the kubernetes context
        uses: azure/k8s-set-context@v4
        with:
          method: service-account
          k8s-url: https://p.kaliki.eu:25032
          k8s-secret: ${{ secrets.KUBERNETES_SECRET }}

      - name: Checkout repository
        uses: actions/checkout@v4

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

#todo 
- [ ] service account
- [ ] port forward kube api server
- [ ] service account secret in github repo
- [ ] manifests

---

## 8. Sources
### Ubiquiti EdgeRouterX
https://dl.ubnt.com/guides/edgemax/EdgeRouter_ER-X_QSG.pdf

### Wireguard op EdgeOS
https://www.hostifi.com/blog/edgerouter-wireguard-remote-access-vpn
https://www.erianna.com/wireguard-ubiquity-edgeos/

### Installatie ESXi
https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.esxi.install.doc/GUID-6FFA928F-7F7D-4B1A-B05C-777279233A77.html

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

### Uitzetten SWAP
https://askubuntu.com/questions/214805/how-do-i-disable-swap

### Kubernetes (kubeadm, kubectl en kubelet)
https://kubernetes.io/docs/concepts/overview
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
https://devops.stackexchange.com/questions/9483/how-can-i-add-an-additional-ip-hostname-to-my-kubernetes-certificate

### MetalLB
https://metallb.org/installation/
https://metallb.org/configuration/

### Cert manager
https://cert-manager.io/docs/installation/kubectl/
https://cert-manager.io/docs/usage/gateway/
https://cert-manager.io/docs/configuration/acme/dns01/cloudflare
https://gateway-api.sigs.k8s.io/guides/tls/
https://letsencrypt.org/getting-started/
https://letsencrypt.org/docs/staging-environment/

### Gateway API & Istio
[Use Kubernetes Gateway API instead of Ingress! (TLS - Cert Manager - Istio - Prometheus)](https://www.youtube.com/watch?v=nJUzGJQR3tM)
https://istio.io/latest/docs/setup/additional-setup/getting-started/
https://gateway-api.sigs.k8s.io/
https://gateway-api.sigs.k8s.io/guides/simple-gateway/

### Website
https://dev.to/paschalogu/how-i-deployed-my-website-as-a-container-3fje
https://docs.docker.com/reference/dockerfile/
https://hub.docker.com/_/nginx

### Github Actions
https://github.com/marketplace/actions/kubernetes-set-context
https://github.com/marketplace/actions/deploy-to-kubernetes-cluster
https://nicwortel.nl/blog/2022/continuous-deployment-to-kubernetes-with-github-actions
https://devops.stackexchange.com/questions/9483/how-can-i-add-an-additional-ip-hostname-to-my-kubernetes-certificate