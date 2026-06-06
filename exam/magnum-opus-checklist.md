# Magnum Opus checklist

## Repo-check

- [ ] Repo is private.
- [ ] Lector heeft toegang.
- [ ] Finale code staat op `main`.
- [ ] Tag `submission-v1` bestaat en is gepusht.
- [ ] Geen plaintext secrets in Git.
- [ ] `.gitignore` bevat vault password files, keys, logs en editorrommel.

```bash
git status # controleer dat er geen vergeten wijzigingen zijn
git log --oneline -5 # controleer de laatste commits
git tag # controleer of submission-v1 bestaat
git push origin main # push finale code
git push origin submission-v1 # push de submit-tag
```

## Ansible-check

- [ ] `ansible.cfg` is aanwezig.
- [ ] `requirements.yml` is aanwezig.
- [ ] Inventory is duidelijk.
- [ ] Variabelen staan in `group_vars`/`host_vars` of role defaults.
- [ ] Roles worden gebruikt waar logisch.
- [ ] Templates worden gebruikt voor dynamische config.
- [ ] Handlers worden gebruikt voor restarts.
- [ ] Modules worden verkozen boven shell/command.
- [ ] Shell/command heeft motivatie plus `changed_when`/`failed_when` waar nodig.

## Idempotentie-check

- [ ] Run 1 werkt.
- [ ] Run 2 werkt.
- [ ] Run 2 toont weinig of geen onverwachte `changed`.
- [ ] Je kan uitleggen welke taken bewust `changed` blijven, als dat zo is.

```bash
ansible-playbook -i inventories/lab/hosts.yml playbooks/site.yml --ask-vault-pass # eerste run configureert de omgeving
ansible-playbook -i inventories/lab/hosts.yml playbooks/site.yml --ask-vault-pass # tweede run bewijst idempotentie
```

## Security-check

- [ ] Vault of veilig alternatief gebruikt.
- [ ] Geen secrets in screenshots/logs.
- [ ] `no_log: true` bij gevoelige taken.
- [ ] Alleen noodzakelijke poorten open.
- [ ] TLS voorzien als publiek HTTP(S) relevant is.
- [ ] Securitykeuzes staan in documentatie.

## README-check

README beantwoordt:

- [ ] Wat doet het project?
- [ ] Welke dependencies zijn nodig?
- [ ] Hoe installeer je collections/roles?
- [ ] Welke inventory gebruik je?
- [ ] Hoe voer je run 1 en run 2 uit?
- [ ] Hoe gebruik je Vault?
- [ ] Hoe verifieer je het resultaat?
- [ ] Welke beperkingen zijn er?

## Mondelinge voorbereiding

Voor elk belangrijk projectonderdeel moet je kunnen zeggen:

1. Wat het doet.
2. Waarom je die keuze maakte.
3. Welke theorie erbij hoort.
4. Hoe je bewijst dat het werkt.
