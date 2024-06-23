---
title: Infrastructure & Docker Networks
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-networks
  - omoikane
  - dionysus
  - pnode
  - oracle
  - infrastructure
---

# 1. Docker Networks
Create these networks before continuing with the docker containers.
## Omoikane
```shell
docker network create proxy
docker network create socket
```

## Dionysus
```shell
docker network create proxy
docker network create download
docker network create arr
docker network create socket
```

## Oracle
```shell
docker network create tunnel
docker network create socket
```

## pnode
```shell
docker network create proxy
docker network create socket
```
