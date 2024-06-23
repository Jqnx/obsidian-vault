---
title: Speedtest Tracker
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - speedtest-tracker
  - omoikane
---

# Instructions

## 1. Docker Compose

```yaml title="containers/speedtest-tracker/docker-compose.yml"
---
services:
  speedtest-tracker:
    container_name: speedtest-tracker
    image: lscr.io/linuxserver/speedtest-tracker:latest
    ports:
        - 8511:80
        #- 8512:443
    environment:
        - PUID=1000
        - PGID=1000
        - DB_CONNECTION=sqlite
        - APP_KEY= # Get APP_KEY from https://speedtest-tracker.dev/
        - SPEEDTEST_SCHEDULE=
        - SPEEDTEST_SERVERS=
        - DISPLAY_TIMEZONE=Europe/Brussels
        - APP_URL=https://speedtest.${DDN}
    volumes:
        - ~/containers/speedtest/data:/config
    restart: unless-stopped
    networks:
      - proxy
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.bookstack.loadbalancer.server.port=80
      - traefik.http.routers.bookstack.entrypoints=https
      - traefik.http.routers.bookstack.rule=Host(`speedtest.${DDN}`)
      - traefik.http.routers.bookstack.middlewares=authentik@file
      # watchtower
      - com.centurylinklabs.watchtower.enable=true

networks:
  proxy:
    external: true
```

> Sources:
> [Introduction | Speedtest Tracker](https://docs.speedtest-tracker.dev/)
> [GitHub - alexjustesen/speedtest-tracker](https://github.com/alexjustesen/speedtest-tracker)
> [speedtest-tracker - LinuxServer.io](https://docs.linuxserver.io/images/docker-speedtest-tracker/)