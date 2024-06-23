---
title: Docker Socket Proxy
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker compose
  - docker networks
  - socket-proxy
  - omoikane
  - dionysus
  - pnode
  - oracle
---

# Instructions

## Docker Network
```shell
docker network create socket
```

## Docker Compose
```yaml title="containers/socket-proxy/docker-compose.yml"
---
services:
  socket-proxy:
    container_name: socket-proxy
    image: lscr.io/linuxserver/socket-proxy:latest
    restart: unless-stopped
    environment:
      - LOG_LEVEL=info # debug,info,notice,warning,err,crit,alert,emerg
      # 0 to revoke access.
      # 1 to grant access.
      ## Granted by Default
      - EVENTS=1
      - PING=1
      - VERSION=1
      ## Revoked by Default
      # Security critical
      - AUTH=0
      - SECRETS=0
      - POST=1 # Watchtower
      # Not always needed
      - BUILD=0
      - COMMIT=0
      - CONFIGS=0
      - CONTAINERS=1 # Traefik, portainer, etc.
      - ALLOW_START=1
      - ALLOW_STOP=1
      - ALLOW_RESTARTS=1
      - DISABLE_IPV6=0
      - DISTRIBUTION=0
      - EXEC=0
      - GRPC=0
      - IMAGES=1 # Portainer
      - INFO=0 # Portainer
      - NETWORKS=0 # Portainer
      - NODES=0
      - PLUGINS=0
      - SERVICES=0 # Portainer
      - SESSION=0
      - SWARM=0
      - SYSTEM=0
      - TASKS=0 # Portainer
      - VOLUMES=0 # Portainer
    ports:
      - 2375:2375
    networks:
      - socket
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock # Mounted as read-only
    read_only: true
    tmpfs:
	  - /run
    logging:
      options:
        max-file: 5
        max-size: 15m


networks:
  socket:
    external: true
```



>Source:
>[GitHub - Tecnativa/docker-socket-proxy: Proxy over your Docker socket to restrict which requests it accepts](https://github.com/Tecnativa/docker-socket-proxy)
>[socket-proxy - LinuxServer.io](https://docs.linuxserver.io/images/docker-socket-proxy/)