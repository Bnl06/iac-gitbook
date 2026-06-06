---
title: Role vaultwarden volledig uitgelegd
description: Stap-voor-stap uitleg van de belangrijkste Vaultwarden role.
---

# Role `vaultwarden` volledig uitgelegd

De role `vaultwarden` is de kern van je project. Deze role deployt de applicatie en alles eromheen.

## Stap 1: vereiste instellingen controleren

```yaml
- name: Assert required public settings are configured # stopt vroeg als verplichte instellingen ontbreken
  ansible.builtin.assert: # assert-module voor validatie
    that: # voorwaarden die waar moeten zijn
      - vaultwarden_domain == "vaultwarden-magnumopus.duckdns.org" # controleert verwacht domein
      - acme_account_email is not match("^CHANGE_ME") # email moet aangepast zijn
      - vaultwarden_admin_token is defined # admin token moet bestaan
      - (vaultwarden_admin_token | default("")) | length > 20 # token moet lang genoeg zijn
      - duckdns_token is defined # DuckDNS token moet bestaan
      - (duckdns_token | default("")) | length > 20 # token moet lang genoeg zijn
    fail_msg: "Vul eerst inventories/lab/hosts.yml, acme_account_email en de versleutelde group_vars/all/vault.yml in." # duidelijke foutmelding
```

### Waarom is dit belangrijk?

Je voorkomt dat de deployment half start en later faalt door ontbrekende secrets of placeholderwaarden.

## Stap 2: directories maken

```yaml
- name: Create Vaultwarden and ACME directories # maakt alle nodige mappen
  ansible.builtin.file: # file-module beheert bestanden en mappen
    path: "{{ item.path }}" # pad uit de loop
    state: directory # het moet een map zijn
    owner: root # eigenaar root
    group: root # groep root
    mode: "{{ item.mode }}" # rechten per map
  loop: # lijst van mappen
    - path: "{{ vaultwarden_data_dir }}" # persistente Vaultwarden data
      mode: "0750" # niet publiek leesbaar
    - path: "{{ vaultwarden_env_dir }}" # locatie voor env-file
      mode: "0750" # beschermd
    - path: /etc/containers/systemd # locatie voor Quadlet units
      mode: "0755" # systemd/Podman moet kunnen lezen
    - path: /etc/letsencrypt # certificaatmap
      mode: "0755" # standaard certbot pad
    - path: /etc/letsencrypt/credentials # map voor DNS credentials
      mode: "0700" # alleen root toegang
    - path: /var/log/letsencrypt # logs van certbot
      mode: "0755" # logmap
```

## Stap 3: DuckDNS record updaten

```yaml
- name: Update DuckDNS record to the lab VM IP # update A-record naar lab-IP
  ansible.builtin.uri: # doet HTTP request vanuit Ansible
    url: "https://www.duckdns.org/update?domains={{ duckdns_subdomain }}&token={{ duckdns_token }}&ip={{ duckdns_target_ip }}" # DuckDNS update endpoint
    return_content: true # lees response body
  register: vaultwarden_duckdns_ip_update # bewaar response in variabele
  changed_when: false # markeer niet als changed, want dit is een check/update endpoint
  failed_when: (vaultwarden_duckdns_ip_update.content | trim) != "OK" # faal als DuckDNS niet OK teruggeeft
  no_log: true # verberg token in output
  when: duckdns_update_ip | bool # alleen uitvoeren als variabele true is
```

### Waarom `no_log: true`?

Omdat de URL het DuckDNS token bevat. Zonder `no_log` kan je token in logs terechtkomen.

## Stap 4: credentials template renderen

```yaml
- name: Render DuckDNS credentials for Certbot DNS challenge # maakt credentials file voor certbot
  ansible.builtin.template: # gebruikt Jinja2 template
    src: duckdns.ini.j2 # templatebestand in roles/vaultwarden/templates
    dest: /etc/letsencrypt/credentials/duckdns.ini # bestemming op target
    owner: root # root is eigenaar
    group: root # rootgroep
    mode: "0600" # alleen root kan lezen/schrijven
  no_log: true # token mag niet in logs verschijnen
```

Template:

```jinja
dns_duckdns_token={{ duckdns_token }} # token dat certbot gebruikt om DNS TXT records te plaatsen
```

## Stap 5: Vaultwarden environment file

