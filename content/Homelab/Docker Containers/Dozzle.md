---
title: Dozzle
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - dozzle
  - omoikane
---

# Instructions

## 1. Docker Compose
```yaml title="containers/dozzle/docker-compose.yml"
---
services:
  dozzle:
    container_name: dozzle
    image: amir20/dozzle:latest
    restart: unless-stopped
    networks:
      - proxy
      - socket
    environment:
      - DOZZLE_HOSTNAME=omoikane # Hostname of local dozzle connection
      - DOCKER_HOST=tcp://socket-proxy:2375
      - DOZZLE_REMOTE_HOST=tcp://x.x.x.x:2375|dionysus,tcp://x.x.x.x:2375|pnode2 # IPs of remote connections with labels using | symbol, all connecting via tcp://
    ports:
      - 9999:8080
    labels:
      # Default traefik labels
      - traefik.enable=true
      - traefik.http.services.dozzle.loadbalancer.server.port=8080
      - traefik.http.routers.dozzle-secure.entrypoints=https
      - traefik.http.routers.dozzle-secure.rule=Host(`dozzle.${DDN}`)
      - traefik.http.routers.dozzle-secure.middlewares=authentik@file
      # Watchtower
      - com.centurylinklabs.watchtower.enable=true

networks:
  proxy:
    name: proxy
    external: true
  socket:
    name: socket
    external: true
```


## Supported env variables
|Flag|Env Variable|Default|
|---|---|---|
|`--addr`|`DOZZLE_ADDR`|`:8080`|
|`--base`|`DOZZLE_BASE`|`/`|
|`--hostname`|`DOZZLE_HOSTNAME`|`""`|
|`--level`|`DOZZLE_LEVEL`|`info`|
|`--auth-provider`|`DOZZLE_AUTH_PROVIDER`|`none`|
|`--auth-header-user`|`DOZZLE_AUTH_HEADER_USER`|`Remote-User`|
|`--auth-header-email`|`DOZZLE_AUTH_HEADER_EMAIL`|`Remote-Email`|
|`--auth-header-name`|`DOZZLE_AUTH_HEADER_NAME`|`Remote-Name`|
|`--enable-actions`|`DOZZLE_ENABLE_ACTIONS`|false|
|`--wait-for-docker-seconds`|`DOZZLE_WAIT_FOR_DOCKER_SECONDS`|0|
|`--filter`|`DOZZLE_FILTER`|`""`|
|`--no-analytics`|`DOZZLE_NO_ANALYTICS`|false|
|`--remote-host`|`DOZZLE_REMOTE_HOST`|

> Sources:
> [Home | Dozzle](https://dozzle.dev/)
> [GitHub - amir20/dozzle](https://github.com/amir20/dozzle)