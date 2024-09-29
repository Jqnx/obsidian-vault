---
title: Traefik
draft: false
tags:
  - homelab
  - linux
  - docker
  - docker/compose
  - docker/networks
  - traefik
  - reverse-proxy
  - omoikane
  - oracle
date modified: 2024-09-30T01:01:09+02:00
date created: 2024-09-22T17:42:44+02:00
---

![[traefik.png]]

Traefik (pronounced _traffic_) is a modern HTTP reverse proxy and load balancer that makes deploying microservices easy. Traefik integrates with your existing infrastructure components ([Docker](https://www.docker.com/), [Swarm mode](https://docs.docker.com/engine/swarm/), [Kubernetes](https://kubernetes.io/), [Consul](https://www.consul.io/), [Etcd](https://coreos.com/etcd/), [Rancher v2](https://rancher.com/), [Amazon ECS](https://aws.amazon.com/ecs), ...) and configures itself automatically and dynamically. Pointing Traefik at your orchestrator should be the _only_ configuration step you need.

---

# Instructions

## Pre-requisites
- Registered domain name and control over DNS records
- [Docker & Docker Compose](https://docs.docker.com/desktop/install/linux/)
- [[Watchtower]]
- [[Docker Socket Proxy]]
- [Cloudflare API Token](https://dash.cloudflare.com/profile/api-tokens)

## Folder Structure
Create following file/folder structure:
```shell
./traefik
├── data
│	├── logs
│	│	├── access.log
│	│	└── traefik.log
│   ├── acme.json
│   ├── config.yml
│   └── traefik.yml
├── cf_api_token
├── .env
└── docker-compose.yml
```

> [!warning]
> `acme.json` requires a permission value of `600`. Make sure to set this using:
> ```shell
> chmod 600 acme.json
>```

## Docker Network
Create our docker network. Our applications will communicate over this network.
```shell
docker network create proxy
```

## .env
> [!info]
> Generating a password is only necessary if using the dashboard. In production use this is generally not recommended. If using the dashboard don't expose it to the internet or look into a proper authentication service e.g. [[Zitadel]], [Authelia](https://www.authelia.com/), [Authentik](https://goauthentik.io/), or [Keycloak](https://www.keycloak.org/). However, it is useful for testing if our staging certificate works when setting up for the first time.

Generate a password using the following command:
```shell
echo $(htpasswd -nB user) | sed -e s/\\$/\\$\\$/g
```

Change the example password with your generated password:
```toml title=".env"
DDN=example.com # change to your domain name
TRAEFIK_DASHBOARD_CREDENTIALS=user:$$2y$$05$$lSaEi.G.aIygyXRdiFpt7OqmUMW9QUG5I1N.j0bXoXxIjxQmoGOWu # swap with generated password
```

## Cloudflare API Token

Generate an API token for your zone [here](https://dash.cloudflare.com/profile/api-tokens).
```toml title="cf_api_token.txt"
PASTE_TOKEN_HERE
```

## Docker Compose

```yaml title="docker-compose.yml"
secrets:
  cf_api_token:
    file: ./cf_api_token.txt

services:
  traefik:
    image: traefik:v3.1
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    networks:
      - proxy
      - socket
    ports:
      - 80:80
      - 443:443
      # - 443:443/tcp # Uncomment if you want HTTP3
      # - 443:443/udp # Uncomment if you want HTTP3
    environment:
      CF_DNS_API_TOKEN_FILE: /run/secrets/cf_api_token # note using _FILE for docker secrets
      TRAEFIK_DASHBOARD_CREDENTIALS: ${TRAEFIK_DASHBOARD_CREDENTIALS}
    secrets:
      - cf_api_token
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./data/traefik.yml:/traefik.yml:ro
      - ./data/acme.json:/acme.json
      - ./data/config.yml:/config.yml:ro
    labels:
      # default traefik labels
      - traefik.enable=true
      # traefik basicauth
      - traefik.http.middlewares.traefik-auth.basicauth.users=${TRAEFIK_DASHBOARD_CREDENTIALS}
      - traefik.http.routers.traefik-secure.middlewares=traefik-auth
      # traefik customheaders
      - traefik.http.middlewares.sslheader.headers.customrequestheaders.X-Forwarded-Proto=https
      # traefik https
       - traefik.http.routers.traefik-secure.entrypoints=https
       - traefik.http.routers.traefik-secure.tls=true
       - traefik.http.routers.traefik-secure.rule=Host(`traefik.${DDN}`)
       - traefik.http.routers.traefik-secure.service=api@internal
      # traefik certificate setup
      - traefik.http.routers.traefik-secure.tls.certresolver=cloudflare
      - traefik.http.routers.traefik-secure.tls.domains[0].main=${DDN}
      - traefik.http.routers.traefik-secure.tls.domains[0].sans=*.${DDN}
      # traefik additional middlewares
      # watchtower
      - com.centurylinklabs.watchtower.enable=true

networks:
  proxy:
    external: true
  socket:
	external: true
```

## traefik.yml

Traefik configuration file.
```yaml title="traefik.yml"
#API
api:
  dashboard: true # set to false after first setup
  debug: true

#LOGS
log:
  filePath: logs/traefik.log
  level: INFO
accessLog:
  filePath: logs/access.log
  bufferingSize: 100
  filters:
    statusCodes:
      - "204-299"
      - "400-499"
      - "500-599"

#ENTRYPOINTS
entryPoints:
  http:
    address: :80
    forwardedHeaders:
      trustedIPs: &trustedIps
      #Local Subnets
        - "127.0.0.1/32"
        - "10.0.0.0/8"
        - "172.16.0.0/12"
        - "192.168.0.0/16"
      #Cloudflare Subnets
        - "173.245.48.0/20"
        - "103.21.244.0/22"
        - "103.22.200.0/22"
        - "103.31.4.0/22"
        - "141.101.64.0/18"
        - "108.162.192.0/18"
        - "190.93.240.0/20"
        - "188.114.96.0/20"
        - "197.234.240.0/22"
        - "198.41.128.0/17"
        - "162.158.0.0/15"
        - "104.16.0.0/13"
        - "104.24.0.0/14"
        - "172.64.0.0/13"
        - "131.0.72.0/22"
      #EOL
    http:
      redirections:
        entryPoint:
          to: https
          scheme: https
  https:
    address: :443
    forwardedHeaders:
      # Reuse list of Cloudflare Trusted IP's above for HTTPS requests
      trustedIPs: *trustedIps
    http:
      tls:
        certResolver: cloudflare
        domains:
          - main: 'vanlook.dev'
          - sans:
              - '*.vanlook.dev'
              - '*.home.vanlook.dev' # added san for internal home use

serversTransport:
  insecureSkipVerify: true

#PROVIDERS
providers:
  docker:
    watch: true
    network: proxy
    #endpoint: "unix:///var/run/docker.sock" # docker socket (default)
    endpoint: "tcp://socket-proxy:2375" # docker socket proxy (recommended)
    exposedByDefault: false
  file:
    filename: /config.yml
    watch: true

#CERT RESOLVERS
certificatesResolvers:
  cloudflare:
    acme:
	  # caServer: https://acme-v02.api.letsencrypt.org/directory # prod (default)
      caServer: https://acme-staging-v02.api.letsencrypt.org/directory # staging
      email: youremail@example.com
      storage: acme.json
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "1.0.0.1:53"
```

## config.yml

Used for setting static routes, services, and middlewares.
```yaml title="config.yml"
http:
 #region routers 
  routers:
    proxmox:
      entryPoints:
        - "https"
      rule: "Host(`proxmox.home.example.com`)"
      middlewares:
        - default-headers
      tls: {}
      service: proxmox
  
#endregion
#region services
  services:
    proxmox:
      loadBalancer:
        servers:
          - url: "https://192.168.50.2:8006"
        passHostHeader: true
#endregion
  middlewares:
    default-headers:
      headers:
        frameDeny: true
        browserXssFilter: true
        contentTypeNosniff: true
        forceSTSHeader: true
        stsIncludeSubdomains: true
        stsPreload: true
        stsSeconds: 15552000
        customFrameOptionsValue: SAMEORIGIN
        customRequestHeaders:
          X-Forwarded-Proto: https

    default-whitelist:
      ipAllowList:
        sourceRange:
        - "10.0.0.0/8"
        - "192.168.0.0/16"
        - "172.16.0.0/12"

    secured:
      chain:
        middlewares:
        - default-whitelist
        - default-headers
```

## Start the docker-compose stack
```shell
docker compose up -d
```

## Switch to production acme endpoint
```yaml title="traefik.yml" {6,7}
...
#CERT RESOLVERS
certificatesResolvers:
  cloudflare:
    acme:
	  caServer: https://acme-v02.api.letsencrypt.org/directory # prod (default)
      # caServer: https://acme-staging-v02.api.letsencrypt.org/directory # staging
      email: youremail@example.com
      storage: acme.json
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "1.0.0.1:53"
```

Also clean out the staging certificates:
```shell
vim data/acme.json
```

> [!tip]
>It sometimes doesn't want to use the cleaned out file. If this happens you just have to recreate the file.
>```shell
>cd data
>rm acme.json
>touch acme.json
>chmod 600 acme.json
>```

Restart the stack
```shell
docker compose restart
```

## Exposing other workloads using traefik

Add following labels to your docker-compose file:
```yaml title="docker-compose-example.yml"
...
	labels:
      # traefik
      - traefik.enable=true
      - traefik.http.services.example.loadbalancer.server.port=80 # this depends on what port the service is running on
      - traefik.http.routers.example.entrypoints=https
      - traefik.http.routers.example.tls=true
      - traefik.http.routers.example.rule=Host(`example.${DDN}`)
...
```

> [!tip]
> Don't forget to change the router names to whatever the service is you are running. Also change the url to whatever you want.
> > [!example]- Option 1
> > ```yaml title="docker-compose-example.yml"
> > ...
> > 	labels:
> > 	   # traefik
> > 	   - traefik.enable=true
> >        - traefik.http.services.nginx.loadbalancer.server.port=80 # this depends on what port the service is running on
> >        - traefik.http.routers.nginx.entrypoints=https
> >        - traefik.http.routers.nginx.tls=true
> >        - traefik.http.routers.nginx.rule=Host(`nginx.${DDN}`)
> > ...
> > ```
> 
> > [!example]- Option 2
> > ```yaml title="docker-compose-example.yml"
> > ...
> > 	labels:
> > 	   # traefik
> > 	   - traefik.enable=true
> >        - traefik.http.services.${TRAEFIK_SERVICE_NAME}.loadbalancer.server.port=${TRAEFIK_SERVICE_PORT}
> >        - traefik.http.routers.${TRAEFIK_SERVICE_NAME}.entrypoints=https
> >        - traefik.http.routers.${TRAEFIK_SERVICE_NAME}.tls=true
> >        - traefik.http.routers.${TRAEFIK_SERVICE_NAME}.rule=Host(`${TRAEFIK_HOST}`)
> > ...
> > ```
> > ```toml title=".env"
> > TRAEFIK_SERVICE_NAME=nginx
> > TRAEFIK_SERVICE_PORT=80
> > TRAEFIK_HOST=nginx.example.com
> > ```

# What's next

- [[Traefik-kop (traefik-proxy)]]

## Sources
- [Traefik Proxy Documentation - Traefik](https://doc.traefik.io/traefik/)
- [Traefik 3 and FREE Wildcard Certificates with Docker | Techno Tim](https://technotim.live/posts/traefik-3-docker-certificates)