```yaml
- name: Render Vaultwarden environment file # schrijft Vaultwarden configuratie naar env-file
  ansible.builtin.template: # Jinja2 template
    src: vaultwarden.env.j2 # bron-template
    dest: "{{ vaultwarden_env_dir }}/vaultwarden.env" # doelbestand
    owner: root # eigenaar root
    group: root # groep root
    mode: "0600" # bevat admin token, dus streng afschermen
  notify: Restart vaultwarden # herstart Vaultwarden als config wijzigt
```

Voorbeeld uit de template:

```jinja
DOMAIN={{ vaultwarden_domain_url }} # URL die Vaultwarden gebruikt
SIGNUPS_ALLOWED={{ vaultwarden_signups_allowed | lower }} # registratie aan/uit
ADMIN_TOKEN={{ vaultwarden_admin_token }} # admin token hash uit Ansible Vault
```

## Stap 6: Quadlet unit maken

```yaml
- name: Render Vaultwarden Quadlet unit # maakt Podman Quadlet container unit
  ansible.builtin.template: # template module
    src: vaultwarden.container.j2 # bronbestand
    dest: /etc/containers/systemd/vaultwarden.container # Quadlet locatie
    owner: root # eigenaar root
    group: root # groep root
    mode: "0644" # leesbaar voor systemd
  notify: # handlers die nodig zijn bij wijziging
    - Reload systemd # systemd moet nieuwe unit zien
    - Restart vaultwarden # service herstarten met nieuwe config
```

Belangrijk uit de Quadlet:

```ini
Image={{ vaultwarden_image }} # container image voor Vaultwarden
EnvironmentFile={{ vaultwarden_env_dir }}/vaultwarden.env # laad configuratie uit env-file
Volume={{ vaultwarden_data_dir }}:/data:Z # persistente data met SELinux label
PublishPort={{ vaultwarden_http_bind }}:{{ vaultwarden_http_port }}:80 # host 127.0.0.1:8080 naar containerpoort 80
Pull=missing # pull image alleen als die ontbreekt
```

## Stap 7: container images pullen

```yaml
- name: Pull Vaultwarden image # downloadt Vaultwarden image indien nodig
  containers.podman.podman_image: # Podman image module
    name: "{{ vaultwarden_image }}" # image uit variabele
    state: present # image moet aanwezig zijn
```

```yaml
- name: Pull Certbot DuckDNS image # downloadt certbot image met DuckDNS plugin
  containers.podman.podman_image: # Podman image module
    name: "{{ certbot_duckdns_image }}" # certbot image
    state: present # image moet aanwezig zijn
```

## Stap 8: handlers flushen

```yaml
- name: Flush handlers before starting Vaultwarden # voert pending handlers nu al uit
  ansible.builtin.meta: flush_handlers # forceert handlers vóór de volgende taken
```

Waarom? Als de Quadlet unit net gewijzigd is, moet systemd eerst reloaden voordat je de service start.

## Stap 9: service starten

```yaml
- name: Enable and start Vaultwarden service # zet service aan en start die
  ansible.builtin.systemd_service: # systemd module
    name: vaultwarden.service # service die door Quadlet ontstaat
    enabled: true # start automatisch bij reboot
    state: started # moet nu draaien
    daemon_reload: true # systemd units opnieuw inlezen
```

## Stap 10: health check

```yaml
- name: Wait until Vaultwarden answers locally # wacht tot Vaultwarden gezond is
  ansible.builtin.uri: # HTTP request module
    url: "http://{{ vaultwarden_http_bind }}:{{ vaultwarden_http_port }}/alive" # lokale health endpoint
    status_code: 200 # verwachte HTTP status
    return_content: false # inhoud is niet nodig
  register: vaultwarden_health # bewaar resultaat
  retries: 36 # maximaal 36 pogingen
  delay: 5 # 5 seconden tussen pogingen
  until: vaultwarden_health.status == 200 # stop als HTTP 200 terugkomt
```

### Waarom health check?

Je wil niet Nginx configureren voor een backend die nog niet werkt. Dit maakt de deployment betrouwbaarder.

## Stap 11: certificaat aanvragen

