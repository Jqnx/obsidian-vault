---
title: Jackett
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - arr
  - jackett
  - dionysus
---

# Instructions

## 1. Docker Compose
```yaml title="containers/jackett/docker-compose.yml"
---
services:
  jackett:
    image: lscr.io/linuxserver/jackett:latest
    container_name: jackett
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Brussels
      - AUTO_UPDATE=true
    volumes:
      - ~/containers/jackett/config:/config
      - ~/containers/jackett/downloads:/downloads
    networks:
      - proxy
      - arr
    ports:
      - 9117:9117
    restart: unless-stopped
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.jackett.loadbalancer.server.port=9117
      - traefik.http.routers.jackett.entrypoints=https
      - traefik.http.routers.jackett.rule=Host(`jackett.${DDN}`)
      - traefik.http.routers.jackett.middlewares=authentik-basic@file
      # watchtower
      - com.centurylinklabs.watchtower.enable=true

networks:
  arr:
    external: true
  proxy:
    external: true
```

> Sources:
> [jackett - LinuxServer.io](https://docs.linuxserver.io/images/docker-jackett/)