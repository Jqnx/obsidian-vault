# 1. configuration.yaml

requires *File Editor* add-on

```yaml
http:
  #ssl_certificate: /ssl/fullchain.pem
  #ssl_key: /ssl/privkey.pem
  use_x_forwarded_for: true
  trusted_proxies:
    - 172.30.33.0/24
    - 127.0.0.1
```

# 2. DuckDNS Add-on

[Home Assistant Add-on: DuckDNS](https://github.com/home-assistant/addons/blob/master/duckdns/DOCS.md)

![[duck-dns.png]]

```yaml
domains:
  - jan-homeassistant.duckdns.org
token: <duckdns-token>
aliases: []
lets_encrypt:
  accept_terms: true
  algo: secp384r1
  certfile: fullchain.pem
  keyfile: privkey.pem
seconds: 300
```

Verschil tussen duckdns en ovh:
Je hebt een limit van 5 sub-domeinnamen bij duckdns en je hebt geen controle over dns records.
Bij ovh heb je dit wel, maar je moet hiervoor wel voor een domeinnaam betalen.

# 3. NGINX Proxy Add-on

[Home Assistant Add-on: NGINX Home Assistant SSL proxy](https://github.com/home-assistant/addons/blob/master/nginx_proxy/DOCS.md)

```yaml
domain: jan-homeassistant.duckdns.org
hsts: max-age=31536000; includeSubDomains
certfile: fullchain.pem
keyfile: privkey.pem
cloudflare: false
customize:
  active: false
  default: nginx_proxy_default*.conf
  servers: nginx_proxy/*.conf
```

# 4. Port Forwarding

### 1. Geef de home assistant vm een static ip
![[Static DHCP Lease.png]]

### 2. Maak port forwarding regels aan
![[Port forwarding rules.png]]

# 5. Philips Hue

[Philips Hue](https://www.home-assistant.io/integrations/hue)

Voeg de Philips Hue integratie toe en volg erna de stappen in home assistant.

De apparaten worden automatisch aan de dashboard toegevoegd