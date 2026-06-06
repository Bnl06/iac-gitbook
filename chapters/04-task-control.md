# 04 - Task control: loops, when en blocks

## Kernzin voor het mondeling

**Loops vermijden copy-paste, `when` maakt taken contextbewust en blocks geven foutafhandeling en guardrails aan groepen taken.**

## Waarom task control bestaat

Zonder task control krijg je veel dubbele taken. Je installeert bijvoorbeeld drie packages met drie bijna identieke taken. Dat is foutgevoelig. Als je later één taak aanpast en de andere vergeet, krijg je copy-paste drift.

## Loops

Een loop herhaalt één taak over meerdere items.

```yaml
- name: Installeer basispackages
  ansible.builtin.package:
    name: "{{ item }}" # item is telkens één package uit de loop
    state: present # package moet geïnstalleerd zijn
  loop:
    - vim # teksteditor
    - git # versiebeheer
    - curl # HTTP-client voor testen/API-calls
```

Loop over list of dictionaries:

```yaml
- name: Maak applicatiegebruikers aan
  ansible.builtin.user:
    name: "{{ item.name }}" # username uit dictionary
    shell: "{{ item.shell }}" # shell uit dictionary
    groups: "{{ item.groups }}" # groepen uit dictionary
    state: present # user moet bestaan
  loop:
    - { name: "deploy", shell: "/bin/bash", groups: "www-data" } # deploy-user
    - { name: "backup", shell: "/usr/sbin/nologin", groups: "backup" } # service-user zonder login
```

## When

`when` bepaalt of een taak uitgevoerd wordt.

```yaml
- name: Installeer nginx op Debian-family
  ansible.builtin.package:
    name: nginx # package naam
    state: present # moet aanwezig zijn
  when: ansible_facts['os_family'] == 'Debian' # alleen Debian/Ubuntu
```

Meerdere voorwaarden:

```yaml
- name: Start applicatie alleen in productie
  ansible.builtin.service:
    name: myapp # servicenaam
    state: started # service moet draaien
  when:
    - env == 'prod' # alleen productieomgeving
    - myapp_enabled | bool # alleen als feature aan staat
```

When met loop:

```yaml
- name: Installeer alleen packages die enabled zijn
  ansible.builtin.package:
    name: "{{ item.name }}" # package naam uit item
    state: present # moet geïnstalleerd zijn
  loop:
    - { name: "nginx", enabled: true } # installeren
    - { name: "telnet", enabled: false } # overslaan
  when: item.enabled | bool # voert task alleen uit voor enabled items
```

## Blocks

Een block groepeert taken. Je kan er `rescue` en `always` aan koppelen.

```yaml
- name: Deploy applicatie met rollback-achtige foutafhandeling
  block:
    - name: Kopieer config
      ansible.builtin.template:
        src: app.conf.j2 # templatebron
        dest: /etc/myapp/app.conf # doelpad op server
        mode: '0644' # veilige leesrechten

    - name: Herstart applicatie
      ansible.builtin.service:
        name: myapp # servicenaam
        state: restarted # herstart na configwijziging

  rescue:
    - name: Meld deploy-fout
      ansible.builtin.debug:
        msg: "Deploy is mislukt; controleer config en logs" # duidelijke foutmelding

  always:
    - name: Toon einde van deployblok
      ansible.builtin.debug:
        msg: "Deployblok is afgewerkt" # loopt altijd, ook bij fout
```

## Retries, delay en until

Sommige problemen zijn tijdelijk. Bijvoorbeeld: een service heeft enkele seconden nodig om op te starten.

```yaml
- name: Wacht tot webapp antwoordt
  ansible.builtin.uri:
    url: "http://localhost:8080/health" # health endpoint
    status_code: 200 # verwachte HTTP-status
  register: healthcheck # bewaart resultaat van elke poging
  retries: 5 # maximaal vijf pogingen
  delay: 3 # drie seconden wachten tussen pogingen
  until: healthcheck.status == 200 # stoppen zodra status 200 is
```

## Waarom `ignore_errors` meestal slecht is

`ignore_errors` laat een fout verdwijnen uit de flow. Soms is dat nodig, maar vaak maskeert het echte problemen. Beter is expliciet modelleren welke fouten aanvaardbaar zijn met `failed_when`.

```yaml
- name: Controleer of user bestaat
  ansible.builtin.command: id appuser # zoekt appuser op
  register: appuser_check # bewaart exitcode en output
  changed_when: false # check verandert niets
  failed_when: appuser_check.rc not in [0, 1] # rc 1 betekent: user bestaat niet, dat is geen technische fout
```

## Typische examenvraag

**Vraag:** Wanneer gebruik je loops, when en blocks?

**Sterk antwoord:**

Loops gebruik ik om herhaalde taken te vermijden. `when` gebruik ik om taken alleen uit te voeren wanneer ze relevant zijn, bijvoorbeeld op basis van OS-facts of omgeving. Blocks gebruik ik om taken logisch te groeperen en foutafhandeling toe te voegen met `rescue` en `always`. Samen maken ze playbooks onderhoudbaar, correcter en minder copy-paste gevoelig.

## Mini-checklist

- Kan je een loop over een lijst uitleggen?
- Kan je een loop over dictionaries uitleggen?
- Kan je `when` met facts uitleggen?
- Kan je uitleggen waarom `ignore_errors` gevaarlijk kan zijn?