```yaml
- name: Request Let's Encrypt certificate with DuckDNS DNS-01 # vraagt TLS-certificaat aan via DNS-01
  ansible.builtin.command: # voert command uit omdat certbot in container draait
    argv: # veilige lijstvorm, geen shell parsing
      - podman # container runtime
      - run # start tijdelijke container
      - --rm # verwijder container na afloop
      - --network # netwerkoptie
      - host # gebruik host networking
      - --security-opt # security optie voor SELinux
      - label=disable # voorkomt SELinux labelproblemen bij certbot volumes
      - -v # volume mount
      - /etc/letsencrypt:/etc/letsencrypt # certbot certificaten persistent bewaren
      - -v # volume mount
      - /var/log/letsencrypt:/var/log/letsencrypt # logs bewaren
      - -v # volume mount
      - /etc/letsencrypt/credentials/duckdns.ini:/conf/duckdns.ini:ro # credentials read-only in container
      - "{{ certbot_duckdns_image }}" # certbot image met DuckDNS plugin
      - certonly # alleen certificaat aanvragen, geen webserver aanpassen
      - --non-interactive # geen interactieve vragen
      - --agree-tos # akkoord met Let's Encrypt voorwaarden
      - --email # account email volgt
      - "{{ acme_account_email }}" # Let's Encrypt account email
      - --preferred-challenges # challenge type volgt
      - dns # DNS challenge gebruiken
      - --authenticator # authenticator plugin volgt
      - dns-duckdns # DuckDNS plugin
      - --dns-duckdns-credentials # credentials file volgt
      - /conf/duckdns.ini # pad binnen container
      - --dns-duckdns-propagation-seconds # wachttijd DNS verspreiding
      - "{{ duckdns_propagation_seconds | string }}" # bijvoorbeeld 120 seconden
      - -d # domein volgt
      - "{{ vaultwarden_domain }}" # domein waarvoor certificaat wordt aangevraagd
    creates: "/etc/letsencrypt/live/{{ vaultwarden_domain }}/fullchain.pem" # idempotentie: niet opnieuw aanvragen als cert bestaat
```

### Waarom `creates` belangrijk is

Zonder `creates` zou de command elke run opnieuw draaien. Met `creates` weet Ansible: als dit bestand bestaat, is de taak al gedaan.

## Stap 12: Nginx config renderen

```yaml
- name: Render final HTTPS reverse proxy config # schrijft Nginx config voor HTTPS proxy
  ansible.builtin.template: # gebruikt Jinja2 template
    src: nginx-https.conf.j2 # Nginx template
    dest: /etc/nginx/conf.d/vaultwarden.conf # Nginx config op target
    owner: root # eigenaar root
    group: root # groep root
    mode: "0644" # leesbaar voor Nginx
  when: vaultwarden_cert_after.stat.exists # alleen als certificaat bestaat
  notify: Reload nginx # herlaad Nginx bij wijziging
```

## Stap 13: Nginx valideren

```yaml
- name: Validate Nginx configuration # controleert of Nginx config geldig is
  ansible.builtin.command: # voert command uit
    argv: # lijstvorm is veiliger dan shell string
      - nginx # programma
      - -t # test configuratie
  changed_when: false # configuratietest verandert niets
```

## Stap 14: renewal via cron

```yaml
- name: Configure automated certificate renewal # maakt cronjob voor certificaatvernieuwing
  ansible.builtin.cron: # beheert cronjobs idempotent
    name: Renew Let's Encrypt certificates for Vaultwarden # unieke naam van de cronjob
    user: root # cronjob draait als root
    minute: "17" # minuut 17
    hour: "3" # om 03:17 's nachts
    job: >- # command over meerdere regels
      podman run --rm --network host --security-opt label=disable
      -v /etc/letsencrypt:/etc/letsencrypt
      -v /var/log/letsencrypt:/var/log/letsencrypt
      -v /etc/letsencrypt/credentials/duckdns.ini:/conf/duckdns.ini:ro
      {{ certbot_duckdns_image }} renew --quiet
      --deploy-hook 'systemctl reload nginx'
```

## Kernantwoord voor mondeling

> De `vaultwarden` role valideert eerst of alle nodige variabelen en secrets bestaan. Daarna maakt ze directories, schrijft templates, configureert Quadlet, trekt container images binnen, start Vaultwarden, controleert de health endpoint, vraagt een certificaat aan via Certbot DNS-01, configureert Nginx als HTTPS reverse proxy en zet automatische renewal klaar via cron. De role gebruikt handlers zodat services alleen herstarten als configuratie wijzigt.
