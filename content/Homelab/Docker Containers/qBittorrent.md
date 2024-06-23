---
title: qBittorrent
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - qbittorrent
  - dionysus
---

# Instructions

## Docker Compose

```yaml
services:
  qbittorrent:
    container_name: qbittorrent
    image: ghcr.io/hotio/qbittorrent:latest
    restart: unless-stopped
    ports:
      - 8080:8080
    networks:
      - download
      - proxy
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Brussels
      - VPN_ENABLED=true
      - VPN_CONF=wg0
      - VPN_LAN_NETWORK=192.168.x.0/24
      - VPN_LAN_LEAK_ENABLED=false
      - VPN_EXPOSE_PORTS_ON_LAN=8080/tcp
      - VPN_IP_CHECK_DELAY=5
      - VPN_HEALTHCHECK_ENABLED=true
      - PRIVOXY_ENABLED=false
    volumes:
      - ~/containers/qbittorrent/config:/config
      - /mnt/<library1>/torrents:/data/torrents
    cap_add:
      - NET_ADMIN
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv6.conf.all.disable_ipv6=1
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.qbittorrent.loadbalancer.server.port=8080
      - traefik.http.routers.qbittorrent.entrypoints=https
      - traefik.http.routers.qbittorrent.rule=Host(`qbittorrent.${DDN}`)
      - traefik.http.routers.qbittorrent.middlewares=authentik@file
      # watchtower
      - com.centurylinklabs.watchtower.enable=true

networks:
  download:
    external: true
  proxy:
    external: true
```

> Sources:
> [hotio/qbittorrent - hotio.dev](https://hotio.dev/containers/qbittorrent/)

## Setup
Use TRaSH Guides
[Basic-Setup - TRaSH Guides](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#qbittorrent-basic-setup)