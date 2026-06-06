# 07 - Secrets en Vault

## Kernzin voor het mondeling

**Secrets horen niet plaintext in Git; Ansible Vault versleutelt gevoelige waarden of bestanden zodat je repo toch de bron van waarheid kan blijven zonder wachtwoorden bloot te leggen.**

## Wat is een secret?

Een secret is informatie waarmee iemand toegang kan krijgen of schade kan doen als die uitlekt.

Voorbeelden:

- wachtwoorden;
- API-tokens;
- private keys;
- database credentials;
- cloud tokens;
- Vault password zelf.

Geen secret: een gewone poort zoals `8080`, een username zonder wachtwoord, een packagenaam.

## Wat is Ansible Vault?

Ansible Vault is een manier om gevoelige Ansible-data te versleutelen. Het is geen volledige secret manager zoals HashiCorp Vault, maar het is wel nuttig voor versleutelde variabelen of bestanden in een Ansible-repo.

## Encrypt string

```bash
ansible-vault encrypt_string 'SuperSecret123' --name 'db_password' # maakt een versleutelde variabele db_password
```

Dat geeft output die je in YAML kan plakken.

```yaml
db_password: !vault | # Ansible herkent dit als vaulted waarde
  $ANSIBLE_VAULT;1.1;AES256
  613861... # versleutelde inhoud, niet leesbaar zonder vault password
```

## Encrypt file

```bash
ansible-vault create group_vars/prod/vault.yml # maakt nieuw versleuteld variabelenbestand
ansible-vault edit group_vars/prod/vault.yml # bewerkt versleuteld bestand veilig
ansible-vault view group_vars/prod/vault.yml # toont inhoud na wachtwoordprompt
```

## Playbook uitvoeren met Vault

```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/site.yml --ask-vault-pass # vraagt Vault-wachtwoord interactief
```

Met password file, alleen als die veilig buiten Git staat:

```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/site.yml --vault-password-file .vault_pass # gebruikt lokaal wachtwoordbestand dat in .gitignore staat
```

## Vault is maar zo sterk als het wachtwoordbeheer

Als je `.vault_pass` commit, heeft Vault weinig zin. Als je het wachtwoord in een screenshot toont, ook niet. Daarom:

- commit nooit vault password files;
- toon geen secrets in logs;
- gebruik `no_log: true` bij gevoelige taken;
- documenteer hoe secrets gebruikt worden;
- gebruik in teams liever AWX credentials of een externe secret manager waar nodig.

```yaml
- name: Maak database user aan
  community.mysql.mysql_user:
    name: "{{ db_user }}" # databasegebruiker
    password: "{{ db_password }}" # vaulted wachtwoord
    priv: '*.*:ALL' # rechten voor voorbeeld
    state: present # user moet bestaan
  no_log: true # voorkomt dat secret in output verschijnt
```

## Typische examenvraag

**Vraag:** Wat lost Ansible Vault op en wat niet?

**Sterk antwoord:**

Vault zorgt dat gevoelige variabelen of bestanden versleuteld in de repo kunnen staan. Het voorkomt plaintext secrets in Git. Maar Vault is geen volledige secret-managementoplossing en is maar zo veilig als het vault password en je werkwijze. Als het vault password uitlekt of je secrets in logs toont, is het probleem niet opgelost.

