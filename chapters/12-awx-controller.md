# 12 - AWX / Automation Controller

## Kernzin voor het mondeling

**AWX/Automation Controller vervangt je Ansible-project niet, maar operationaliseert het: gecontroleerd starten, credentials beheren, inventories koppelen, repo syncen en job output bewaren.**

## Waarom AWX?

De CLI is prima om te leren en lokaal te testen. In een teamomgeving ontstaan andere vragen:

- Wie mag een job starten?
- Welke inventory wordt gebruikt?
- Welke credentials worden gekoppeld?
- Welke versie van de repo draait?
- Waar staat de output?
- Wie kan achteraf controleren wat er gebeurd is?

AWX maakt die keuzes expliciet.

## CLI naar AWX-objecten

CLI-run:

```bash
ansible-playbook -i inventories/demo/hosts.yml playbooks/demo.yml --ask-vault-pass # start playbook lokaal met inventory en Vault-prompt
```

In AWX wordt dit opgesplitst:

| CLI-deel | AWX-object |
|---|---|
| Git repo | Project |
| `-i inventories/demo/hosts.yml` | Inventory of Inventory Source |
| SSH/WinRM/API/Vault toegang | Credentials |
| `playbooks/demo.yml` + opties | Job Template |
| concrete uitvoering | Job |
| terminaloutput | Job Output |

## Project

Een Project in AWX verwijst meestal naar je Git-repo. Project sync bepaalt welke versie AWX lokaal heeft.

Belangrijke zin:

> Als AWX niet gesynct is, draait het misschien niet de repo-versie die jij denkt.

## Execution Environment versus requirements

- **Execution Environment**: runtime waarin Ansible draait, inclusief Python, ansible-core en tools.
- **requirements.yml**: Ansible-content van je project, zoals collections en roles.

Ze vullen elkaar aan. Een project met ontbrekende dependencies blijft fout, ook in AWX.

## Inventory en Inventory Source

Inventory bepaalt waar een job draait. Dat kan manueel, uit Git of dynamisch uit cloud/virtualisatieplatformen.

Goede volgorde:

1. Project sync.
2. Inventory sync.
3. Hosts/groepen controleren.
4. Credentials controleren.
5. Job Template starten.
6. Job output lezen.
7. Resultaat verifiëren.

## Credentials

| Secret/toegang | AWX credential type |
|---|---|
| Private Git repo token | Source Control Credential |
| SSH private key | Machine Credential |
| Linux sudo password | Machine Credential |
| Windows user/password | Machine Credential |
| Vault password | Vault Credential |
| Cloud API token | Cloud/custom credential |
| API-token in playbook | Custom credential of vaulted variable |

Credentials zijn geen vars-file. Ze zijn bedoeld om toegang veilig beschikbaar te maken voor jobs.

## Job Template

Een Job Template is een opgeslagen runconfiguratie:

- project;
- playbook;
- inventory;
- credentials;
- extra vars indien toegestaan;
- limit/tags/check mode;
- survey eventueel.

Belangrijke zin:

> Job Template is het recept. Job is de uitvoering. Job Output is het bewijs.

## AWX repareert geen rommelige repo

Als je repo hardcoded secrets, ontbrekende requirements, slechte roles of niet-idempotente taken bevat, maakt AWX dat niet automatisch goed. AWX maakt uitvoering beheerbaar, maar de inhoud moet nog steeds correct zijn.

## Typische examenvraag

**Vraag:** Wat voegt AWX toe bovenop Ansible CLI?

**Sterk antwoord:**

AWX organiseert en beveiligt Ansible-uitvoering. De CLI voert een playbook uit, maar AWX maakt expliciet welke repo-versie, inventory, credentials en job template gebruikt worden. Het bewaart job output en maakt uitvoering deelbaar in een teamcontext. Maar AWX vervangt de repo niet: playbooks, roles en dependencies moeten nog steeds goed opgebouwd zijn.

