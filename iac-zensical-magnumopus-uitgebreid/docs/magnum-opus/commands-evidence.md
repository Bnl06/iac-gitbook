---
title: Commands, bewijs en checks
description: Alle belangrijke commands met uitleg voor uitvoeren, testen en bewijsmateriaal.
---

# Commands, bewijs en checks

Deze pagina is bedoeld om je project te kunnen demonstreren en verdedigen.

## 1. Collections installeren

```bash
ansible-galaxy collection install -r requirements.yml # installeert ansible.posix en containers.podman collections
```

Waarom?

- `ansible.posix` bevat `firewalld` en `seboolean`.
- `containers.podman` bevat Podman modules.

## 2. SSH-connectie testen

```bash
ssh -i ~/.ssh/id_ed25519_iac youness@172.16.120.11 # test handmatig of je met SSH op de VM geraakt
```

Als dit faalt, zal Ansible ook falen.

## 3. Ansible ping testen

```bash
ansible rocky1 -m ping # test of Ansible de host kan bereiken via inventory en SSH
```

Verwacht:

```text
rocky1 | SUCCESS => {"ping": "pong"}
```

## 4. Syntax check

```bash
ansible-playbook --syntax-check playbooks/site.yml --ask-vault-pass # controleert YAML/playbook syntax zonder deployment
```

Waarom?

- Snel fouten vinden.
- Veilig, want er wordt niets gewijzigd op de server.

## 5. Eerste run uitvoeren

```bash
ansible-playbook playbooks/site.yml --ask-vault-pass -K | tee docs/evidence/run1.txt # voert deployment uit en bewaart output als bewijs
```

Uitleg:

| Deel | Betekenis |
|---|---|
| `ansible-playbook` | voert een playbook uit |
| `playbooks/site.yml` | hoofdplaybook van je project |
| `--ask-vault-pass` | vraagt wachtwoord om Ansible Vault te openen |
| `-K` | vraagt sudo/become password als nodig |
| `tee docs/evidence/run1.txt` | toont output én schrijft die naar bewijsbestand |

## 6. Tweede run voor idempotentie

```bash
ansible-playbook playbooks/site.yml --ask-vault-pass -K | tee docs/evidence/run2.txt # tweede run bewijst idempotentie
```

In jouw bewijs toont run 2:

```text
rocky1 : ok=26 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

Waarom sterk?

- `failed=0`: deployment is gelukt.
- `changed=0`: tweede run moest niets meer aanpassen.
- Dat is het bewijs van idempotentie.

## 7. HTTPS headers controleren

```bash
curl -I https://vaultwarden-magnumopus.duckdns.org # vraagt alleen HTTP headers op via HTTPS
```

Waar kijk je naar?

| Header/status | Betekenis |
|---|---|
| `HTTP/2 200` of `HTTP/1.1 200` | site antwoordt correct |
| `server: nginx` | verkeer komt via Nginx |
| security headers | Nginx config wordt toegepast |

## 8. Health endpoint controleren

```bash
curl https://vaultwarden-magnumopus.duckdns.org/alive # controleert of Vaultwarden zelf gezond antwoordt
```

Verwacht meestal:

```text
Alive!
```

## 9. Services controleren via Ansible ad-hoc commands

```bash
ansible rocky1 -b --ask-vault-pass -m command -a "systemctl status vaultwarden.service --no-pager -l" # controleert Vaultwarden systemd service
```

```bash
ansible rocky1 -b --ask-vault-pass -m command -a "systemctl status nginx --no-pager -l" # controleert Nginx service
```

```bash
ansible rocky1 -b --ask-vault-pass -m command -a "podman ps" # toont actieve Podman containers
```

```bash
ansible rocky1 -b --ask-vault-pass -m command -a "firewall-cmd --list-all" # toont firewallzone en toegestane services
```

```bash
ansible rocky1 -b --ask-vault-pass -m command -a "grep '^LogDenied=' /etc/firewalld/firewalld.conf" # controleert firewalld denied logging
```

## 10. Certificaatinformatie controleren

```bash
ansible rocky1 -b --ask-vault-pass -m command -a "podman run --rm --network host --security-opt label=disable -v /etc/letsencrypt:/etc/letsencrypt docker.io/infinityofspace/certbot_dns_duckdns:latest certificates" # toont Let's Encrypt certificaatstatus via certbot container
```

## 11. Hosts-file workaround

Als je domein niet naar de lab-VM wijst vanaf je laptop, kun je tijdelijk een hosts entry zetten:

```text
172.16.120.11 vaultwarden-magnumopus.duckdns.org # laat je laptop het domein naar de lab-VM sturen
```

Linux:

```bash
sudo nano /etc/hosts # opent hosts-file met rootrechten
```

Windows:

```text
C:\Windows\System32\drivers\etc\hosts # locatie van hosts-file op Windows
```

## 12. GitHub Actions quality check

```bash
git add . # zet alle wijzigingen klaar voor commit
git commit -m "Update Magnum Opus documentation" # maakt een commit met duidelijke boodschap
git push # pusht naar GitHub en start workflow
```

De workflow doet:

```bash
python -m pip install ansible ansible-lint # installeert Ansible tooling
ansible-galaxy collection install -r requirements.yml # installeert collections
ansible-playbook --syntax-check playbooks/site.yml # controleert syntax
ansible-lint # controleert Ansible best practices
```

## Bewijsstukken die je moet tonen

| Bestand | Bewijst |
|---|---|
| `run1.txt` | eerste volledige deployment |
| `run2.txt` | idempotentie |
| `https-head.txt` | HTTPS werkt |
| `alive.txt` | Vaultwarden health endpoint werkt |
| `firewall.txt` | firewall is beperkt |
| `firewalld-logdenied.txt` | denied logging staat aan |
| `podman-ps.txt` | container draait |

!!! success "Belangrijkste bewijszin"

    Mijn sterkste bewijs is `run2.txt`, omdat die aantoont dat de tweede run `changed=0` heeft. Dat bewijst dat mijn Ansible-code idempotent is.
