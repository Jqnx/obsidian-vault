---
title: Jellyfin & Jellyseerr
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - jellyfin
  - jellyseerr
  - dionysus
---

# Instructions

## 1. Jellyfin

### Docker Compose
```yaml title="containers/jellyfin/docker-compose.yml"
---
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Brussels
    volumes:
      - ~/containers/jellyfin/config:/config
      - /mnt/<library1>/media:/data/media
      - /mnt/<library2>/media:/data/media/<library2>
    ports:
      - 8096:8096
    networks:
      - proxy
    restart: unless-stopped
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.jellyfin.loadbalancer.server.port=8096
      - traefik.http.routers.jellyfin.entrypoints=https
      - traefik.http.routers.jellyfin.rule=Host(`jellyfin.${DDN}`)
      - traefik.http.routers.jellyfin.middlewares=authentik@file
      # watchtower
      - com.centurylinklabs.watchtower.enable=true

  ...

networks:
  proxy:
    external: true
```

> Sources:
> [jellyfin - LinuxServer.io](https://docs.linuxserver.io/images/docker-jellyfin/)
> [Jellyfin](https://jellyfin.org/docs/)

### Custom Configuration
#todo 
- [ ] add custom config to documentation

#### Custom CSS
```css
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/fixes.css');
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/jf_font.css');
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/base.css');
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/accentlist.css');
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/rounding.css');
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/smallercast.css');
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/episodelist/episodes_compactlist.css');
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/header/header_transparent.css');
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/login/login_minimalistic.css');
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/fields/fields_noborder.css');
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/cornerindicator/indicator_corner.css');
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/type/colorful.css');
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/titlepage/title_banner-logo.css');
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/progress/floating.css');
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/effects/hoverglow.css');
@import url('https://cdn.jsdelivr.net/gh/CTalvio/Ultrachromic/effects/glassy.css');
@import url('https://ctalvio.github.io/Monochromic/backdrop-hack_style.css');

@Import url("https://cdn.jsdelivr.net/gh/prayag17/Jellyfin-Icons/round.css");

/*Style backdrop*/
.backdropImage {filter: blur(2px) saturate(120%) contrast(120%) brightness(40%);}

/*Login background*/
#loginPage {
  background: url(https://i.imgur.com/wTyPKCo.png) !important;
  background-size: cover !important;
  background-position: center !important;
}

/*Accent and rounding*/
:root {--accent: 175, 122, 197;}
:root {--rounding: 12px;}

/*Custom Logo*/
.adminDrawerLogo img { content: url(https://i.imgur.com/jOad6DO.png) !important; } imgLogoIcon { content: url(https://i.imgur.com/jOad6DO.png) !important; }
.pageTitleWithLogo { background-image: url(https://i.imgur.com/jOad6DO.png) !important; }
```

> Sources:
> [GitHub - CTalvio/Ultrachromic: The final form, the true evolution of the chromic theme saga!](https://github.com/CTalvio/Ultrachromic)

## 2. Jellyseerr

### Docker Compose
```yaml title="containers/jellyfin/docker-compose.yml"
---
services:
  
  ...

  jellyseerr:
    image: fallenbagel/jellyseerr:latest
    container_name: jellyseerr
    environment:
      - LOG_LEVEL=debug
      - TZ=Europe/Brussels
    ports:
      - 5055:5055
    networks:
      - proxy
    volumes:
      - ~/containers/jellyfin/jellyseerr:/app/config
    restart: unless-stopped
    labels:
      # default traefik labels
      - traefik.enable=true
      - traefik.http.services.jellyseerr.loadbalancer.server.port=5055
      - traefik.http.routers.jellyseerr.entrypoints=https
      - traefik.http.routers.jellyseerr.rule=Host(`jellyseerr.${DDN}`)
      - traefik.http.routers.jellyseerr.middlewares=authentik@file
      # watchtower
      - com.centurylinklabs.watchtower.enable=true

  ...

networks:
  proxy:
    external: true
```

> Sources:
> [GitHub - Fallenbagel/jellyseerr: Fork of overseerr for jellyfin support](https://github.com/Fallenbagel/jellyseerr)