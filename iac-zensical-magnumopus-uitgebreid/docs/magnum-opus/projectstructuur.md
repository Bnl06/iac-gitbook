---
title: Projectstructuur uitgelegd
description: Uitleg van alle belangrijke mappen en bestanden in het Magnum Opus-project.
---

# Projectstructuur uitgelegd

Je Magnum Opus is opgebouwd als een klassieke Ansible-repository met inventories, group variables, playbooks en roles.

```text
.
|-- ansible.cfg
|-- requirements.yml
|-- inventories/lab/hosts.yml
|-- group_vars/all/main.yml
|-- group_vars/all/vault.yml
|-- playbooks/site.yml
|-- roles/common/
|-- roles/security_baseline/
|-- roles/vaultwarden/
|-- docs/evidence/
`-- .github/workflows/ansible-quality.yml
```

## `ansible.cfg`

```ini
[defaults]
inventory = inventories/lab/ # standaard inventorymap; je moet dus niet telkens -i meegeven
roles_path = roles # Ansible zoekt roles in de map roles
host_key_checking = False # praktisch in lab, maar productie gebruikt beter bekende host keys
retry_files_enabled = False # maakt geen oude .retry-bestanden aan
result_format = yaml # output is leesbaarder
interpreter_python = auto_silent # Ansible kiest automatisch de juiste Python interpreter

[privilege_escalation]
become = True # taken draaien standaard met sudo
become_method = sudo # sudo is de gebruikte privilege escalation methode
become_ask_pass = False # vraagt geen sudo-wachtwoord tenzij je tijdelijk -K gebruikt
```

### Mondeling uitleggen

> In `ansible.cfg` leg ik projectbrede instellingen vast. Daardoor wordt mijn project eenvoudiger uit te voeren: Ansible weet automatisch waar de inventory en roles staan, en taken worden standaard met sudo uitgevoerd.

## Inventory: `inventories/lab/hosts.yml`

```yaml
--- # start van YAML-document
all: # hoogste groep in Ansible
  children: # subgroepen onder all
    vaultwarden: # groep voor hosts waarop Vaultwarden moet komen
      vars: # variabelen voor deze groep
        ansible_user: youness # SSH-user waarmee Ansible verbindt
        ansible_ssh_private_key_file: ~/.ssh/id_ed25519_iac # private key voor SSH-login
      hosts: # hosts binnen deze groep
        rocky1: # logische hostnaam binnen Ansible
          ansible_host: 172.16.120.11 # echte IP-adres van de VM
```

### Belangrijk verschil

| Naam | Betekenis |
|---|---|
| `rocky1` | Ansible-hostnaam |
| `172.16.120.11` | IP-adres waar SSH naartoe gaat |
| `vaultwarden` | groep waarop het playbook draait |

## `group_vars/all/main.yml`

Hier staan gewone variabelen die niet geheim zijn.

```yaml
vaultwarden_domain: vaultwarden-magnumopus.duckdns.org # domeinnaam van de service
vaultwarden_domain_url: "https://{{ vaultwarden_domain }}" # volledige URL voor Vaultwarden config
vaultwarden_image: docker.io/vaultwarden/server:latest # container image voor Vaultwarden
vaultwarden_data_dir: /opt/vaultwarden/data # persistente data op de host
vaultwarden_env_dir: /etc/vaultwarden # map waar de env-file komt
vaultwarden_http_bind: 127.0.0.1 # Vaultwarden luistert alleen lokaal
vaultwarden_http_port: 8080 # lokale poort achter Nginx
```

### Waarom variabelen?

Variabelen zorgen dat je code niet hardcoded is. Als je later een ander domein, andere poort of andere image wilt gebruiken, pas je één plek aan.

## `group_vars/all/vault.yml`

Hier staan geheime waarden, zoals:

```yaml
vaultwarden_admin_token: '$argon2id$v=19$m=...' # hash voor adminpaneel van Vaultwarden
duckdns_token: "JOUW_DUCKDNS_TOKEN" # token om DuckDNS DNS-records aan te passen
```

Dit bestand moet versleuteld zijn met Ansible Vault.

```bash
ansible-vault encrypt group_vars/all/vault.yml # versleutelt het secrets-bestand
head group_vars/all/vault.yml # controleert of het bestand begint met $ANSIBLE_VAULT
```

## `playbooks/site.yml`

```yaml
--- # start YAML-document
- name: Deploy Vaultwarden with Podman, Nginx and TLS # duidelijke naam van de play
  hosts: vaultwarden # voer deze play uit op de groep vaultwarden
  become: true # gebruik sudo/become voor beheertaken
  gather_facts: true # verzamel systeeminformatie zoals OS, SELinux en distributie
  vars_files: # laad variabelenbestanden expliciet in
    - ../group_vars/all/main.yml # gewone variabelen
    - ../group_vars/all/vault.yml # geheime variabelen via Ansible Vault

  roles: # roles worden in deze volgorde uitgevoerd
    - role: common # basispackages en services
    - role: security_baseline # firewall en SELinux
    - role: vaultwarden # applicatie, TLS, Nginx en renewal
```

### Waarom deze volgorde?

1. `common` moet eerst, want Podman/Nginx/firewalld moeten geïnstalleerd zijn.
2. `security_baseline` zet firewall en SELinux klaar.
3. `vaultwarden` gebruikt die basis om de applicatie te deployen.

## `requirements.yml`

```yaml
--- # start YAML-document
collections: # externe Ansible collections die nodig zijn
  - name: ansible.posix # bevat firewalld en SELinux modules
  - name: containers.podman # bevat Podman modules zoals podman_image
```

Installeren:

```bash
ansible-galaxy collection install -r requirements.yml # installeert collections die je project nodig heeft
```

## `.github/workflows/ansible-quality.yml`

Deze workflow doet geen echte deployment. Hij controleert de kwaliteit:

```yaml
- name: Syntax check # controleert of het playbook geldig is
  run: ansible-playbook --syntax-check playbooks/site.yml

- name: Ansible lint # controleert stijl en best practices
  run: ansible-lint
```

### Mondeling uitleggen

> Mijn GitHub Actions workflow is een beperkte CI-check. Hij deployt niet naar mijn lab-VM, want GitHub kan die privé-VM niet bereiken. Hij controleert wel syntax en linting, zodat fouten sneller zichtbaar worden.
