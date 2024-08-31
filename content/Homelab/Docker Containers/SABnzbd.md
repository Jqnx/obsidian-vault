---
title: SABnzbd
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - sabnzbd
  - dionysus
---

# Instructions

---

## Docker Compose

```yaml
services:
  sabnzbd:
    container_name: sabnzbd
    image: binhex/arch-sabnzbdvpn:latest
    restart: unless-stopped
    ports:
      - 8082:8080
      - 8090:8090
    networks:
      - download
      - proxy
    volumes:
      - ~/containers/sabnzbd/config:/config
      - /mnt/<library1>/usenet:/data/usenet
      - /etc/localtime:/etc/localtime:ro
    environment:
      - VPN_ENABLED=yes
      - VPN_PROV=custom
      - VPN_CLIENT=wireguard
      - STRICT_PORT_FORWARD=yes
      - ENABLE_PRIVOXY=no
      - LAN_NETWORK=192.168.x.0/24
      - NAME_SERVERS=1.1.1.1,1.0.0.1,8.8.8.8,8.8.4.4,9.9.9.9
      - DEBUG=false
      - UMASK=000
      - PUID=1000
      - PGID=1000
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    privileged: true
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.sabnzbd.loadbalancer.server.port=8082
      - traefik.http.routers.sabnzbd.entrypoints=https
      - traefik.http.routers.sabnzbd.rule=Host(`sabnzbd.${DDN}`)
      - traefik.http.routers.sabnzbd.middlewares=authentik@file
      # watchtower
      - com.centurylinklabs.watchtower.enable=true

networks:
  download:
    external: true
  proxy:
    external: true
```

---

## Sources
- [hub.docker.com/r/binhex/arch-sabnzbdvpn/](https://hub.docker.com/r/binhex/arch-sabnzbdvpn/)
- [documentation/docker/guides/vpn.md at master · binhex/documentation · GitHub](https://github.com/binhex/documentation/blob/master/docker/guides/vpn.md)