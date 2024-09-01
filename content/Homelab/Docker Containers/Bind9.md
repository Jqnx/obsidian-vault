---
title: Bind9
draft: false
tags:
  - homelab
  - linux
  - ubuntu
  - docker
  - docker-compose
  - bind9
  - dns
  - hermes
  - terraform
---

# Introduction
BIND (Berkeley Internet Name Domain) is a complete, highly portable implementation of the Domain Name System (DNS) protocol.

The BIND name server, `named`, can act as an authoritative name server, recursive resolver, DNS forwarder, or all three simultaneously. It implements views for split-horizon DNS, automatic DNSSEC zone signing and key management, catalog zones to facilitate provisioning of zone data throughout a name server constellation, response policy zones (RPZ) to protect clients from malicious data, response rate limiting (RRL) and recursive query limits to reduce distributed denial of service attacks, and many other advanced DNS features. BIND also includes a suite of administrative tools, including the `dig` and `delv` DNS lookup tools, `nsupdate` for dynamic DNS zone updates, `rndc` for remote name server administration, and more.

---

# Instructions

## Pre-requisites
- [[Watchtower]]
- Disabled port 53 (if using ubuntu-server)

## 1. Docker Compose
```yaml title="containers/bind/docker-compose.yml"
---
services:
  bind9:
    image: docker.io/ubuntu/bind9:9.18-22.04_beta
    container_name: bind9
    ports:
      - "53:53/tcp"
      - "53:53/udp"
    environment:
      #- BIND9_USER=root
      - TZ=Europe/Brussels
    networks:
	  - bind
    volumes:
      - /etc/bind/:/etc/bind/
      - /var/cache/bind::/var/cache/bind
      - /var/lib/bind:/var/lib/bind
    restart: unless-stopped
    labels:
      - com.centurylinklabs.watchtower.enable=true

networks:
  bind:
    name: bind
    driver: bridge
    ipam:
      config:
        - subnet: 172.31.0.0/16
```

## 2. Permissions
Give the volumes the following permissions:
```bash
sudo chown -R root:bind /etc/bind/
sudo chmod 775 /etc/bind/
sudo chown -R bind:bind /var/cache/bind
sudo chown -R bind:bind /var/lib/bind
```

In my case that looks like this:
```bash
sudo chown -R 0:101 /etc/bind/
sudo chmod 775 /etc/bind/
sudo chown -R 101:101 /var/cache/bind
sudo chown -R 101:101 /var/lib/bind
```

> [!tip]
> Check the `/etc/passwd` file inside the container for the UID and GID of the root and bind users.
> ``` title="/etc/passwd"
> root:x:0:0:root:/root:/bin/bash
> bind:x:101:101::/var/cache/bind:/usr/sbin/nologin
> ```

## 3. Custom Config

### named.conf.key
First create `tsig-key` using the following command: 
```shell
docker exec bind9 tsig-keygen -a hmac-sha256
```

Then add the output to a new file called `named.conf.key`.

```cpp title="containers/bind/config/named.conf.key"
key "tsig-key" {
        algorithm hmac-sha256;
        secret "random base64 string";
};
```

### named.conf
Main forwarder points to [[AdGuardHome]] instance.

```cpp title="containers/bind/config/named.conf"
include "/etc/bind/named.conf.key";

acl internal {
    192.168.50.0/24; //LAN IP
};

acl docker {
    172.31.0.0/16; //IP of bind docker network
};

options {
    forwarders {
        192.168.50.3; //IP of AdGuardHome or PiHole
        1.1.1.1; // IP of Cloudflare DNS
      };
      allow-query { internal; docker; };
      listen-on-v6 { none; }; // Disable listening on IPv6
      directory "/var/cache/bind";
};

zone "home.<sld>.<tld>" IN {
    type master;
    file "/etc/bind/home-<sld>-<tld>.zone";
    update-policy { grant tsig-key zonesub any;  }; // Allow updating of DNS records using the secret key. Mostly used for terraform.
};
```

### Zone file
Change `sld` and `tld` to your domain, e.g. `example.com`. Email uses `.` instead of `@`.

```dns-zone-file title="containers/bind/config/home-<sld>-<tld>.zone"
$ORIGIN .
$TTL 172800     ; 2 days
home.<sld.tld>          IN SOA  ns1.home.<sld.tld>. example.gmail.com. (
                                2024062601 ; serial
                                43200      ; refresh (12 hours)
                                900        ; retry (15 minutes)
                                1814400    ; expire (3 weeks)
                                7200       ; minimum (2 hours)
                                )
                        NS      ns1.home.<sld.tld>.
$ORIGIN home.<sld.tld>.
$TTL 3600     ; 1 hour
ns1                     A       192.168.50.6
pve1                    A       192.168.50.2
pve2                    A       192.168.50.40
router                  A       192.168.50.1
```

---

## Sources

- [videos/bind9-docker at main · ChristianLempa/videos · GitHub](https://github.com/ChristianLempa/videos/tree/main/bind9-docker)
- [videos/bind9-terraform-tutorial at main · ChristianLempa/videos · GitHub](https://github.com/ChristianLempa/videos/tree/main/bind9-terraform-tutorial)
- [BIND 9 Administrator Reference Manual — BIND 9 9.19.25-dev documentation](https://bind9.readthedocs.io/en/latest/index.html)