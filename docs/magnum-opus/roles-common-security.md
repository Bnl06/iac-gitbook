---
title: Roles common en security_baseline
description: Grondige uitleg van de basis- en securityrollen van het Magnum Opus-project.
---

# Roles: `common` en `security_baseline`

## Waarom roles?

Roles maken je project overzichtelijk. In plaats van één lang playbook splits je taken per verantwoordelijkheid.

| Role | Verantwoordelijkheid |
|---|---|
| `common` | basiscontrole, repositories, packages en services |
| `security_baseline` | firewall, logging en SELinux |
| `vaultwarden` | applicatie, container, TLS, Nginx en renewal |

## Role `common`

### 1. Controleren of het juiste OS gebruikt wordt

```yaml
- name: Assert supported target distribution # controleert of de target ondersteund is
  ansible.builtin.assert: # assert faalt bewust als voorwaarden niet kloppen
    that: # voorwaarden die waar moeten zijn
      - ansible_facts.os_family == "RedHat" # enkel RedHat-family zoals Rocky/Alma
      - ansible_facts.distribution_major_version | int == 10 # enkel major version 10
    fail_msg: "Dit project is getest en bedoeld voor Rocky Linux 10 of AlmaLinux 10." # foutmelding bij verkeerde target
```

### Waarom is dit goed?

Je voorkomt dat je playbook op een verkeerd systeem draait. Dat is professioneel, want je maakt je scope expliciet.

Mondeling:

> Ik gebruik een assert om vroeg te falen als de target geen Rocky/Alma Linux 10 is. Zo voorkom ik onverwachte fouten later in de deployment.

## 2. EPEL repository inschakelen

```yaml
- name: Enable EPEL repository # zorgt dat extra packages beschikbaar zijn
  ansible.builtin.dnf: # gebruikt de package manager op RedHat-family systemen
    name: epel-release # package dat de EPEL repository activeert
    state: present # moet geïnstalleerd zijn
```

EPEL bevat extra packages die niet altijd standaard in de basisrepositories zitten.

## 3. Vereiste packages installeren

```yaml
- name: Install required packages # installeert alle basispackages
  ansible.builtin.dnf: # gebruikt dnf als package manager
    name: # lijst van packages
      - cronie # nodig voor cronjobs zoals certificaatrenewal
      - firewalld # firewallbeheer
      - nginx # reverse proxy
      - podman # container runtime
      - policycoreutils-python-utils # nodig voor SELinux tools/modules
    state: present # packages moeten aanwezig zijn
    update_cache: true # vernieuwt package metadata
```

### Waarom deze packages?

| Package | Reden |
|---|---|
| `cronie` | automatische cert-renewal via cron |
| `firewalld` | firewall beheren met Ansible |
| `nginx` | reverse proxy en HTTPS |
| `podman` | container draaien zonder Docker daemon |
| `policycoreutils-python-utils` | SELinux configuratie beheren |

## 4. Basisservices starten

```yaml
- name: Enable and start base services # zorgt dat basisservices draaien en opstarten bij boot
  ansible.builtin.systemd_service: # beheert systemd services
    name: "{{ item }}" # service uit de loop
    enabled: true # start automatisch bij boot
    state: started # service moet nu draaien
  loop: # herhaalt de taak voor meerdere services
    - crond # cron daemon
    - firewalld # firewall daemon
```

### Wat betekent `loop`?

`loop` zorgt dat dezelfde taak meerdere keren wordt uitgevoerd, telkens met een andere waarde voor `item`.

## Role `security_baseline`

Deze role beperkt wat openstaat en zorgt dat security relevant aantoonbaar is.

## 1. Firewalld denied logging

```yaml
- name: Configure firewalld denied logging # zet logging aan voor geweigerd verkeer
  ansible.builtin.lineinfile: # beheert één lijn in een tekstbestand
    path: /etc/firewalld/firewalld.conf # configuratiebestand van firewalld
    regexp: "^LogDenied=" # zoekt bestaande LogDenied-regel
    line: "LogDenied=all" # gewenste regel
    owner: root # eigenaar blijft root
    group: root # groep blijft root
    mode: "0644" # leesbaar maar alleen root schrijft
  notify: Reload firewalld # triggert handler als het bestand wijzigt
```

### Waarom `lineinfile`?

Je wil niet heel het configuratiebestand overschrijven. Je wil alleen één instelling beheren.

## 2. Alleen noodzakelijke services openen

```yaml
- name: Open only required public services # opent enkel de services die nodig zijn
  ansible.posix.firewalld: # module om firewalld te beheren
    service: "{{ item }}" # service uit firewall_allowed_services
    permanent: true # blijft na reboot bestaan
    immediate: true # wordt direct actief
    state: enabled # service wordt geopend
  loop: "{{ firewall_allowed_services }}" # herhaal voor ssh, http en https
```

De variabele komt uit `group_vars/all/main.yml`:

```yaml
firewall_allowed_services:
  - ssh # Ansible/SSH toegang
  - http # redirect naar HTTPS
  - https # echte webtoegang
```

## 3. SELinux toestaan dat Nginx proxyt

```yaml
- name: Allow Nginx to proxy to local Vaultwarden under SELinux # laat Nginx naar backend proxyen
  ansible.posix.seboolean: # beheert SELinux booleans
    name: httpd_can_network_connect # geeft HTTP-daemons toestemming voor netwerkconnecties
    state: true # boolean aanzetten
    persistent: true # behouden na reboot
  when: # alleen uitvoeren als SELinux actief is
    - ansible_facts.selinux is defined # SELinux facts bestaan
    - ansible_facts.selinux.status == "enabled" # SELinux staat effectief aan
```

### Waarom `when`?

Als SELinux niet actief is of facts niet bestaan, zou deze taak niet nodig zijn. Met `when` maak je je playbook robuuster.

## Handler: firewalld reload

```yaml
- name: Reload firewalld # handlernaam die aangeroepen wordt door notify
  ansible.builtin.systemd_service: # beheert systemd
    name: firewalld # firewalld service
    state: reloaded # herlaadt configuratie zonder volledige restart
```

## Kernantwoord voor mondeling

> De `common` role zorgt voor de basis: juiste distributie, repositories, packages en basisservices. De `security_baseline` role beperkt daarna de aanvalsvector: alleen ssh, http en https zijn open, denied logging staat aan en SELinux laat Nginx enkel toe om nodig proxyverkeer te doen. Zo combineer ik automatisatie met security.
