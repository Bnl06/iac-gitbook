# Cheatsheet: termen en ezelsbruggen

## Superkorte definities

| Term | Onthouden als |
|---|---|
| IaC | infra als software behandelen |
| Repo | bron van waarheid |
| Inventory | waar draait Ansible? |
| Playbook | welke stappen/rollen worden uitgevoerd? |
| Role | herbruikbaar bouwblok |
| Variable | input die jij kiest |
| Fact | info die Ansible ontdekt |
| Register | output bewaren |
| Handler | alleen actie bij wijziging |
| Template | dynamische config genereren |
| Vault | secrets versleutelen |
| Dynamic inventory | hosts uit API/metadata |
| AWX | gecontroleerde uitvoering |
| Verify | bewijs dat het werkt |
| Drift | huidige toestand wijkt af |

## Ezelsbruggen

### IAC

**I** = Infrastructure  
**A** = As  
**C** = Code  
Maar voor mondeling: **In Git, Automatisch, Controleerbaar**.

### Ansible run

**W-W-W-B**:

- **Waar**: inventory.
- **Wat**: playbook/roles.
- **Waarmee**: credentials/vars.
- **Bewijs**: output/verify.

### Goede taak

Een goede taak is:

- duidelijk benoemd;
- gebruikt een module;
- heeft gewenste state;
- gebruikt variabelen;
- is idempotent;
- lekt geen secrets.

## Zinnen die je kan gebruiken

> Mijn repo is de bron van waarheid: inventory, variabelen, playbooks, roles en dependencies staan daarin zodat het project vanaf een fresh clone reproduceerbaar is.

> Idempotentie betekent dat run 2 niet opnieuw onnodige wijzigingen mag doen. Als dat wel gebeurt, is dat een signaal dat mijn taak niet goed gemodelleerd is.

> Ik gebruik handlers omdat een service restart alleen nodig is wanneer een configuratiebestand echt gewijzigd is.

> Ik gebruik Vault omdat secrets niet plaintext in Git mogen staan, maar ik moet ook vermijden dat secrets via logs of screenshots lekken.

> AWX maakt uitvoering beheerbaar, maar repareert geen slechte Ansible-code.
