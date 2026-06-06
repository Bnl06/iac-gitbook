<<<<<<< HEAD
# 🌿 Infrastructure as Code - Zensical Studiegids

Welkom in je **niet-standaard GitBook** voor het mondeling examen Infrastructure as Code. Dit is gemaakt als een rustige, duidelijke studiegids: eerst begrijpen, dan kunnen uitleggen, daarna kunnen toepassen.

> **Doel van deze gids**  
> Jij moet mondeling kunnen aantonen dat je IaC begrijpt: niet alleen commando's vanbuiten leren, maar kunnen uitleggen waarom je iets doet, wat het voordeel is en waar de valkuilen zitten.

## Hoe studeer je dit?

Gebruik per hoofdstuk dezelfde volgorde:

1. Lees de **kernzin**. Die moet je bijna letterlijk kunnen zeggen op het mondeling.
2. Lees de **uitleg alsof je het aan een klasgenoot uitlegt**.
3. Bekijk de **voorbeelden** en let op de commentaarregels bij Ansible-code.
4. Oefen met de **mondelinge vragen** achteraan.
5. Herhaal de **checklist** voor Magnum Opus en examen.

## Zensical studiemethode

- **Zen**: rustig overzicht, geen chaos.
- **Practical**: altijd koppelen aan een voorbeeld.
- **Oral-first**: elk thema bevat zinnen die je op het mondeling kan gebruiken.
- **No blind copy-paste**: code is er om te begrijpen, niet om blind te plakken.

## Repo lokaal bekijken

```bash
git clone <jouw-repo-url> # haalt je GitBook-repo lokaal binnen
cd iac-zensical-gitbook # ga naar de repo-map
npm install # installeert de klassieke GitBook CLI dependencies
npm run serve # start een lokale GitBook preview
```

Publiceren kan via GitBook door deze repository te koppelen, of via GitHub door de Markdown rechtstreeks te bekijken.

## Belangrijke afspraak over code

Wanneer er Ansible-code of CLI-code staat, staat er uitleg bij met `# uitleg`. Bijvoorbeeld:

```bash
ansible-playbook -i inventories/lab/hosts.yml playbooks/site.yml # voert site.yml uit tegen de lab-inventory
```

=======
# iac-gitbook
>>>>>>> b50cd88e700afb29b59a66213157b869df816aa9
