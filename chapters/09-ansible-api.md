# 09 - Ansible en API's

## Kernzin voor het mondeling

**Met `ansible.builtin.uri` kan Ansible REST-API's aanspreken, maar je moet statuscodes, authenticatie, JSON-body's, idempotentie en secret-leaks bewust modelleren.**

## Waarom API's in Ansible?

Niet alles is een server waarop je SSH of WinRM gebruikt. Veel platformen werken via API's: cloud, monitoring, GitHub, NetBox, Keycloak, firewalls, SaaS-platformen. Ansible kan die platformen aansturen via modules of via de algemene `uri`-module.

## GET-request

```yaml
- name: Haal huidige API-status op
  ansible.builtin.uri:
    url: "https://api.example.local/status" # endpoint dat status teruggeeft
    method: GET # HTTP GET vraagt data op
    return_content: true # bewaar response body in result.content
    status_code: 200 # verwachte statuscode
  register: api_status # bewaart API-response
```

Debug:

```yaml
- name: Toon API-content
  ansible.builtin.debug:
    var: api_status.json # toont JSON als Ansible die kon parsen
```

## Headers en authenticatie

```yaml
- name: Vraag beveiligde resource op
  ansible.builtin.uri:
    url: "https://api.example.local/v1/projects" # API-endpoint
    method: GET # data ophalen
    headers:
      Authorization: "Bearer {{ demo_api_token }}" # token uit Vault of credential
      Accept: "application/json" # verwacht JSON terug
    status_code: 200 # alleen 200 is ok
  register: projects_response # bewaart resultaat
  no_log: true # voorkomt token in output/logs
```

## POST met JSON-body

```yaml
- name: Maak project aan via API
  ansible.builtin.uri:
    url: "https://api.example.local/v1/projects" # endpoint voor projecten
    method: POST # maakt nieuwe resource aan
    headers:
      Authorization: "Bearer {{ demo_api_token }}" # API-token
      Content-Type: "application/json" # body is JSON
    body_format: json # Ansible zet body om naar JSON
    body:
      name: "{{ project_name }}" # projectnaam uit variabele
      owner: "{{ project_owner }}" # eigenaar uit variabele
    status_code: [200, 201] # accepteer OK of Created
  register: create_project # bewaart API-resultaat
  no_log: true # bescherm token en mogelijke gevoelige output
```

## Idempotentie met API's

Een API-call is niet automatisch idempotent. Als je elke run opnieuw `POST /projects` doet, maak je misschien telkens een nieuw project. Betere flow:

1. GET huidige toestand.
2. Vergelijk met gewenste toestand.
3. Alleen POST/PUT/PATCH als nodig.
4. Verify na wijziging.

```yaml
- name: Haal bestaand project op
  ansible.builtin.uri:
    url: "https://api.example.local/v1/projects/{{ project_name }}" # zoekt project op naam
    method: GET # alleen lezen
    headers:
      Authorization: "Bearer {{ demo_api_token }}" # API-token
    status_code: [200, 404] # 404 betekent: bestaat nog niet
  register: project_get # bewaart huidige toestand
  no_log: true # bescherm token

- name: Maak project alleen als het niet bestaat
  ansible.builtin.uri:
    url: "https://api.example.local/v1/projects" # endpoint voor nieuwe projecten
    method: POST # resource aanmaken
    headers:
      Authorization: "Bearer {{ demo_api_token }}" # API-token
      Content-Type: "application/json" # JSON body
    body_format: json # body als JSON versturen
    body:
      name: "{{ project_name }}" # gewenste projectnaam
    status_code: 201 # verwacht Created
  when: project_get.status == 404 # alleen uitvoeren als project ontbreekt
  no_log: true # voorkom secret leaks
```

## Grenzen van uri

Gebruik `uri` wanneer er geen goede gespecialiseerde module is of wanneer je eenvoudige API-logica nodig hebt. Voor complexe platformen is een officiële collection/module vaak beter, omdat die idempotentie en datamodellen al beter ondersteunt.

## Typische examenvraag

**Vraag:** Waarom is API-automatisering met `uri` gevaarlijk als je niet nadenkt over idempotentie?

**Sterk antwoord:**

Omdat HTTP-methodes zoals POST vaak resources creëren. Als je die elke run blind uitvoert, kan je dubbele resources maken of ongewenste wijzigingen veroorzaken. Daarom moet je eerst de huidige toestand opvragen, vergelijken met de gewenste toestand en alleen aanpassen wanneer nodig. Ook moet je statuscodes en secrets correct behandelen.

