# Mondelinge voorbeeldvragen

## IaC basis

### Wat is Infrastructure as Code?

Infrastructure as Code is infrastructuur beheren en uitrollen via machineleesbare definitiebestanden. Je behandelt infrastructuur als software: versiebeheer, review, reproduceerbare uitvoering en bewijs via output. Het doel is niet gewoon automatiseren, maar infrastructuur voorspelbaar en controleerbaar maken.

### Waarom is een random script niet hetzelfde als IaC?

Een script kan automatiseren, maar zonder structuur kan het hardcoded waarden, duplicatie en onduidelijke foutafhandeling bevatten. IaC vraagt dat de gewenste toestand, inventory, variabelen, dependencies en documentatie reproduceerbaar in een repo staan.

## Ansible project

### Waarom gebruik je `requirements.yml`?

Omdat dependencies reproduceerbaar moeten zijn. Een project mag niet alleen werken omdat mijn laptop toevallig een collection heeft. Met `requirements.yml` kan iemand vanaf een fresh clone de nodige collections/roles installeren.

```bash
ansible-galaxy collection install -r requirements.yml # installeert de collections die het project nodig heeft
```

### Waarom modules boven `shell` of `command`?

Modules kennen meestal gewenste toestand en rapporteren beter `ok`, `changed` en `failed`. Shell of command voert gewoon een commando uit en is vaak minder idempotent. Je mag shell gebruiken als er geen goede module bestaat, maar dan moet je `changed_when` en `failed_when` bewust modelleren.

## Variables, facts, register

### Wat is het verschil tussen variables en facts?

Variables zijn input die jij definieert, bijvoorbeeld poorten, usernames en packagelijsten. Facts zijn gegevens die Ansible verzamelt over de host, zoals OS-family, IP-adressen en geheugen. Variables zeggen wat jij wil; facts beschrijven de context waarin je werkt.

### Waarvoor gebruik je register?

Met `register` bewaar je output van een taak, bijvoorbeeld stdout, stderr en return code. Daarna kan je die output tonen, controleren of gebruiken in een condition.

## Task control

### Waarom gebruik je loops?

Loops vermijden copy-paste. Eén taak kan meerdere items verwerken, waardoor het playbook korter en consistenter wordt.

### Waarom is `ignore_errors` gevaarlijk?

Omdat het fouten kan verbergen. Soms lijkt de playbook succesvol, terwijl een belangrijke taak faalde. Beter is exact definiëren welke return codes aanvaardbaar zijn met `failed_when`.

## Templates, handlers en roles

### Waarom gebruik je templates?

Templates maken configuratiebestanden dynamisch. De structuur van het bestand staat in de template, terwijl waarden zoals poorten, namen en paden uit variabelen komen.

### Waarom gebruik je handlers?

Handlers voeren acties alleen uit als een taak iets veranderde. Dat is ideaal voor service restarts na configwijzigingen en helpt idempotentie.

### Wat is een role?

Een role is een herbruikbaar bouwblok waarin taken, defaults, handlers, templates en bestanden samen zitten. Daardoor blijft het playbook dun en kan je dezelfde role in meerdere omgevingen gebruiken.

## Secrets

### Wat lost Ansible Vault op?

Vault voorkomt dat secrets plaintext in Git staan. Je kan variabelen of bestanden versleutelen. Maar Vault is geen wondermiddel: het vault password moet veilig beheerd worden en secrets mogen niet in logs verschijnen.

## Windows

### Waarom is Windows anders met Ansible?

Windows wordt meestal via WinRM en PowerShell-modules beheerd, niet via SSH en Python zoals Linux. Daardoor gebruik je andere connection variables en Windows-specifieke modules zoals `win_service`, `win_copy` en Chocolatey modules.

## API's

### Hoe maak je API-werk idempotent?

Niet blind elke run POST doen. Eerst GET je de huidige toestand, dan vergelijk je met de gewenste toestand en pas je alleen aan als het nodig is. Daarna verify je het resultaat.

## Cloud

### Waarom dynamic inventory?

Cloudhosts veranderen snel. Dynamic inventory haalt actuele hosts en metadata op via de cloud-API. Daardoor kan je groeperen op labels zoals `env=prod` en `role=web` zonder handmatig inventory bij te werken.

## Network automation

### Waarom is verify zo belangrijk bij netwerkautomation?

Omdat netwerkfouten grote impact kunnen hebben. Na een wijziging moet je bewijzen dat de gewenste toestand klopt en dat bereikbaarheid niet kapot is. Verify is dus niet extra, maar deel van de workflow.

## AWX

### Wat is het verschil tussen Job Template en Job?

Een Job Template is het recept: project, playbook, inventory, credentials en opties. Een Job is één concrete uitvoering van dat recept. De Job Output is het bewijs van wat er gebeurd is.

## PXE

### Waarom is PXE geen volledige provisioning?

PXE is de netwerkbootfase. Volledige provisioning omvat ook OS-installatie, identiteit, netwerk, credentials, applicaties, post-configuratie en validatie. Ansible past vooral in post-configuratie en verify.
