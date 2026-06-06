---
title: Mondelinge verdediging Magnum Opus
description: Voorbeeldvragen en sterke antwoorden voor je mondeling examen over je eigen project.
---

# Mondelinge verdediging Magnum Opus

Gebruik deze pagina om hardop te oefenen.

## 1. Wat doet jouw project?

**Sterk antwoord:**

> Mijn project deployt Vaultwarden, een self-hosted password manager, automatisch op een Rocky/Alma Linux 10 VM. Ik gebruik Ansible om packages te installeren, firewall en SELinux te configureren, Vaultwarden als Podman container via Quadlet te draaien, Nginx als HTTPS reverse proxy te zetten en een Let's Encrypt certificaat aan te vragen via DuckDNS DNS-01. Secrets beheer ik met Ansible Vault en ik bewijs idempotentie met twee runs.

## 2. Waarom heb je voor Vaultwarden gekozen?

> Vaultwarden is een realistische applicatie met meerdere infrastructuurcomponenten: container runtime, persistent storage, reverse proxy, TLS, firewall, secrets en servicebeheer. Daardoor toont het project meer IaC-kennis dan alleen een simpele webserver installeren.

## 3. Waarom gebruik je Ansible roles?

> Roles zorgen voor structuur en herbruikbaarheid. Mijn `common` role doet basisinstallatie, `security_baseline` doet firewall en SELinux, en `vaultwarden` doet de applicatiedeployment. Zo blijft het project overzichtelijk en verdedigbaar.

## 4. Waarom Podman en niet Docker?

> Podman past goed bij RedHat-family systemen zoals Rocky en Alma Linux. Het werkt daemonless en integreert goed met systemd via Quadlet. Daardoor kan ik de container beheren als een normale Linux-service.

## 5. Wat is Quadlet?

> Quadlet is een Podman-manier om containers declaratief te beschrijven in `.container` bestanden. Systemd zet die om naar services. Daardoor kan ik `systemctl status vaultwarden.service`, `systemctl restart` en `enable` gebruiken alsof de container een gewone service is.

## 6. Waarom luistert Vaultwarden op 127.0.0.1?

> Omdat Nginx de publieke ingang is. Vaultwarden hoeft niet rechtstreeks vanaf het netwerk bereikbaar te zijn. Door te binden op `127.0.0.1:8080` verklein ik de attack surface.

## 7. Waarom gebruik je Nginx?

> Nginx verzorgt HTTPS, reverse proxy, redirects en security headers. Het stuurt inkomend verkeer op 443 veilig door naar de lokale Vaultwarden backend.

## 8. Waarom DNS-01 in plaats van HTTP-01?

> Mijn VM draait in een lab/VMware omgeving en heeft geen publiek IP. HTTP-01 vereist dat Let's Encrypt mijn server via poort 80 kan bereiken. DNS-01 vereist alleen dat ik een DNS TXT-record kan aanpassen via DuckDNS, dus dat werkt ook zonder publieke port-forward.

## 9. Hoe beheer je secrets?

> Secrets zoals het DuckDNS token en Vaultwarden admin token staan in `group_vars/all/vault.yml`, versleuteld met Ansible Vault. Daarnaast gebruik ik `no_log: true` bij taken waar secrets in output zouden kunnen verschijnen.

## 10. Wat bewijst idempotentie in jouw project?

> Mijn tweede run eindigt met `changed=0` en `failed=0`. Dat bewijst dat de gewenste staat al bereikt was en dat het playbook veilig opnieuw uitgevoerd kan worden zonder onnodige wijzigingen.

## 11. Welke taak vond je technisch het moeilijkst?

> De TLS-configuratie via DNS-01 was het moeilijkst, omdat de VM niet publiek bereikbaar is. Ik moest daarom Certbot in een Podman container draaien, DuckDNS credentials veilig templaten, DNS-propagatie voorzien en met `creates` voorkomen dat het certificaat opnieuw wordt aangevraagd bij elke run.

## 12. Wat zou je verbeteren als je meer tijd had?

> Ik zou backups toevoegen voor `/opt/vaultwarden/data`, monitoring voorzien en eventueel SMTP configureren. Voor productie zou ik ook host key checking aanzetten en misschien PostgreSQL gebruiken. Maar voor deze opdracht heb ik bewust de scope beperkt tot een sterke en reproduceerbare basisdeployment.

## 13. Wat is je sterkste technische keuze?

> De combinatie van Podman Quadlet, Nginx reverse proxy, DNS-01 TLS en Ansible Vault. Die toont dat ik niet alleen packages installeer, maar een volledige veilige service lifecycle automatiseer.

## 14. Wat is het verschil tussen je applicatiepoort en publieke poort?

> Publiek gebruikt de gebruiker HTTPS op poort 443 via Nginx. Intern stuurt Nginx door naar Vaultwarden op `127.0.0.1:8080`. De containerpoort is 80, maar die wordt alleen lokaal gepubliceerd.

## 15. Wat als de docent vraagt om een demo?

Volg deze volgorde:

```bash
ansible rocky1 -m ping # toon dat Ansible de VM bereikt
ansible-playbook --syntax-check playbooks/site.yml --ask-vault-pass # toon dat playbook syntactisch correct is
ansible-playbook playbooks/site.yml --ask-vault-pass -K # voer deployment of herhaalrun uit
curl https://vaultwarden-magnumopus.duckdns.org/alive # toon dat Vaultwarden gezond antwoordt
ansible rocky1 -b -m command -a "podman ps" # toon dat container draait
ansible rocky1 -b -m command -a "firewall-cmd --list-all" # toon firewallconfiguratie
```

## Mini-spiekbrief voor het mondeling

```text
Project = Vaultwarden password manager
IaC tool = Ansible
Target = Rocky/Alma Linux 10
Runtime = Podman
Servicebeheer = Quadlet + systemd
Proxy = Nginx
TLS = Let's Encrypt via DuckDNS DNS-01
Secrets = Ansible Vault
Security = firewalld + SELinux + lokale bind
Bewijs = run1 + run2, run2 changed=0
```
