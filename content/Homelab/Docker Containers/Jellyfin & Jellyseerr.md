---
title: Jellyfin & Jellyseerr
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - jellyfin
  - jellyseerr
  - dionysus
---

# Instructions

## 1. Jellyfin

### Docker Compose
```yaml title="containers/jellyfin/docker-compose.yml"
---
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Brussels
    volumes:
      - ~/containers/jellyfin/config:/config
      - /mnt/<library1>/media:/data/media
      - /mnt/<library2>/media:/data/media/<library2>
    ports:
      - 8096:8096
    networks:
      - proxy
    restart: unless-stopped
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.jellyfin.loadbalancer.server.port=8096
      - traefik.http.routers.jellyfin.entrypoints=https
      - traefik.http.routers.jellyfin.rule=Host(`jellyfin.${DDN}`)
      - traefik.http.routers.jellyfin.middlewares=authentik@file
      # watchtower
      - com.centurylinklabs.watchtower.enable=true

  ...

networks:
  proxy:
    external: true
```

> Sources:
> [jellyfin - LinuxServer.io](https://docs.linuxserver.io/images/docker-jellyfin/)
> [Jellyfin](https://jellyfin.org/docs/)

### Custom Configuration
#todo 
- [ ] add custom config to documentation

## 2. Jellyseerr

### Docker Compose
```yaml title="containers/jellyfin/docker-compose.yml"
---
services:
  
  ...

  jellyseerr:
    image: fallenbagel/jellyseerr:latest
    container_name: jellyseerr
    environment:
      - LOG_LEVEL=debug
      - TZ=Europe/Brussels
    ports:
      - 5055:5055
    networks:
      - proxy
    volumes:
      - ~/containers/jellyfin/jellyseerr:/app/config
    restart: unless-stopped
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.jellyseerr.loadbalancer.server.port=5055
      - traefik.http.routers.jellyseerr.entrypoints=https
      - traefik.http.routers.jellyseerr.rule=Host(`jellyseerr.${DDN}`)
      - traefik.http.routers.jellyseerr.middlewares=authentik@file
      # watchtower
      - com.centurylinklabs.watchtower.enable=true

  ...

networks:
  proxy:
    external: true
```

> Sources:
> [GitHub - Fallenbagel/jellyseerr: Fork of overseerr for jellyfin support](https://github.com/Fallenbagel/jellyseerr)