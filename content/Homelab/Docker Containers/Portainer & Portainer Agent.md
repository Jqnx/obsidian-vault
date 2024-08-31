---
title: Portainer & Portainer Agent
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - portainer
  - portainer-agent
  - omoikane
  - dionysus
---

# Instructions

---

## Portainer

### Docker Compose
```yaml containers/portainer/docker-compose.yml
```

### Sources
- temp

---

## Portainer Agent

### Docker Compose
```yaml containers/portainer-agent/docker-compose.yml
---
services:
  portainer-agent:
    image: portainer/agent:latest
    container_name: portainer-agent
    restart: always
    ports:
      - 9001:9001
    privileged: true
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    labels:
      - com.centurylinklabs.watchtower.enable=true
```

### Sources
- [Install Portainer Agent on Docker Standalone | Portainer Documentation](https://docs.portainer.io/admin/environments/add/docker/agent)