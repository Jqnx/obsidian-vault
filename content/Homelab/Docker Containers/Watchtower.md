---
title: Watchtower
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - watchtower
  - omoikane
  - dionysus
  - pnode
  - oracle
  - hermes
  - aphrodite
---

# Introduction
Watchtower is a container-based solution for automating Docker container base image updates.

---

# Instructions

## Pre-requisites
- [[Docker Socket Proxy]]


## 1. Docker Compose

```yaml title="containers/watchtower/docker-compose.yml"
---
services:
  watchtower:
    container_name: watchtower
    image: containrrr/watchtower:latest
    restart: always
    networks:
      - socket
    environment:
      - TZ=Europe/Brussels
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_LABEL_ENABLE=true
      - WATCHTOWER_INCLUDE_RESTARTING=true
      - WATCHTOWER_SCHEDULE=0 0 0 * * *
      - DOCKER_HOST=tcp://socket-proxy:2375
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    labels:
      - com.centurylinklabs.watchtower.enable=true

networks:
  socket:
    external: true
```

> [!tip]
> Enable watchtower monitoring on containers using the following label:
> ```yaml
> com.centurylinklabs.watchtower.enable=true
> ```

---

## Sources
- [Watchtower](https://containrrr.dev/watchtower/)