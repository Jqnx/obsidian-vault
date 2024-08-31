---
title: Radarr
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - radarr
  - arr
  - dionysus
---

# Instructions

---

## Docker Compose

```yaml title="containers/jellyfin/docker-compose.yml"
---
services:

  ...

  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Brussels
    volumes:
      - ~/containers/jellyfin/radarr:/config
      - ~/containers/jellyfin/radarr/extended/services:/custom-services.d
      - ~/containers/jellyfin/radarr/extended/cont:/custom-cont-init.d
      - /mnt/<library1>:/data
    ports:
      - 7878:7878
    networks:
      - arr
      - download
      - proxy
    restart: unless-stopped
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.radarr.loadbalancer.server.port=7878
      - traefik.http.routers.radarr.entrypoints=https
      - traefik.http.routers.radarr.rule=Host(`radarr.${DDN}`)
      - traefik.http.routers.radarr.middlewares=authentik-basic@file
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
[Radarr - TRaSH Guides](https://trash-guides.info/Radarr/#radarr)

---

## Sources
- [radarr - LinuxServer.io](https://docs.linuxserver.io/images/docker-radarr/)
- [Radarr | Servarr Wiki](https://wiki.servarr.com/radarr)
- [Radarr - TRaSH Guides](https://trash-guides.info/Radarr/#radarr)