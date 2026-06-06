# 05 - Templates en handlers

## Kernzin voor het mondeling

**Templates maken configuratiebestanden dynamisch met variabelen, en handlers voeren acties zoals service restarts alleen uit wanneer een taak echt iets veranderde.**

## Het probleem

Configuratiebestanden verschillen vaak per omgeving. Een webserver in lab luistert misschien op poort 8080, terwijl productie op 443 draait. Je wil niet voor elke host een volledig apart configbestand kopiëren.

Daarom gebruik je **Jinja2 templates**.

## Templatebestand

Voorbeeld `templates/nginx.conf.j2`:

```jinja2
server {
    listen {{ nginx_port }}; # poort komt uit variabelen
    server_name {{ nginx_server_name }}; # servernaam komt uit variabelen

    location / {
        proxy_pass http://127.0.0.1:{{ app_port }}; # backendpoort komt uit variabelen
    }
}
```

## Template-task

```yaml
- name: Plaats nginx-configuratie
  ansible.builtin.template:
    src: nginx.conf.j2 # Jinja2-template in templates-map
    dest: /etc/nginx/conf.d/myapp.conf # doelbestand op managed node
    owner: root # eigenaar van configbestand
    group: root # groep van configbestand
    mode: '0644' # leesbaar, niet schrijfbaar voor gewone users
  notify: Herstart nginx # roept handler alleen aan bij changed
```

## Handler

Een handler is een taak die pas draait wanneer hij genotified wordt. Typisch gebruik: service herstarten na configwijziging.

```yaml
handlers:
  - name: Herstart nginx
    ansible.builtin.service:
      name: nginx # servicenaam
      state: restarted # herstart service
```

Belangrijk: als de template niet wijzigt, wordt de handler niet uitgevoerd. Dat helpt idempotentie.

## Jinja2 essentials

Variabele:

```jinja2
{{ app_port }} # print de waarde van app_port
```

Condition:

```jinja2
{% if tls_enabled %} # als TLS aan staat
ssl_certificate {{ tls_cert_path }}; # certificaatpad uit variabele
ssl_certificate_key {{ tls_key_path }}; # sleutelpad uit variabele
{% endif %} # einde condition
```

Loop:

```jinja2
{% for domain in server_aliases %} # loop over alle aliassen
server_name {{ domain }}; # voeg elke alias toe
{% endfor %} # einde loop
```

## Welke module kies je?

| Situatie | Module |
|---|---|
| Statisch bestand kopiëren | `ansible.builtin.copy` |
| Config genereren met variabelen | `ansible.builtin.template` |
| Eén lijn beheren | `ansible.builtin.lineinfile` |
| Blok tekst beheren | `ansible.builtin.blockinfile` |

Voorbeeld `lineinfile`:

```yaml
- name: Zet PermitRootLogin uit
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config # SSH-configbestand
    regexp: '^PermitRootLogin' # bestaande lijn zoeken
    line: 'PermitRootLogin no' # gewenste lijn
  notify: Herstart ssh # SSH herstarten als config wijzigt
```

## Typische fouten

- Templatevariabele bestaat niet.
- Handlernaam in `notify` komt niet exact overeen.
- Handler staat op de verkeerde plaats.
- Service wordt altijd herstart in een gewone task in plaats van alleen bij wijziging.
- Secrets worden per ongeluk in templates of logs getoond.

## Typische examenvraag

**Vraag:** Waarom gebruik je handlers in plaats van gewoon altijd een service te herstarten?

**Sterk antwoord:**

Omdat een service restart alleen nodig is als de configuratie effectief gewijzigd is. Een handler wordt alleen getriggerd door een taak met `changed`. Daardoor blijft de playbook idempotent en vermijd je onnodige downtime of verstoring.

