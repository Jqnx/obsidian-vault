---
title: Traefik-kop (traefik-proxy)
draft: false
tags:
  - homelab
  - linux
  - docker
  - docker/compose
  - docker/networks
  - traefik-kop
  - dionysus
  - pnode
  - aphrodite
date modified: 2024-09-30T00:34:10+02:00
date created: 2024-09-22T17:42:44+02:00
---

# Introduction
A dynamic docker->redis->traefik discovery agent.

Solves the problem of running a non-Swarm/Kubernetes multi-host cluster with a single public-facing traefik instance.

```
                        +---------------------+          +---------------------+
                        |                     |          |                     |
+---------+     :443    |  +---------+        |   :8088  |  +------------+     |
|   WAN   |--------------->| traefik |<-------------------->| svc-nginx  |     |
+---------+             |  +---------+        |          |  +------------+     |
                        |       |             |          |                     |
                        |  +---------+        |          |  +-------------+    |
                        |  |  redis  |<-------------------->| traefik-kop |    |
                        |  +---------+        |          |  +-------------+    |
                        |             docker1 |          |             docker2 |
                        +---------------------+          +---------------------+
```

`traefik-kop` solves this problem by using the same `traefik` docker-provider logic. It reads the container labels from the local docker node and publishes them to a given `redis` instance. Simply configure your `traefik` node with a `redis` provider and point it to the same instance, as in the diagram above.


---

# Instructions

## Pre-requisites
- [[Docker Socket Proxy]]
- [[Traefik]]

## Traefik Configuration

Add `redis` to traefik's docker-compose file:
```yaml title="traefik/docker-compose.yml"
services:
	...
	redis:
	  image: docker.io/library/redis:alpine
	  command: --save 60 1 --loglevel warning
	  restart: unless-stopped
	  container_name: traefik-redis
	  networks:
	    - proxy
	  ports:
	    - 6379:6379
	  volumes:
	    - ~/containers/traefik/data/redis/:/data
...
```

Apply changes to the stack
```shell
docker compose up -d
```

Add `redis` to traefik providers
```yaml title="containers/traefik/data/traefik.yml"
...
providers:
  ...
  redis:
    endpoints:
      - "traefik-redis:6379"
...
```

## Docker Compose

```yaml title="containers/traefik-kop/docker-compose.yml"
services:
  traefik-kop:
    image: ghcr.io/jittering/traefik-kop:0.14
    container_name: traefik-kop
    restart: unless-stopped
    networks:
      - proxy
      - socket
    environment:
      - BIND_IP=x.x.x.x # IP of system running this container
      - REDIS_ADDR=x.x.x.x:6379 # IP of system running main traefik instance
      - DOCKER_HOST=tcp://socket-proxy:2375

networks:
  proxy:
    external: true
  socket:
    external: true
```

---

## Sources
- [GitHub - jittering/traefik-kop: A dynamic docker-\>redis-\>traefik discovery agent](https://github.com/jittering/traefik-kop)