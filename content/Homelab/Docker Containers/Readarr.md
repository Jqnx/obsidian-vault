---
title: Readarr
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - arr
  - readarr
  - dionysus
---

# Instructions

## 1. Docker Compose
```yaml title="containers/calibre/docker-compose.yml"
---
services:
  ...
  readarr:
    image: lscr.io/linuxserver/readarr:develop
    container_name: readarr
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Brussels
    volumes:
      - ~/containers/calibre/readarr:/config
      - /mnt/<library1>:/data
    ports:
      - 8787:8787
    networks:
      - arr
      - download
      - proxy
    restart: unless-stopped
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.readarr.loadbalancer.server.port=8787
      - traefik.http.routers.readarr.entrypoints=https
      - traefik.http.routers.readarr.rule=Host(`readarr.${DDN}`)
      - traefik.http.routers.readarr.middlewares=authentik-basic@file
      # watchtower
      - com.centurylinklabs.watchtower.enable=true

networks:
  arr:
    external: true
  download:
    external: true
  proxy:
    external: true
```

> Sources:
> [readarr - LinuxServer.io](https://docs.linuxserver.io/images/docker-readarr/)
> [Readarr | Servarr Wiki](https://wiki.servarr.com/readarr)