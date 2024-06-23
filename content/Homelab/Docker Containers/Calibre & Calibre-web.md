---
title: Calibre & Calibre-web
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - calibre
  - calibre-web
  - dionysus
---

# Instructions

## Docker Compose
```yaml title="containers/calibre/docker-compose.yml"
---
services:
  calibre:
    image: lscr.io/linuxserver/calibre:latest
    container_name: calibre
    security_opt:
      - seccomp:unconfined
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Brussels
    volumes:
      - ~/containers/calibre/config:/config
      - /mnt/<library1>/media/books:/data/media/books
    ports:
      - 8181:8181
      - 8081:8081
    restart: unless-stopped
    labels:
      - com.centurylinklabs.watchtower.enable=true

  calibre-web:
    image: lscr.io/linuxserver/calibre-web:latest
    container_name: calibre-web
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Brussels
      - DOCKER_MODS=linuxserver/mods:universal-calibre #optional
      - OAUTHLIB_RELAX_TOKEN_SCOPE=1 #optional
    volumes:
      - ~/containers/calibre/calibre-web:/config
      - /mnt/<library1>/media/books:/books
    ports:
      - 8083:8083
    networks:
      - proxy
    restart: unless-stopped
    depends_on:
      - calibre
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.calibre.loadbalancer.server.port=8083
      - traefik.http.routers.calibre.entrypoints=https
      - traefik.http.routers.calibre.rule=Host(`calibre.${DDN}`)
      - traefik.http.routers.calibre.middlewares=authentik@file
      # watchtower
      - com.centurylinklabs.watchtower.enable=true

networks:
  proxy:
    external: true
```

> Sources:
> [calibre - LinuxServer.io](https://docs.linuxserver.io/images/docker-calibre/)
> [calibre-web - LinuxServer.io](https://docs.linuxserver.io/images/docker-calibre-web/)