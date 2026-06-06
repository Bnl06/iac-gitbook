# 02 - Ansible projectstructuur

## Kernzin voor het mondeling

**Een Ansible-project heeft structuur nodig zodat inventory, variabelen, playbooks, dependencies, roles en documentatie reproduceerbaar samenwerken vanuit Git als bron van waarheid.**

## Waarom structuur geen luxe is

Zonder structuur krijg je snel copy-paste drift. Dat betekent dat verschillende playbooks bijna hetzelfde doen, maar net anders. Daardoor ontstaan fouten: hardcoded waarden, oude variabelen, niet-idempotente taken en onduidelijke dependencies.

Een goede repo maakt duidelijk:

- welke hosts beheerd worden;
- welke playbooks bestaan;
- welke variabelen per omgeving gelden;
- welke roles en collections nodig zijn;
- hoe je het project uitvoert;
- hoe je controleert dat het werkt.

## Minimumstructuur

```text
iac-ansible/
├── ansible.cfg # projectinstellingen voor Ansible
├── requirements.yml # externe collections en roles
├── README.md # hoe gebruik je dit project?
├── .gitignore # voorkomt dat secrets en rommel in Git komen
├── playbooks/
│   └── site.yml # hoofdplaybook of startpunt
├── inventories/
│   └── lab/
│       └── hosts.yml # hosts en groepen voor lab
├── group_vars/ # variabelen per groep
└── host_vars/ # variabelen per host
```

## Vollediger projectbeeld

```text
iac-ansible/
├── ansible.cfg # vaste instellingen zoals inventory path en roles path
├── requirements.yml # collections/roles die nodig zijn
├── ansible-lint.yml # kwaliteitsregels voor linting
├── README.md # praktische uitvoerdocumentatie
├── playbooks/
│   ├── site.yml # thin playbook dat roles aanroept
│   ├── baseline.yml # bijvoorbeeld basisconfiguratie
│   └── nginx.yml # specifieke applicatieconfiguratie
├── inventories/
│   ├── lab/
│   │   ├── hosts.yml # lab-hosts
│   │   ├── group_vars/ # lab-groepsvariabelen
│   │   └── host_vars/ # lab-hostvariabelen
│   └── prod/
│       ├── hosts.yml # productiehosts
│       ├── group_vars/ # productievariabelen
│       └── host_vars/ # productie-hostvars
└── roles/
    └── linux_baseline/
        ├── defaults/main.yml # standaardwaarden, makkelijk overschrijfbaar
        ├── vars/main.yml # interne variabelen van de role
        ├── tasks/main.yml # taken van de role
        ├── handlers/main.yml # handlers zoals service restart
        ├── templates/ # Jinja2 templates
        ├── files/ # statische bestanden
        ├── meta/main.yml # metadata/dependencies
        └── README.md # uitleg over role-input en output
```

## ansible.cfg

`ansible.cfg` zorgt dat iedereen dezelfde projectinstellingen gebruikt.

```ini
[defaults]
inventory = inventories/lab/hosts.yml # standaard inventory voor lokale runs
roles_path = roles # waar Ansible lokale roles zoekt
collections_path = collections # waar projectcollections staan
host_key_checking = False # handig in lab, maar in productie voorzichtig gebruiken
stdout_callback = yaml # leesbaardere output
```

## Inventory

Inventory bepaalt **waar** Ansible op draait.

```yaml
all:
  children:
    web:
      hosts:
        web01:
          ansible_host: 192.168.56.10 # IP of DNS-naam waarmee Ansible verbindt
    db:
      hosts:
        db01:
          ansible_host: 192.168.56.20 # databasehost in de labomgeving
```

Runnen:

```bash
ansible-inventory -i inventories/lab/hosts.yml --graph # toont groepen en hosts als controle
ansible all -i inventories/lab/hosts.yml -m ping # test basisconnectiviteit naar alle hosts
```

## requirements.yml

Dependencies moeten reproduceerbaar zijn. Je wil niet dat jouw project alleen werkt omdat op jouw laptop toevallig een collection geïnstalleerd staat.

```yaml
collections:
  - name: community.general # extra modules voor algemene automatisering
    version: ">=8.0.0" # versieconstraint zodat gedrag voorspelbaarder blijft
  - name: ansible.windows # Windows-modules voor WinRM-beheer
    version: ">=2.0.0" # minimaal benodigde versie
```

Installeren:

```bash
ansible-galaxy collection install -r requirements.yml # installeert alle collections uit requirements.yml
```

## Modules boven command/shell

Gebruik liever modules dan `command` of `shell`, omdat modules idempotenter en leesbaarder zijn.

Slecht:

```yaml
- name: Installeer nginx met shell
  ansible.builtin.shell: apt install -y nginx # niet ideaal: Ansible weet minder goed of er iets veranderde
```

Beter:

```yaml
- name: Installeer nginx via package module
  ansible.builtin.package:
    name: nginx # package die aanwezig moet zijn
    state: present # idempotente gewenste toestand
```

## README is examenmateriaal

Een goede README moet antwoorden op:

- Wat doet dit project?
- Welke dependencies zijn nodig?
- Welke inventory gebruik je?
- Hoe installeer je requirements?
- Hoe voer je het playbook uit?
- Hoe gebruik je Vault?
- Hoe controleer je het resultaat?

Voorbeeld:

```bash
ansible-galaxy collection install -r requirements.yml # haalt benodigde collections binnen
ansible-playbook -i inventories/lab/hosts.yml playbooks/site.yml --ask-vault-pass # voert project uit met Vault-prompt
```

## Typische examenvraag

**Vraag:** Waarom moet een Ansible-project een duidelijke structuur hebben?

**Sterk antwoord:**

Omdat IaC reproduceerbaar en reviewbaar moet zijn. De structuur scheidt inventory, variabelen, playbooks, roles en dependencies. Daardoor vermijd je hardcoding en copy-paste drift. Een fresh clone moet uitvoerbaar zijn: iemand anders moet requirements kunnen installeren, de juiste inventory kiezen en het playbook kunnen starten zonder geheime lokale kennis.

## Mini-checklist

- Kan je het verschil tussen `playbooks/`, `roles/`, `inventories/`, `group_vars/` en `host_vars/` uitleggen?
- Kan je uitleggen waarom `requirements.yml` nodig is?
- Kan je uitleggen waarom modules beter zijn dan shell/command?
- Kan je zeggen wat in README moet staan?
