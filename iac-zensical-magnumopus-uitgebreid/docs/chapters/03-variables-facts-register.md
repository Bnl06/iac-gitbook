# 03 - Variables, facts en register

## Kernzin voor het mondeling

**Variables, facts en register maken Ansible data-gedreven: playbooks beschrijven de logica, terwijl waarden uit inventory, group_vars, host_vars, facts of taakoutput komen.**

## Van hardcoded naar data-gedreven

Een playbook moet niet vol vaste IP-adressen, poorten en gebruikersnamen staan. Dat maakt het moeilijk herbruikbaar. Beter is: de taak beschrijft wat moet gebeuren, en de concrete waarden komen uit data.

Slecht:

```yaml
- name: Maak vaste user aan
  ansible.builtin.user:
    name: jan # hardcoded naam: minder herbruikbaar
    state: present # user moet bestaan
```

Beter:

```yaml
- name: Maak applicatiegebruiker aan
  ansible.builtin.user:
    name: "{{ app_user }}" # waarde komt uit variabelen
    state: present # user moet bestaan
```

## Variabelen

Variabelen zijn input voor je playbooks. Je kan ze zetten in inventory, `group_vars`, `host_vars`, role defaults, extra vars enzovoort.

Voorbeeld `group_vars/web.yml`:

```yaml
app_user: webapp # gebruiker voor webapplicatie
app_port: 8080 # poort waarop de applicatie luistert
packages:
  - nginx # reverse proxy package
  - git # nodig om code op te halen
```

Gebruik in een playbook:

```yaml
- name: Installeer packages voor webserver
  ansible.builtin.package:
    name: "{{ packages }}" # lijst met packages uit group_vars
    state: present # alle packages moeten geïnstalleerd zijn
```

## Variable precedence

Als dezelfde variabele op meerdere plaatsen staat, moet Ansible kiezen welke wint. De vuistregel: **specifieker wint meestal van algemener** en **extra vars winnen heel sterk**.

Voor het mondeling hoef je niet elke precedence-regel uit het hoofd te kennen, maar je moet het principe begrijpen.

Voorbeeld:

- `group_vars/all.yml`: `app_port: 80`
- `group_vars/web.yml`: `app_port: 8080`
- `host_vars/web01.yml`: `app_port: 9090`

Voor `web01` wint meestal de hostspecifieke waarde `9090`, omdat die specifieker is.

## Facts

Facts zijn gegevens die Ansible verzamelt over een host: OS, IP-adressen, CPU, geheugen, distributie, hostname enzovoort.

```bash
ansible web01 -i inventories/lab/hosts.yml -m ansible.builtin.setup # verzamelt en toont facts van web01
```

Voorbeeld in een playbook:

```yaml
- name: Installeer Apache op Debian-family hosts
  ansible.builtin.package:
    name: apache2 # Debian/Ubuntu package naam
    state: present # package moet aanwezig zijn
  when: ansible_facts['os_family'] == 'Debian' # alleen uitvoeren op Debian-family systemen
```

Facts zijn handig, maar niet gratis. Fact gathering kost tijd en kan veel output geven. Gebruik facts bewust.

## Magic variables

Magic variables zijn speciale variabelen die Ansible zelf beschikbaar maakt.

| Magic variable | Betekenis |
|---|---|
| `inventory_hostname` | naam van de host zoals in inventory |
| `groups` | alle inventorygroepen |
| `hostvars` | variabelen/facts van andere hosts |
| `play_hosts` | hosts in de huidige play |

Voorbeeld:

```yaml
- name: Toon inventorynaam
  ansible.builtin.debug:
    msg: "Deze host heet {{ inventory_hostname }} in de inventory" # gebruikt magic variable
```

## Register

Met `register` bewaar je output van een taak in een variabele. Daarna kan je die output gebruiken voor debug, conditions of vervolgacties.

```yaml
- name: Controleer of nginx actief is
  ansible.builtin.command: systemctl is-active nginx # vraagt status op via systemctl
  register: nginx_status # bewaart stdout, stderr, rc en metadata
  changed_when: false # status check verandert niets aan de server
  failed_when: nginx_status.rc not in [0, 3] # rc 0=actief, rc 3=inactief; andere codes zijn fout

- name: Toon nginx-status
  ansible.builtin.debug:
    msg: "Nginx status is {{ nginx_status.stdout }}" # gebruikt output van vorige taak
```

## Typische valkuil

Veel studenten registreren output, maar weten daarna niet wat erin zit. Gebruik tijdelijk `debug` om de structuur te bekijken.

```yaml
- name: Debug volledige geregistreerde variabele
  ansible.builtin.debug:
    var: nginx_status # toont de volledige datastructuur voor studie/debug
```

## Typische examenvraag

**Vraag:** Waarom zijn variables en facts belangrijk voor IaC?

**Sterk antwoord:**

Variables en facts maken playbooks herbruikbaar en contextbewust. De taken blijven dezelfde, maar de input kan verschillen per host, groep of omgeving. Facts geven informatie over de actuele host, zoals OS-family, waardoor je taken conditioneel correct kan uitvoeren. Register laat toe om output van een taak te bewaren en daarop verder te beslissen.

## Mini-checklist

- Kan je uitleggen waarom hardcoding slecht is?
- Kan je `group_vars` en `host_vars` uitleggen?
- Kan je facts uitleggen met een OS-voorbeeld?
- Kan je `register` gebruiken en uitleggen wat `stdout`, `stderr` en `rc` zijn?
