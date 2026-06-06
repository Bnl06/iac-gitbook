---
title: Magnum Opus checklist
description: Checklist om te controleren of je alles van je project kent.
---

# Magnum Opus checklist

Gebruik deze checklist vlak voor je mondeling.

## Basisbegrip

- [ ] Ik kan in 30 seconden uitleggen wat mijn project doet.
- [ ] Ik weet waarom Vaultwarden een goede applicatie is voor IaC.
- [ ] Ik kan het verschil uitleggen tussen control node en target node.
- [ ] Ik weet welke OS-versie ondersteund wordt.
- [ ] Ik weet waarom mijn project op Rocky/Alma Linux gericht is.

## Ansible

- [ ] Ik kan `ansible.cfg` uitleggen.
- [ ] Ik kan mijn inventory uitleggen.
- [ ] Ik weet waarom ik `group_vars/all/main.yml` gebruik.
- [ ] Ik weet waarom `vault.yml` versleuteld is.
- [ ] Ik kan `playbooks/site.yml` stap voor stap uitleggen.
- [ ] Ik kan uitleggen waarom roles in deze volgorde draaien: `common`, `security_baseline`, `vaultwarden`.

## Roles

- [ ] Ik kan uitleggen wat de `common` role doet.
- [ ] Ik kan uitleggen wat de `security_baseline` role doet.
- [ ] Ik kan uitleggen wat de `vaultwarden` role doet.
- [ ] Ik weet wat handlers zijn.
- [ ] Ik weet waarom handlers beter zijn dan altijd services herstarten.

## Security

- [ ] Ik weet waarom alleen ssh/http/https openstaan.
- [ ] Ik kan firewalld uitleggen.
- [ ] Ik kan SELinux boolean `httpd_can_network_connect` uitleggen.
- [ ] Ik weet waarom Vaultwarden alleen op `127.0.0.1` luistert.
- [ ] Ik weet waarom secrets niet in Git mogen.
- [ ] Ik weet wat `no_log: true` doet.

## TLS en DNS

- [ ] Ik kan uitleggen wat Let's Encrypt doet.
- [ ] Ik kan DNS-01 uitleggen.
- [ ] Ik weet waarom HTTP-01 niet ideaal is in mijn labomgeving.
- [ ] Ik weet waarvoor DuckDNS gebruikt wordt.
- [ ] Ik weet waarom `creates` belangrijk is bij Certbot.
- [ ] Ik weet hoe certificaatrenewal via cron werkt.

## Podman en Nginx

- [ ] Ik kan Podman uitleggen.
- [ ] Ik kan Quadlet uitleggen.
- [ ] Ik kan systemd-servicebeheer uitleggen.
- [ ] Ik kan Nginx reverse proxy uitleggen.
- [ ] Ik weet wat `proxy_pass` doet.
- [ ] Ik weet waarom websockets headers in Nginx staan.

## Idempotentie

- [ ] Ik kan idempotentie definiëren.
- [ ] Ik kan uitleggen waarom run 2 belangrijk is.
- [ ] Ik kan `changed=0` uitleggen.
- [ ] Ik weet wanneer je `changed_when: false` gebruikt.
- [ ] Ik weet waarom Ansible modules beter zijn dan shell commands.

## Demo klaarzetten

- [ ] Ik heb mijn Vault password.
- [ ] Mijn VM staat aan.
- [ ] SSH werkt.
- [ ] `ansible rocky1 -m ping` werkt.
- [ ] Mijn bewijsbestanden staan in `docs/evidence`.
- [ ] Ik kan `/alive` tonen.
- [ ] Ik kan `podman ps` tonen.
- [ ] Ik kan `firewall-cmd --list-all` tonen.

## Laatste zin voor het examen

> Mijn project is niet alleen een werkende deployment, maar ook reproduceerbaar, idempotent, beveiligd en controleerbaar met bewijsstukken.
