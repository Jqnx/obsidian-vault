---
title: Lidarr
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - lidarr
  - arr
  - dionysus
---

# Instructions

## Docker Compose

```yaml title="containers/jellyfin/docker-compose.yml"
---
services:

  ...

  lidarr:
    image: lscr.io/linuxserver/lidarr:latest
    container_name: lidarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Brussels
    volumes:
      - ~/containers/jellyfin/lidarr:/config
      - /mnt/<library1>:/data
    ports:
      - 8686:8686
    networks:
      - proxy
      - arr
      - download
    restart: unless-stopped
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.lidarr.loadbalancer.server.port=8686
      - traefik.http.routers.lidarr.entrypoints=https
      - traefik.http.routers.lidarr.rule=Host(`lidarr.${DDN}`)
      - traefik.http.routers.lidarr.middlewares=authentik-basic@file
      # watchtower
      - com.centurylinklabs.watchtower.enable=true

  ...

networks:
  proxy:
    external: true
  arr:
    external: true
  download:
    external: true

```

> Sources:
> [lidarr - LinuxServer.io](https://docs.linuxserver.io/images/docker-lidarr/)
> [Lidarr | Servarr Wiki](https://wiki.servarr.com/lidarr)