---
title: Sonarr
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - sonarr
  - arr
  - dionysus
---

# Instructions

## Docker Compose

```yaml title="containers/jellyfin/docker-compose.yml"
---
services:

  ...

  sonarr:
    image: lscr.io/linuxserver/sonarr:develop
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Brussels
    volumes:
      - ~/containers/jellyfin/sonarr:/config
      - ~/containers/jellyfin/sonarr/extended/services:/custom-services.d
      - ~/containers/jellyfin/sonarr/extended/cont:/custom-cont-init.d
      - /mnt/<library1>:/data
    ports:
      - 8989:8989
    networks:
      - arr
      - download
      - proxy
    restart: unless-stopped
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.sonarr.loadbalancer.server.port=8989
      - traefik.http.routers.sonarr.entrypoints=https
      - traefik.http.routers.sonarr.rule=Host(`sonarr.${DDN}`)
      - traefik.http.routers.sonarr.middlewares=authentik-basic@file
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

## Custom Quality Profiles
Use TRaSH Guides
[Sonarr - TRaSH Guides](https://trash-guides.info/Sonarr/#sonarr)

> Sources:
> [sonarr - LinuxServer.io](https://docs.linuxserver.io/images/docker-sonarr/)
> [Sonarr | Servarr Wiki](https://wiki.servarr.com/sonarr)