# 06 - Roles

## Kernzin voor het mondeling

**Roles zijn herbruikbare Ansible-bouwblokken die taken, defaults, handlers, templates en bestanden logisch bundelen zodat playbooks dun en overzichtelijk blijven.**

## Waarom roles nodig worden

In het begin kan één playbook nog overzichtelijk zijn. Maar zodra je packages, users, services, templates en firewallregels combineert, wordt een playbook te groot. Dan wil je herbruikbare bouwblokken.

Een role is zo'n bouwblok.

## Role-structuur

```text
roles/
└── nginx_reverse_proxy/
    ├── defaults/main.yml # standaardvariabelen die makkelijk te overschrijven zijn
    ├── vars/main.yml # interne variabelen met hogere precedence
    ├── tasks/main.yml # hoofdtaaklijst van de role
    ├── handlers/main.yml # handlers zoals service restart
    ├── templates/ # Jinja2 templates
    ├── files/ # statische bestanden
    ├── meta/main.yml # metadata en role dependencies
    └── README.md # documentatie over input en gebruik
```

## Role aanmaken

```bash
ansible-galaxy role init roles/nginx_reverse_proxy # maakt standaard role-structuur aan
```

## Thin playbook

Een playbook dat vooral roles aanroept is een **thin playbook**.

```yaml
- name: Configureer webservers
  hosts: web # draait op groep web
  become: true # voert taken uit met privilege escalation
  roles:
    - role: nginx_reverse_proxy # herbruikbare role voor nginx reverse proxy
```

## Defaults

`defaults/main.yml` bevat waarden die de gebruiker van de role makkelijk kan overschrijven.

```yaml
nginx_port: 80 # standaardpoort voor nginx
app_port: 8080 # standaardpoort van backendapplicatie
nginx_server_name: example.local # standaard server_name
```

## Tasks

```yaml
- name: Installeer nginx
  ansible.builtin.package:
    name: nginx # webserver package
    state: present # moet geïnstalleerd zijn

- name: Plaats nginx-template
  ansible.builtin.template:
    src: nginx.conf.j2 # template in role/templates
    dest: /etc/nginx/conf.d/app.conf # doelconfiguratie
    mode: '0644' # veilige bestandsrechten
  notify: Herstart nginx # handler alleen bij wijziging

- name: Zorg dat nginx draait
  ansible.builtin.service:
    name: nginx # servicenaam
    state: started # service moet actief zijn
    enabled: true # service start bij boot
```

## Handlers in role

```yaml
- name: Herstart nginx
  ansible.builtin.service:
    name: nginx # servicenaam
    state: restarted # herstart service na configwijziging
```

## Veelgemaakte fouten

- Alles in één mega-role stoppen.
- Variabelen hardcoden in tasks.
- Geen README bij role.
- Defaults en vars verwarren.
- Role alleen laten werken op jouw laptop door ontbrekende dependencies.

## Typische examenvraag

**Vraag:** Wat is het voordeel van roles?

**Sterk antwoord:**

Roles maken Ansible-code modulair en herbruikbaar. In plaats van één groot playbook plaats je taken, templates, handlers en defaults samen in een logisch bouwblok. Het playbook blijft dun en roept de role aan. Daardoor is het project beter onderhoudbaar, testbaar en herbruikbaar in meerdere omgevingen.

