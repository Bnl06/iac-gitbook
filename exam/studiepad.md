# Studiepad voor het mondeling

## Hoe je dit vak best instudeert

Voor het mondeling moet je niet alleen definities kunnen geven. Je moet kunnen uitleggen alsof je een junior collega overtuigt waarom deze werkwijze logisch is.

## Fase 1 - Begrippen vastzetten

Studeer eerst deze kernbegrippen:

| Begrip | Moet je kunnen zeggen |
|---|---|
| IaC | infrastructuur beheren via machineleesbare definities |
| Desired state | hoe het volgens de code moet zijn |
| Current state | hoe het nu echt is |
| Idempotentie | tweede run verandert niets als input gelijk blijft |
| Inventory | waar Ansible op draait |
| Variables | data/input voor playbooks |
| Facts | gegevens die Ansible over hosts verzamelt |
| Register | output van een taak bewaren |
| Handler | taak die alleen draait na `notify` bij wijziging |
| Role | herbruikbaar bouwblok |
| Vault | versleuteling van secrets in Ansible |
| Dynamic inventory | inventory uit API/metadata |
| AWX | beheerde uitvoering van Ansible-jobs |

## Fase 2 - Per hoofdstuk 3 lagen

Gebruik deze volgorde:

1. **Definitie**: wat is het?
2. **Waarom**: welk probleem lost het op?
3. **Voorbeeld**: hoe ziet het eruit in Ansible of in je repo?

Voorbeeld bij handlers:

- Definitie: handler is een taak die door `notify` wordt getriggerd.
- Waarom: onnodige service restarts vermijden.
- Voorbeeld: template verandert nginx-config en notifyt `Herstart nginx`.

## Fase 3 - Run 1 en run 2 kunnen uitleggen

Voor Magnum Opus is dit superbelangrijk:

```bash
ansible-playbook -i inventories/lab/hosts.yml playbooks/site.yml --ask-vault-pass # run 1 brengt omgeving naar gewenste toestand
ansible-playbook -i inventories/lab/hosts.yml playbooks/site.yml --ask-vault-pass # run 2 toont idempotentie, normaal minder/geen changed
```

Op het mondeling zeg je:

> Run 1 configureert de omgeving. Run 2 is mijn bewijs dat de gewenste toestand stabiel is. Als run 2 opnieuw veel `changed` toont, moet ik onderzoeken welke taken niet idempotent zijn.

## Fase 4 - Magnum Opus linken aan theorie

Bij elk projectonderdeel moet je kunnen verwijzen naar theorie:

| Projectonderdeel | Theorie die je kan noemen |
|---|---|
| Repo-structuur | projectarchitectuur, Git als bron van waarheid |
| Roles | herbruikbaarheid, thin playbooks |
| Vault | secrets niet plaintext in Git |
| Templates | config dynamisch maken |
| Handlers | alleen restart bij wijziging |
| Verify tasks | bewijs en idempotentie |
| README | reproduceerbaarheid vanaf fresh clone |
| Dynamic inventory | cloudmetadata en actuele hosts |
| AWX | operationalisatie en job output |

## Laatste herhaling

Gebruik de techniek: **zeg het luidop in 60 seconden**. Als je een thema niet binnen 60 seconden helder kan uitleggen, ken je het nog niet mondeling genoeg.
