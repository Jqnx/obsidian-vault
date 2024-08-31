---
title: Pterodactyl
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - pterodactyl
  - wings
  - dionysus
  - pnode
---

# Instructions

---

## Pterodactyl Panel

### Docker Compose

```yaml
x-common:
  database:
    &db-environment
    # Do not remove the "&db-password" from the end of the line below, it is important
    # for Panel functionality.
    MYSQL_PASSWORD: &db-password "CHANGE_ME"
    MYSQL_ROOT_PASSWORD: "CHANGE_ME"
  panel:
    &panel-environment
    # This URL should be the URL that your reverse proxy routes to the panel server
    APP_URL: "https://panel.${DDN}"
    # A list of valid timezones can be found here: http://php.net/manual/en/timezones.php
    APP_TIMEZONE: "Europe/Brussels"
    APP_SERVICE_AUTHOR: "noreply@example.com"
    TRUSTED_PROXIES: "192.168.50.3/24,173.245.48.0/20,103.21.244.0/22,103.22.200.0/22,103.31.4.0/22,141.101.64.0/18,108.162.192.0/18,190.93.240.0/20,188.114.96.0/20,197.234.240.0/22,198.41.128.0/17,162.158.0.0/15,104.16.0.0/13,104.24.0.0/14,172.64.0.0/13,131.0.72.0/22" # Set this to your proxy IP, currently added are all cloudflare proxies
    # Uncomment the line below and set to a non-empty value if you want to use Let's Encrypt
    # to generate an SSL certificate for the Panel.
    # LE_EMAIL: ""
  mail:
    &mail-environment
    MAIL_FROM: "pterodactyl@${DDN}"
    MAIL_DRIVER: "smtp"
    MAIL_HOST: "smtp.eu.mailgun.org"
    MAIL_PORT: "587"
    MAIL_USERNAME: "pterodactyl@${DDN}"
    MAIL_PASSWORD: "<pterodactyl-mailgun-password>"
    MAIL_ENCRYPTION: "true"

#
# ------------------------------------------------------------------------------------------
# DANGER ZONE BELOW
#
# The remainder of this file likely does not need to be changed. Please only make modifications
# below if you understand what you are doing.
#
services:
  database:
    image: mariadb:10.5
    restart: always
    command: --default-authentication-plugin=mysql_native_password
    volumes:
      - "/srv/pterodactyl/database:/var/lib/mysql"
    environment:
      <<: *db-environment
      MYSQL_DATABASE: "panel"
      MYSQL_USER: "pterodactyl"
    networks:
      - default
  cache:
    image: redis:alpine
    restart: always
    networks:
      - default
  panel:
    image: ghcr.io/pterodactyl/panel:latest
    restart: always
    ports:
      - "80:80"
      - "443:443"
    networks:
      - default
      - proxy
    links:
      - database
      - cache
    volumes:
      - "/srv/pterodactyl/var/:/app/var/"
      - "/srv/pterodactyl/nginx/:/etc/nginx/http.d/"
      - "/srv/pterodactyl/certs/:/etc/letsencrypt/"
      - "/srv/pterodactyl/logs/:/app/storage/logs"
    environment:
      <<: [*panel-environment, *mail-environment]
      DB_PASSWORD: *db-password
      APP_ENV: "production"
      APP_ENVIRONMENT_ONLY: "false"
      CACHE_DRIVER: "redis"
      SESSION_DRIVER: "redis"
      QUEUE_DRIVER: "redis"
      REDIS_HOST: "cache"
      DB_HOST: "database"
      DB_PORT: "3306"
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.pterodactyl.loadbalancer.server.port=80
      - traefik.http.routers.pterodactyl.entrypoints=https
      - traefik.http.routers.pterodactyl.rule=Host(`panel.${DDN}`)
      #- traefik.http.routers.pterodactyl.middlewares=authentik@file,corsALL@file
      # watchtower
      - com.centurylinklabs.watchtower.enable=true
networks:
  default:
    driver: bridge
  proxy:
    external: true
```

### Sources
- [I Built the PERFECT Game Server with Pterodactyl and Docker - YouTube](https://www.youtube.com/watch?v=_ypAmCcIlBE)
- [I Built the PERFECT Game Server with Pterodactyl and Docker | Techno Tim](https://technotim.live/posts/pterodactyl-game-server/)
- [Introduction | Pterodactyl](https://pterodactyl.io/project/introduction.html)
- [GitHub - pelican-eggs/eggs: Service eggs for the pterodactyl panel](https://github.com/pelican-eggs/eggs)

---

## Pterodactyl Wings

### Docker Compose

```yaml
```

#todo 
- [ ] pterodactyl wings documentation

## Sources
- [I Built the PERFECT Game Server with Pterodactyl and Docker - YouTube](https://www.youtube.com/watch?v=_ypAmCcIlBE)
- [I Built the PERFECT Game Server with Pterodactyl and Docker | Techno Tim](https://technotim.live/posts/pterodactyl-game-server/)
- [Introduction | Pterodactyl](https://pterodactyl.io/project/introduction.html)