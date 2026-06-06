# 08 - Windows, WinRM en Chocolatey

## Kernzin voor het mondeling

**Windows beheer met Ansible gebeurt niet via SSH zoals bij Linux, maar meestal via WinRM; Windows-modules en Chocolatey maken beheer van configuratie en software idempotent mogelijk.**

## Wat verandert bij Windows?

Bij Linux gebruikt Ansible meestal SSH en Python. Bij Windows gebruikt Ansible meestal **WinRM** en PowerShell-gebaseerde modules. Je moet dus andere connection variables en modules gebruiken.

## Minimale WinRM-variabelen

Voorbeeld `host_vars/win01.yml`:

```yaml
ansible_host: 192.168.56.30 # IP of DNS-naam van Windows-host
ansible_connection: winrm # gebruik WinRM als transport
ansible_user: Administrator # Windows-gebruiker
ansible_password: "{{ vault_windows_admin_password }}" # wachtwoord uit Vault
ansible_winrm_transport: ntlm # transportmethode voor lab/AD-context
ansible_winrm_server_cert_validation: ignore # alleen lab: negeert certificaatvalidatie
```

## Connectivity proof

```bash
ansible win -i inventories/lab/hosts.yml -m ansible.windows.win_ping --ask-vault-pass # test WinRM-connectiviteit naar Windows hosts
```

## Windows facts

```yaml
- name: Verzamel Windows facts
  ansible.windows.setup: # Windows-equivalent voor facts verzamelen
```

```yaml
- name: Toon Windows OS
  ansible.builtin.debug:
    msg: "Windows versie: {{ ansible_facts['os_name'] }}" # gebruikt verzamelde fact
```

## Windows modules

| Linux-denken | Windows-module |
|---|---|
| `package` | `ansible.windows.win_package` of Chocolatey module |
| `service` | `ansible.windows.win_service` |
| `copy` | `ansible.windows.win_copy` |
| `template` | `ansible.windows.win_template` |
| `file` | `ansible.windows.win_file` |
| command | `ansible.windows.win_command` |
| shell | `ansible.windows.win_shell` |

## Chocolatey

Chocolatey is een package manager voor Windows. Het maakt software-installaties meer automatisch en herhaalbaar.

```yaml
- name: Installeer Git via Chocolatey
  chocolatey.chocolatey.win_chocolatey:
    name: git # Chocolatey package naam
    state: present # package moet geïnstalleerd zijn
```

Meerdere packages:

```yaml
- name: Installeer basissoftware op Windows
  chocolatey.chocolatey.win_chocolatey:
    name: "{{ item }}" # elk item is een Chocolatey package
    state: present # package moet aanwezig zijn
  loop:
    - git # versiebeheer
    - 7zip # archieftool
    - notepadplusplus # teksteditor
```

## Chocolatey-valkuil

Chocolatey 2.x heeft nieuwere vereisten dan oudere Windows-installaties. Je moet controleren of de Windows-host de juiste .NET/runtimevereisten heeft. Anders faalt packagebeheer al vóór je echte configuratie begint.

## Typische examenvraag

**Vraag:** Waarom is Windows met Ansible anders dan Linux?

**Sterk antwoord:**

Omdat Ansible voor Linux meestal SSH en Python gebruikt, terwijl Windows meestal via WinRM en PowerShell-modules beheerd wordt. Daardoor heb je andere connection variables, andere modules en soms extra voorbereiding nodig. Voor softwarebeheer is Chocolatey handig omdat het Windows packages idempotenter en reproduceerbaarder maakt.

