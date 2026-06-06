# Ansible codevoorbeelden met uitleg

Alle voorbeelden hebben commentaar bij de code. Gebruik ze om te begrijpen wat je op het mondeling zegt.

## Connectivity

```bash
ansible-inventory -i inventories/lab/hosts.yml --graph # toont inventorygroepen en hosts als boomstructuur
ansible all -i inventories/lab/hosts.yml -m ping # test of Linux-hosts bereikbaar zijn via Ansible
ansible win -i inventories/lab/hosts.yml -m ansible.windows.win_ping # test of Windows-hosts bereikbaar zijn via WinRM
```

## Basisplaybook

```yaml
- name: Configureer basisinstellingen
  hosts: all # draait op alle hosts uit de gekozen inventory
  become: true # gebruikt privilege escalation waar nodig
  gather_facts: true # verzamelt facts zodat OS-specifieke logica mogelijk is

  tasks:
    - name: Installeer basispackages
      ansible.builtin.package:
        name: "{{ base_packages }}" # lijst komt uit group_vars of defaults
        state: present # packages moeten geïnstalleerd zijn
```

## Users beheren

```yaml
- name: Maak gebruikers aan
  ansible.builtin.user:
    name: "{{ item.name }}" # username uit lijst
    groups: "{{ item.groups | default(omit) }}" # groepen alleen zetten als ze bestaan
    shell: "{{ item.shell | default('/bin/bash') }}" # standaard shell als niets is opgegeven
    state: present # gebruiker moet bestaan
  loop: "{{ managed_users }}" # lijst met gebruikers uit variabelen
```

## Service beheren

```yaml
- name: Zorg dat nginx draait en start bij boot
  ansible.builtin.service:
    name: nginx # servicenaam
    state: started # service moet actief zijn
    enabled: true # service moet automatisch starten
```

## Template + handler

```yaml
- name: Plaats applicatieconfiguratie
  ansible.builtin.template:
    src: app.conf.j2 # templatebestand
    dest: /etc/myapp/app.conf # doelpad op de server
    owner: root # eigenaar
    group: root # groep
    mode: '0644' # bestandsrechten
  notify: Herstart myapp # handler wordt alleen getriggerd bij wijziging
```

```yaml
handlers:
  - name: Herstart myapp
    ansible.builtin.service:
      name: myapp # servicenaam
      state: restarted # service herstarten na configuratiewijziging
```

## Check zonder changed

```yaml
- name: Controleer of webapp antwoordt
  ansible.builtin.uri:
    url: "http://localhost:8080/health" # health endpoint
    status_code: 200 # verwachte status
  register: health_result # bewaart status en response
  changed_when: false # een controle mag geen changed rapporteren
```

## Assert als bewijs

```yaml
- name: Bewijs dat webapp healthy is
  ansible.builtin.assert:
    that:
      - health_result.status == 200 # verwacht HTTP 200
    fail_msg: "Webapp is niet healthy" # duidelijke melding bij fout
    success_msg: "Webapp is healthy" # duidelijke melding bij succes
```

## Shell correct gebruiken

Gebruik shell alleen als er geen goede module is of wanneer shellfunctionaliteit nodig is.

```yaml
- name: Controleer custom binary versie
  ansible.builtin.command: /usr/local/bin/mytool --version # command zonder shellfeatures
  register: mytool_version # bewaart output
  changed_when: false # versiecheck verandert niets
  failed_when: mytool_version.rc != 0 # alleen rc 0 is succesvol
```

## Vault gebruiken

```yaml
- name: Login op API met secret
  ansible.builtin.uri:
    url: "https://api.example.local/login" # login endpoint
    method: POST # login stuurt data
    body_format: json # body wordt JSON
    body:
      username: "{{ api_user }}" # gewone of vaulted variabele
      password: "{{ api_password }}" # vaulted secret
    status_code: 200 # verwachte status
  no_log: true # voorkomt secret in output
```
