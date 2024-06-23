---
title: LANraragi
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - lanraragi
  - dionysus
---

# Instructions

## Docker Compose

```yaml
---
services:
  lanraragi:
    image: difegue/lanraragi:latest
    container_name: lanraragi
    ports:
      - 3000:3000
    networks:
      - proxy
    environment:
    #  - LRR_UID=1000
    #  - LRR_GID=1000
       - LRR_AUTOFIX_PERMISSIONS=0
    volumes:
      - /mnt/doujins/finished/actual-chapters:/home/koyomi/lanraragi/content
      - lrr-db:/home/koyomi/lanraragi/database
    restart: unless-stopped
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.lanraragi.loadbalancer.server.port=3000
      - traefik.http.routers.lanraragi.entrypoints=https
      - traefik.http.routers.lanraragi.rule=Host(`lanraragi.${DDN}`)
      #- traefik.http.routers.lanraragi.middlewares=authentik@file
      # watchtower
      - com.centurylinklabs.watchtower.enable=true

volumes:
  lrr-db:

networks:
  proxy:
    external: true
```

> Sources:
> [GitHub - Difegue/LANraragi: Web application for archival and reading of manga/doujinshi. Lightweight and Docker-ready for NAS/servers.](https://github.com/Difegue/LANraragi)
> [LANraragi Documentation | LANraragi](https://sugoi.gitbook.io/lanraragi/v/dev)