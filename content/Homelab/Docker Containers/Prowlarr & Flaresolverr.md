---
title: Prowlarr & Flaresolverr
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - prowlarr
  - flaresolverr
  - arr
  - dionysus
---

# Instructions

---

## Prowlarr

### Docker Compose
```yaml title="containers/prowlarr/docker-compose.yml"
---
services:
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Brussels
    volumes:
      - ~/containers/prowlarr/config:/config
    ports:
      - 9696:9696
    networks:
      - proxy
      - arr
    restart: unless-stopped
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.prowlarr.loadbalancer.server.port=9696
      - traefik.http.routers.prowlarr.entrypoints=https
      - traefik.http.routers.prowlarr.rule=Host(`prowlarr.${DDN}`)
      - traefik.http.routers.prowlarr.middlewares=authentik-basic@file
      # watchtower
      - com.centurylinklabs.watchtower.enable=true

  ...
  
networks:
  arr:
    external: true
  proxy:
    external: true
```

### Sources
- [prowlarr - LinuxServer.io](https://docs.linuxserver.io/images/docker-prowlarr/)
- [Prowlarr | Servarr Wiki](https://wiki.servarr.com/prowlarr)

---

## Flaresolverr

### Docker Compose
```yaml title="containers/prowlarr/docker-compose.yml"
---
  ...
  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    environment:
      - LOG_LEVEL=info
      - LOG_HTML=false
      - CAPTCHA_SOLVER=none
      - TZ=Europe/Brussels
    ports:
      - 8191:8191
    networks:
      - arr
    restart: unless-stopped
    labels:
      - com.centurylinklabs.watchtower.enable=true

networks:
  arr:
    external: true
```

### Setup
Use TRaSH Guides
[How to setup FlareSolverr - TRaSH Guides](https://trash-guides.info/Prowlarr/prowlarr-setup-flaresolverr/)

### Sources
- [GitHub - FlareSolverr/FlareSolverr: Proxy server to bypass Cloudflare protection](https://github.com/FlareSolverr/FlareSolverr)
- [How to setup FlareSolverr - TRaSH Guides](https://trash-guides.info/Prowlarr/prowlarr-setup-flaresolverr/)