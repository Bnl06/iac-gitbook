# 10 - Cloud en dynamic inventory

## Kernzin voor het mondeling

**In cloudomgevingen verandert infrastructuur dynamisch, daarom past een static inventory slecht en gebruik je best dynamic inventory op basis van cloudmetadata zoals labels en tags.**

## Cloud is niet automatisch altijd juist

Cloud geeft flexibiliteit, API's en snelle provisioning, maar is niet altijd de beste keuze. Je moet nadenken over kost, security, compliance, netwerk, beheer en lifecycle.

## Waar past Ansible in cloud?

Ansible is sterk in:

- Day 1 configuratie: packages, users, config, services.
- Day 2 operations: updates, checks, drift-correctie, compliance.
- Orchestration: meerdere stappen combineren.
- Cloud API's aanspreken wanneer nodig.

Terraform/OpenTofu is vaak sterker voor pure provisioning van cloud resources. Ansible kan dat ook, maar wordt in veel ontwerpen vooral gebruikt nadat resources bestaan.

## Day 0, Day 1, Day 2

| Fase | Betekenis | Ansible past waar? |
|---|---|---|
| Day 0 | ontwerp/provisioning | soms, maar vaak Terraform/OpenTofu |
| Day 1 | initiële configuratie | zeer sterk |
| Day 2 | beheer, updates, drift, compliance | zeer sterk |

## Waarom static inventory slecht past

Cloudhosts kunnen snel ontstaan, verdwijnen of een ander IP krijgen. Als je handmatig `hosts.yml` bijwerkt, loopt je inventory achter.

Dynamic inventory vraagt aan de cloudprovider welke hosts bestaan.

```bash
ansible-inventory -i inventories/hcloud.yml --graph # toont dynamisch ontdekte hosts en groepen
ansible-inventory -i inventories/hcloud.yml --list # toont volledige JSON-output met hostvars
```

## Inventory plugin-config

Voorbeeldconcept:

```yaml
plugin: hetzner.hcloud.hcloud # dynamic inventory plugin voor Hetzner Cloud
api_token: "{{ lookup('env', 'HCLOUD_TOKEN') }}" # token komt uit environment, niet plaintext in Git
groups:
  web: "'web' in labels.role" # groepeer hosts met label role=web
  prod: "labels.env == 'prod'" # groepeer hosts met label env=prod
keyed_groups:
  - key: labels.env # maak groepen op basis van environment-label
    prefix: env # groep wordt bv. env_prod
```

Token zetten:

```bash
export HCLOUD_TOKEN='jouw-token-hier' # zet cloudtoken tijdelijk in shellomgeving
ansible-inventory -i inventories/hcloud.yml --graph # test of discovery werkt
```

## Metadata als ontwerpkeuze

Labels/tags zijn niet zomaar decoratie. Ze worden operationele input. Als je labels slecht kiest, wordt je dynamic inventory rommelig.

Goede labels:

```text
env=prod # omgeving
role=web # functie van host
managed_by=ansible # beheerbron
owner=team-iac # verantwoordelijke
```

## Day-2 run op dynamische groep

```bash
ansible-playbook -i inventories/hcloud.yml playbooks/baseline.yml --limit env_prod # voert baseline uit op dynamisch gevonden prod hosts
```

## Guardrails

Cloud automation kan snel veel raken. Gebruik daarom:

- `--limit` om scope te beperken;
- duidelijke labels;
- check mode waar zinvol;
- verify tasks;
- geen blind `all` op productie;
- veilige credentials.

```bash
ansible-playbook -i inventories/hcloud.yml playbooks/baseline.yml --check --limit env_lab # simuleert wijzigingen op labhosts
```

## Typische examenvraag

**Vraag:** Waarom gebruik je dynamic inventory in cloud?

**Sterk antwoord:**

Omdat cloudinfrastructuur dynamisch is. Hosts kunnen automatisch ontstaan, verdwijnen of van IP veranderen. Een static inventory raakt dan snel verouderd. Dynamic inventory haalt actuele hosts en metadata op via de cloud-API en kan groepen bouwen op basis van labels zoals environment en role. Daardoor kan Ansible gericht en actueel werken.

