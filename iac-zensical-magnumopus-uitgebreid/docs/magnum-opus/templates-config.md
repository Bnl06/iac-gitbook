---
title: Templates en configuratiebestanden
description: Uitleg van de Jinja2 templates voor Vaultwarden, Quadlet, DuckDNS en Nginx.
---

# Templates en configuratiebestanden

Templates zijn belangrijk omdat ze configuratie dynamisch maken. Je schrijft één template en vult waarden in via variabelen.

## DuckDNS credentials template

Bestand:

```text
roles/vaultwarden/templates/duckdns.ini.j2
```

Inhoud:

```jinja
dns_duckdns_token={{ duckdns_token }} # DuckDNS token uit Ansible Vault
```

Waarom belangrijk?

- Certbot heeft dit token nodig om DNS TXT-records te plaatsen.
- Het bestand wordt met `mode: "0600"` geplaatst.
- De taak gebruikt `no_log: true`, zodat het token niet in output verschijnt.

## Vaultwarden env template

Bestand:

```text
roles/vaultwarden/templates/vaultwarden.env.j2
```

Belangrijke regels:

```jinja
DOMAIN={{ vaultwarden_domain_url }} # publieke URL van Vaultwarden
DATA_FOLDER=/data # datafolder binnen de container
SIGNUPS_ALLOWED={{ vaultwarden_signups_allowed | lower }} # registratie aan of uit
INVITATIONS_ALLOWED={{ vaultwarden_invitations_allowed | lower }} # uitnodigingen aan of uit
SENDS_ALLOWED={{ vaultwarden_sends_allowed | lower }} # Bitwarden Send feature
EMERGENCY_ACCESS_ALLOWED={{ vaultwarden_emergency_access_allowed | lower }} # emergency access feature
ENABLE_WEBSOCKET={{ vaultwarden_enable_websocket | lower }} # websocket support
LOG_LEVEL={{ vaultwarden_log_level }} # logniveau zoals info/debug
EXTENDED_LOGGING=true # extra logging aan
ADMIN_TOKEN={{ vaultwarden_admin_token }} # admin token hash uit Vault
```

### Waarom `| lower`?

Ansible gebruikt booleans zoals `True`/`False`, maar veel applicaties verwachten `true`/`false` in lowercase. De filter `| lower` zet dit netjes om.

## Quadlet template

Bestand:

```text
roles/vaultwarden/templates/vaultwarden.container.j2
```

```ini
[Unit]
Description=Vaultwarden container # beschrijving voor systemd
After=network-online.target # wacht tot netwerk online is
Wants=network-online.target # systemd probeert netwerk-online target mee te starten

[Container]
Image={{ vaultwarden_image }} # Vaultwarden image uit variabele
ContainerName=vaultwarden # vaste containernaam
EnvironmentFile={{ vaultwarden_env_dir }}/vaultwarden.env # laad alle app-instellingen uit env-file
Volume={{ vaultwarden_data_dir }}:/data:Z # persistente data en SELinux relabel
PublishPort={{ vaultwarden_http_bind }}:{{ vaultwarden_http_port }}:80 # bind host 127.0.0.1:8080 naar containerpoort 80
Pull=missing # alleen image pullen als die ontbreekt

[Service]
Restart=always # herstart container als die crasht
RestartSec=10 # wacht 10 seconden voor restart
TimeoutStartSec=180 # geef container genoeg tijd om te starten

[Install]
WantedBy=multi-user.target # service start in normale multi-user boot
```

### Waarom Quadlet sterk is

Podman Quadlet laat je containers declaratief beheren via systemd. Je hoeft niet zelf complexe unit files te schrijven.

Mondeling:

> Quadlet vertaalt mijn `.container` bestand naar een systemd service. Daardoor kan ik mijn container behandelen als een gewone Linux-service: enable, start, restart en status werken via systemctl.

## Nginx HTTPS template

Bestand:

```text
roles/vaultwarden/templates/nginx-https.conf.j2
```

```nginx
map $http_upgrade $connection_upgrade { # maakt websocket Connection header dynamisch
    default upgrade; # bij websocket upgrade gebruikt Nginx upgrade
    '' close; # zonder upgrade sluit Nginx normaal
}

server { # HTTP serverblok
    listen 80; # luistert op poort 80
    server_name {{ vaultwarden_domain }}; # domein uit variabele

    location / { # alle HTTP requests
        return 301 https://$host$request_uri; # redirect naar HTTPS
    }
}

server { # HTTPS serverblok
    listen 443 ssl http2; # luistert op HTTPS met HTTP/2
    server_name {{ vaultwarden_domain }}; # domein uit variabele

    ssl_certificate /etc/letsencrypt/live/{{ vaultwarden_domain }}/fullchain.pem; # publiek certificaat
    ssl_certificate_key /etc/letsencrypt/live/{{ vaultwarden_domain }}/privkey.pem; # private key
    ssl_protocols TLSv1.2 TLSv1.3; # alleen moderne TLS versies
    ssl_prefer_server_ciphers off; # moderne client/server cipher keuze

    client_max_body_size 128M; # laat grotere uploads toe

    add_header Strict-Transport-Security "max-age=31536000" always; # forceer HTTPS voor toekomst
    add_header X-Frame-Options SAMEORIGIN always; # klikjacking bescherming
    add_header X-Content-Type-Options nosniff always; # MIME sniffing beperken
    add_header Referrer-Policy no-referrer always; # minder referrer leakage

    location / { # alle app requests
        proxy_pass http://{{ vaultwarden_http_bind }}:{{ vaultwarden_http_port }}; # stuur door naar lokale Vaultwarden backend
        proxy_http_version 1.1; # nodig voor websockets
        proxy_set_header Host $host; # behoud originele host header
        proxy_set_header X-Real-IP $remote_addr; # geef client IP door
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # proxy chain header
        proxy_set_header X-Forwarded-Proto https; # vertel backend dat request HTTPS was
        proxy_set_header Upgrade $http_upgrade; # websocket upgrade header
        proxy_set_header Connection $connection_upgrade; # websocket connection header
    }
}
```

## Templates koppelen aan idempotentie

De Ansible `template` module vergelijkt de huidige inhoud op de target met wat de template zou genereren. Alleen als er verschil is, wordt de taak `changed` en worden handlers getriggerd.

Voorbeeld:

```yaml
notify: Restart vaultwarden # service herstart alleen als de template effectief veranderd is
```

## Wat kan de docent vragen?

### Waarom gebruik je templates en geen copy?

> Omdat templates variabelen ondersteunen. Mijn domein, poort, data directory en secrets komen uit variabelen. Daardoor is mijn configuratie herbruikbaar en niet overal hardcoded.

### Waarom staat Vaultwarden alleen op 127.0.0.1?

> Omdat Nginx de publieke ingang is. De container hoeft niet rechtstreeks bereikbaar te zijn. Dat verkleint de attack surface.

### Waarom security headers?

> Ze versterken de websecurity: HSTS forceert HTTPS, X-Frame-Options helpt tegen clickjacking en nosniff beperkt MIME-type misbruik.
