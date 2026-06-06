---
title: Idempotentie en mondeling verdedigen
description: Hoe je idempotentie bewijst en je Magnum Opus sterk verdedigt op het mondeling examen.
---

# Idempotentie en mondeling verdedigen

## Wat is idempotentie?

!!! abstract "Definitie"

    Idempotentie betekent dat je dezelfde automatisatie meerdere keren mag uitvoeren en dat het systeem na de eerste correcte run niet onnodig opnieuw wijzigt.

In Ansible betekent dat:

- Run 1 mag `changed` tonen, want de server wordt opgebouwd.
- Run 2 moet zo veel mogelijk `ok` tonen.
- Een perfecte run 2 toont `changed=0`.

## Jouw bewijs

In jouw tweede run staat:

```text
rocky1 : ok=26 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

Dat is zeer sterk omdat:

| Veld | Betekenis |
|---|---|
| `ok=26` | 26 taken waren al correct |
| `changed=0` | niets moest opnieuw aangepast worden |
| `failed=0` | niets is gefaald |
| `unreachable=0` | host was bereikbaar |

## Welke taken zijn idempotent?

| Taaktype | Waarom idempotent? |
|---|---|
| `dnf state: present` | installeert alleen als package ontbreekt |
| `file state: directory` | maakt map alleen als nodig |
| `template` | wijzigt alleen als inhoud verschilt |
| `firewalld state: enabled` | opent service alleen als die nog niet open is |
| `systemd_service state: started` | start alleen als service niet draait |
| `cron` | maakt/update alleen de beheerde cronjob |
| `command` met `creates` | voert command niet opnieuw uit als bestand bestaat |

## Belangrijk voorbeeld: Certbot met `creates`

```yaml
creates: "/etc/letsencrypt/live/{{ vaultwarden_domain }}/fullchain.pem" # voorkomt dat certificaat elke run opnieuw wordt aangevraagd
```

Zonder dit zou je run 2 waarschijnlijk toch `changed` geven en misschien Let's Encrypt rate limits raken.

## `changed_when: false`

Sommige commands of HTTP calls veranderen niet echt iets in de gewenste staat, maar Ansible zou ze anders als changed kunnen tonen.

Voorbeeld DuckDNS:

```yaml
changed_when: false # DuckDNS update/check moet niet als configuratiewijziging tellen
```

Voorbeeld Nginx test:

```yaml
changed_when: false # nginx -t test alleen configuratie en wijzigt niets
```

## Hoe verdedig je `changed_when: false`?

> Ik gebruik `changed_when: false` alleen bij taken die een controle uitvoeren of een externe update endpoint aanspreken waarvan ik de lokale systeemstaat niet als gewijzigd wil beschouwen. Voor echte configuratie gebruik ik modules zoals `template`, `file`, `dnf` en `cron`, die zelf idempotent zijn.

## Handlers en idempotentie

Handlers zorgen dat services alleen herstarten wanneer nodig.

```yaml
notify: Restart vaultwarden # triggert herstart alleen als template changed is
```

Als de template hetzelfde blijft, draait de handler niet. Dat is beter dan altijd restart uitvoeren.

## Mondelinge vragen met antwoorden

### Vraag: Waarom is run 2 belangrijker dan run 1?

> Run 1 bewijst dat mijn deployment werkt. Run 2 bewijst dat mijn deployment idempotent is. Bij Infrastructure as Code is dat belangrijk omdat je automatisatie vaak opnieuw uitvoert en dan geen onnodige wijzigingen of downtime wilt veroorzaken.

### Vraag: Wat zou een probleem zijn bij run 2?

> Als run 2 telkens `changed` toont, betekent dat dat een taak niet stabiel is. Dan moet ik controleren of ik een betere module kan gebruiken, `creates` nodig heb, of `changed_when` correct moet instellen bij checks.

### Vraag: Waarom gebruik je Ansible modules in plaats van overal shell commands?

> Modules kennen de gewenste staat. Bijvoorbeeld `dnf state: present` weet of een package al geïnstalleerd is. Een shell command is vaak minder idempotent en geeft sneller onnodig `changed`.

## Rubric-antwoord: uitstekend niveau

> Mijn project toont idempotentie door een eerste run en tweede run te bewaren. De eerste run bouwt de server op. De tweede run eindigt met `changed=0`, wat betekent dat de gewenste staat al bereikt was. Ik gebruik Ansible modules waar mogelijk, handlers voor service reloads, `creates` bij Certbot en `changed_when: false` bij checks zoals `nginx -t`. Daardoor kan het playbook veilig opnieuw uitgevoerd worden zonder onnodige wijzigingen.
