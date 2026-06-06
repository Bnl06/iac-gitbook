---
title: Troubleshooting Magnum Opus
description: Veelvoorkomende fouten, oorzaken en oplossingen voor het Vaultwarden Magnum Opus-project.
---

# Troubleshooting Magnum Opus

Deze pagina helpt je als de docent vraagt: “Wat als het fout loopt?”

## Fout: SSH werkt niet

Test:

```bash
ssh -i ~/.ssh/id_ed25519_iac youness@172.16.120.11 # test SSH-toegang naar de target VM
```

Mogelijke oorzaken:

| Oorzaak | Oplossing |
|---|---|
| VM staat uit | VM starten |
| IP klopt niet | `inventories/lab/hosts.yml` aanpassen |
| SSH key fout | juiste private key gebruiken |
| user heeft geen toegang | user `youness` controleren |

## Fout: Ansible ping faalt

```bash
ansible rocky1 -m ping -vvv # ping met extra debugoutput
```

Let op:

- Inventory wordt correct geladen?
- `ansible_user` klopt?
- Private key pad klopt?
- SSH werkt handmatig?

## Fout: Vault password ontbreekt

Als je deze fout ziet:

```text
Attempting to decrypt but no vault secrets found
```

Gebruik:

```bash
ansible-playbook playbooks/site.yml --ask-vault-pass # vraagt het vault-wachtwoord interactief
```

## Fout: `firewall_allowed_services is undefined`

Oorzaak: variabelenbestand werd niet geladen.

Controleer in `playbooks/site.yml`:

```yaml
vars_files:
  - ../group_vars/all/main.yml # gewone variabelen laden
  - ../group_vars/all/vault.yml # geheime variabelen laden
```

## Fout: Certbot faalt bij DNS-01

Mogelijke oorzaken:

| Oorzaak | Oplossing |
|---|---|
| DuckDNS token fout | token in Vault controleren |
| Subdomain fout | `duckdns_subdomain` controleren |
| DNS propagatie te traag | `duckdns_propagation_seconds` verhogen |
| Geen internet vanaf VM | netwerk/DNS op VM controleren |

Controleer DuckDNS update:

```bash
curl "https://www.duckdns.org/update?domains=vaultwarden-magnumopus&token=TOKEN&ip=172.16.120.11" # test DuckDNS update, gebruik dit niet in publieke logs
```

!!! warning "Let op"

    Zet je echte token nooit in screenshots of publieke logs.

## Fout: Nginx start niet

Test configuratie:

```bash
sudo nginx -t # controleert syntax van Nginx config
```

Via Ansible:

```bash
ansible rocky1 -b -m command -a "nginx -t" # test Nginx config via Ansible
```

Controleer service:

```bash
ansible rocky1 -b -m command -a "systemctl status nginx --no-pager -l" # toont Nginx status
```

## Fout: Vaultwarden container draait niet

Controleer systemd:

```bash
ansible rocky1 -b -m command -a "systemctl status vaultwarden.service --no-pager -l" # toont service status
```

Controleer containers:

```bash
ansible rocky1 -b -m command -a "podman ps -a" # toont alle containers inclusief gestopte
```

Controleer logs:

```bash
ansible rocky1 -b -m command -a "journalctl -u vaultwarden.service -n 80 --no-pager" # toont laatste 80 logregels
```

## Fout: `/alive` werkt lokaal niet

Test op target:

```bash
curl http://127.0.0.1:8080/alive # test Vaultwarden backend zonder Nginx
```

Als dit faalt, ligt het probleem bij Vaultwarden/container. Als dit werkt maar HTTPS faalt, ligt het probleem waarschijnlijk bij Nginx, firewall, DNS of certificaat.

## Fout: HTTPS werkt niet vanaf laptop

Check eerst of DNS goed resolved:

```bash
nslookup vaultwarden-magnumopus.duckdns.org # toont naar welk IP het domein wijst
```

Als je in een lab zit, kan een hosts-file nodig zijn:

```text
172.16.120.11 vaultwarden-magnumopus.duckdns.org # forceer domein naar lab-VM
```

## Fout: SELinux blokkeert proxy

Controleer SELinux:

```bash
getenforce # toont Enforcing, Permissive of Disabled
```

Controleer boolean:

```bash
getsebool httpd_can_network_connect # controleert of Nginx/httpd mag verbinden naar backend
```

Zet tijdelijk/manueel:

```bash
sudo setsebool -P httpd_can_network_connect on # laat Nginx verbinding maken naar lokale backend
```

In jouw project doet Ansible dit automatisch.

## Fout: run 2 toont toch changes

Aanpak:

1. Kijk welke taak `changed` is.
2. Controleer of het een echte wijziging is of enkel een check.
3. Gebruik indien nodig een betere module.
4. Gebruik `creates` bij commands die een bestand opleveren.
5. Gebruik `changed_when: false` alleen bij echte checks.

## Mondeling sterke troubleshooting-zin

> Ik troubleshoot laag per laag: eerst SSH/Ansible-connectiviteit, dan packages en services, daarna containerstatus, dan lokale health endpoint, daarna Nginx, firewall, DNS en TLS. Zo weet ik snel of het probleem bij de backend, reverse proxy of netwerklaag zit.
