---
title: Secrets, TLS en DNS-01
description: Uitleg van Ansible Vault, Vaultwarden admin token, DuckDNS en Let's Encrypt.
---

# Secrets, TLS en DNS-01

Dit is een belangrijk examenonderdeel, want hier toon je security-inzicht.

## Welke secrets heb je?

| Secret | Waarvoor dient het? | Waar staat het? |
|---|---|---|
| `vaultwarden_admin_token` | toegang tot `/admin` van Vaultwarden | `group_vars/all/vault.yml` |
| `duckdns_token` | DNS-records aanpassen bij DuckDNS | `group_vars/all/vault.yml` |

Deze waarden mogen niet plaintext in Git staan.

## Ansible Vault gebruiken

Eerst maak je een vault-bestand op basis van het voorbeeld:

```bash
cp group_vars/all/vault.yml.example group_vars/all/vault.yml # kopieert voorbeeldbestand naar echt vault-bestand
```

Daarna vul je secrets in:

```yaml
--- # start YAML-document
vaultwarden_admin_token: '$argon2id$v=19$m=...' # gehashte admin token voor Vaultwarden adminpaneel
duckdns_token: "JOUW_DUCKDNS_TOKEN" # DuckDNS token om DNS records te beheren
```

Versleutelen:

```bash
ansible-vault encrypt group_vars/all/vault.yml # versleutelt het secrets-bestand
```

Controleren:

```bash
head group_vars/all/vault.yml # toont de eerste regel om te controleren of Vault actief is
```

Verwacht:

```text
$ANSIBLE_VAULT;1.1;AES256
```

## Waarom admin token hashen?

Vaultwarden verwacht geen gewoon wachtwoord voor het adminpaneel, maar een veilige hash. Je genereert die zo:

```bash
podman run --rm -it docker.io/vaultwarden/server:latest /vaultwarden hash --preset owasp # genereert Argon2 admin token hash via Vaultwarden image
```

Met Docker kan ook:

```bash
docker run --rm -it docker.io/vaultwarden/server:latest /vaultwarden hash --preset owasp # zelfde hashgeneratie via Docker
```

Belangrijk:

- Dit token is voor `/admin`.
- Dit is niet het master password van een gewone gebruiker.
- Je zet de hash in Ansible Vault, niet het gewone wachtwoord.

## Waarom DNS-01?

HTTP-01 zou zo werken:

```text
Let's Encrypt -> internet -> poort 80 van jouw server
```

Maar jouw server staat in VMware/lab en is niet publiek bereikbaar. Daarom werkt DNS-01 beter:

```text
Let's Encrypt -> controleert DNS TXT-record bij DuckDNS
```

Jouw VM hoeft dus niet publiek bereikbaar te zijn voor de certificaataanvraag.

## Certbot command uitgelegd

```bash
podman run --rm --network host --security-opt label=disable \
  -v /etc/letsencrypt:/etc/letsencrypt \
  -v /var/log/letsencrypt:/var/log/letsencrypt \
  -v /etc/letsencrypt/credentials/duckdns.ini:/conf/duckdns.ini:ro \
  docker.io/infinityofspace/certbot_dns_duckdns:latest certonly \
  --non-interactive --agree-tos \
  --email jouw-email@example.com \
  --preferred-challenges dns \
  --authenticator dns-duckdns \
  --dns-duckdns-credentials /conf/duckdns.ini \
  --dns-duckdns-propagation-seconds 120 \
  -d vaultwarden-magnumopus.duckdns.org
```

Met uitleg:

```bash
podman run # start een container
--rm # verwijdert de container automatisch na afloop
--network host # container gebruikt het netwerk van de host
--security-opt label=disable # voorkomt SELinux-labelproblemen bij volumes
-v /etc/letsencrypt:/etc/letsencrypt # bewaart certificaten op de host
-v /var/log/letsencrypt:/var/log/letsencrypt # bewaart certbot logs op de host
-v /etc/letsencrypt/credentials/duckdns.ini:/conf/duckdns.ini:ro # mount DuckDNS credentials read-only
certonly # vraag enkel certificaat aan, configureer geen webserver
--non-interactive # geen interactieve vragen
--agree-tos # accepteert Let's Encrypt voorwaarden
--email jouw-email@example.com # account email voor Let's Encrypt
--preferred-challenges dns # gebruik DNS challenge
--authenticator dns-duckdns # gebruik DuckDNS plugin
--dns-duckdns-credentials /conf/duckdns.ini # pad naar credentials in container
--dns-duckdns-propagation-seconds 120 # wacht op DNS propagatie
-d vaultwarden-magnumopus.duckdns.org # domein waarvoor certificaat wordt aangevraagd
```

## Automatische renewal

Certificaten verlopen, dus je gebruikt cron:

```yaml
- name: Configure automated certificate renewal # maakt cronjob voor certificaatvernieuwing
  ansible.builtin.cron: # beheert cron idempotent
    name: Renew Let's Encrypt certificates for Vaultwarden # unieke naam
    user: root # draait als root
    minute: "17" # minuut 17
    hour: "3" # om 03:17
    job: >- # renewal command
      podman run --rm --network host --security-opt label=disable
      -v /etc/letsencrypt:/etc/letsencrypt
      -v /var/log/letsencrypt:/var/log/letsencrypt
      -v /etc/letsencrypt/credentials/duckdns.ini:/conf/duckdns.ini:ro
      {{ certbot_duckdns_image }} renew --quiet
      --deploy-hook 'systemctl reload nginx'
```

### Waarom `deploy-hook`?

Als een certificaat vernieuwd wordt, moet Nginx reloaden om het nieuwe certificaat te gebruiken.

## Mondelinge vragen

### Waarom staat `duckdns_token` in Vault?

> Omdat iemand met dat token mijn DNS-records kan aanpassen. Het is dus een secret en mag niet in plaintext in Git staan.

### Waarom gebruikt je project `no_log: true`?

> Omdat bepaalde taken secrets in variabelen of URL's gebruiken. `no_log: true` voorkomt dat die secrets in Ansible-output of CI-logs verschijnen.

### Waarom is DNS-01 handig in een lab?

> Omdat de VM geen publiek IP hoeft te hebben. Let's Encrypt valideert via DNS, niet via een inkomende HTTP-connectie naar mijn VM.
