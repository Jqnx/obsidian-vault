---
title: Traefik-kop (traefik-proxy)
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker compose
  - traefik-kop
  - dionysus
  - pnode
---

# Instructions

## Docker Network
```shell
docker network create proxy
```

## Traefik Configuration
Add `redis` to traefik providers
```yaml title="containers/traefik/data/traefik.yml"
providers:
  redis:
    endpoints:
      - "traefik-redis:6379"
```

## Docker Compose
```yaml title="containers/traefik-kop/docker-compose.yml"
---
services:
  traefik-kop:
    image: ghcr.io/jittering/traefik-kop:latest
    container_name: traefik-kop
    restart: unless-stopped
    networks:
      - proxy
      - socket
    environment:
      - BIND_IP=x.x.x.x # IP of system running this container
      - REDIS_ADDR=x.x.x.x:6379 # IP of system running main traefik instance
      - DOCKER_HOST=tcp://socket-proxy:2375

networks:
  proxy:
    external: true
  socket:
    external: true
```

> Sources:
> [GitHub - jittering/traefik-kop: A dynamic docker-\>redis-\>traefik discovery agent](https://github.com/jittering/traefik-kop)