# Bronnenindex van de aangeleverde theorie
Deze GitBook is opgebouwd op basis van de 13 aangeleverde PDF-theoriebundels. Hieronder staat per PDF een compacte index van de belangrijkste hoofdstuktitels die uit de bundels gehaald zijn.
## IaC Theorie 01 Inleiding
- Infrastructure as Code (IaC)
- Theorie 1: basisbegrippen en het idee achter IaC
- Wat gaan we vandaag doen?
- Doel: je kan uitleggen wat IaC is en wanneer je het inzet.
- Wat is Infrastructure as
- Code (IaC)?
- Definitie en kernidee: infrastructuur
- Wat is Infrastructure as Code (IaC)?
- IaC = infrastructuur beheren en uitrollen via machineleesbare definitiebestanden.
- Wat valt onder IaC (en wat niet)?
- Waarom gebruiken we
- IaC?
- Van manuele fouten naar
- Waarom zou je IaC gebruiken?
- Waarom dit “revolutionair” aanvoelt
- Maar het punt is: herhaalbaar werk
- Van scripts naar Infrastructure as Code
- Vroeger (en nog vaak): losse scripts om servers te installeren of te

## IaC Theorie 02 Ansible Projectstructuur
- Infrastructure as Code (IaC)
- Theorie 2: Ansible projectarchitectuur
- Waarom structuur geen luxe is
- Wat gaan we vandaag doen?
- Ansible. Je start jobs via een interface…AAP (Ansible Automation Platform)
- Minimum Ansible projectstructuur
- Volledige Ansible projectstructuur
- Componenten uitgelegd
- Componenten – “minimum” vs “volledig”
- Hoe installeren:
- Collections: ansible-galaxy collection install -r requirements.yml
- Roles: ansible-galaxy role install -r requirements.yml
- Zonder requirements heb je geen reproduceerbaar project.
- Componenten – groep_vars/ host_vars
- Variabelen: waarom bestaan deze mappen?
- Doel: Input los van taken, zodat er niet hardcoded wordt gewerkt!
- Componenten – Roles
- Componenten – Playbooks

## IaC Theorie 03 Variables Facts Register
- Infrastructure as Code (IaC)
- Theorie 3: Variables, Facts & Register +
- Magnum Opus Launch
- Data-gedreven Ansible: variables + facts + magic
- Terugblik 1/2
- Terugblik 2/2
- Waarom “data-gedreven” de echte start is van IaC
- Waar definieer je variabelen en waarom daar?
- Waar kun je variabelen declareren, maar belangrijker, waar horen ze logisch thuis?
- Variable precedence*: wat wint als er conflicten zijn?
- Vb. je hebt een groep web met twee hosts: web01 en web02
- Best practice: variabelen
- Best practice: group_vars/host_vars
- Demo
- Vb:
- Facts: wat zijn het en waarom bestaan ze?
- Best practice: facts bewust gebruiken en niet overal
- Magic/special variables die je nu al kan gebruiken

## IaC Theorie 04 TaskControl Loops When Blocks
- Infrastructure as Code (IaC)
- Theorie 4: Task Control – Loops, When, Blocks
- Task Control – Loops, When, Blocks
- Leerdoelen
- Waar staan we nu
- Het probleem: copy-paste drift
- Overzicht: 3 tools, 3 soorten winst
- Loops
- Herhaal één task netjes over meerdere
- Loops: wat is het wel en niet
- Loop over een lijst:
- Loop over een “list of dictionary”
- Conditionals (when)
- Laat tasks alleen lopen waar ze echt
- When: conditionals voor correct gedrag
- When-expressies: syntax cheat sheet 1/2
- When-expressies: syntax cheat sheet 2/2
- Belangrijk: een YAML-lijst van conditions is impliciet and. Dat principe geldt breed

## IaC Theorie 05 Templates Handlers
- Infrastructure as Code (IaC)
- Theorie 5: Templates Handlers
- Templates, Handlers
- Waar staan we nu?
- Het probleem dat we willen oplossen
- Wat is een template in ansible
- De template-task
- Jinja2 essentials
- Vb:
- Jinja2 in tasks: data bruikbaar maken
- Mini demo
- Welke module kies je?
- Wat is een handler?
- Klassieke fouten
- Kerninzichten van vandaag

## IaC Theorie 06 Roles
- Infrastructure as Code (IaC)
- Theorie 6: Roles als standaard bouwblok
- Roles als standaard bouwblok
- Waar staan we nu?
- Waarom roles nu nodig worden
- Wat is een role?
- De role-structuur: wat zit waar?
- Templates en handlers in een role
- Van een “dikke” playbook naar een thin playbook
- Hoe vind Ansible je roles?
- Praktisch starten: role aanmaken
- Veel gemaakte fouten bij roles
- Inzichten van vandaag

## IaC Theorie 07 Secrets Vault
- Infrastructure as Code (IaC)
- Theorie 7: Secrets en Vault als standaard
- Secret en Vault als standaard
- Het echte probleem: repo als bron van waarheid
- Wat is een secret en wat niet?
- Wat is Ansible Vault en wat is het niet?
- Vault is maar zo sterk als je vault password
- Twee Vault-vormen: encrypt_string versus
- Twee Vault-vormen: waarom wel, waarom niet?
- Demo 1 – encrypted_string: één vaulted
- Ansible-projectmap.
- Demo 2 – encrypted file 1/2
- Demo 2 – encrypted file 2/2
- Wat onthouden we over Vault op de CLI?

## IaC Theorie 08 Windows WinRM Chocolatey
- Infrastructure as Code (IaC)
- Theorie 8: Windows met Ansible, WinRM en
- Chocolatey
- Windows met Ansible, WinRM en Chocolatey
- Waar staan we nu?
- Leerdoelen van vandaag
- Wat verandert er als je target Windows is?
- PowerShell-gebaseerd.
- Hoe geraakt Ansible van de controle node tot op
- WinRM setup: wat moet minimaal in orde zijn?
- WinRM setup: op de Windows Server 2025 1/3
- Enable-PSRemoting –Force                              # Self-signed certificaat maken
- DnsName           = $env:COMPUTERNAME
- New-ItemProperty `
- NotAfter          = (Get-
- Subject           =
- WinRM setup: op de Windows Server 2025 2/3
- Path                  =                                  Action      = 'Allow'

## IaC Theorie 09 Ansible API
- Infrastructure as Code (IaC)
- Theorie 9: Ansible & API’s met ansible.builtin.uri
- Waarom API’s in Ansible?
- REST als praktische interface voor automation
- Waar staan we nu?
- Leerdoelen
- Wat zijn API’s?
- API’s als brug tussen digitale werelden
- Waar kom je API-platformen in de praktijk
- HTTP API voor resources zoals dashboards, users, data sources en alerts.
- API expliciet als een RESTful API om netwerken op schaal programmatisch te beheren en
- API’s, modules en uri: wat is het verschil?
- Waarom Ansible met API’s werkt
- Wat is httpapi en waarom is het vandaag niet de
- REST voor Ansible – overzicht
- Waar draait een API-call eigenlijk?
- API
- Anatomie van ansible.builtin.uri

## IaC Theorie 10 Cloud
- Infrastructure as Code (IaC)
- Theorie 10: IaC, Ansible en Cloud
- IaC, Ansible en Cloud
- Waar staan we nu?
- Leerdoelen
- Cloud in context
- Cloud is niet automatisch altijd de juiste keuze
- Waar past Ansible in de cloud 1/2?
- API’s
- Terraform/OpenTofu of provider-native IaC-tools
- Waar past Ansible in de cloud 2/2?
- Kernboodschap:
- Day 0, Day 1, Day 2: waar zit Ansible sterk?
- Denk maar aan het installeren van packages, users
- Waarom een static inventory slecht past bij de cloud
- DNS of andere metadata kunnen mee wijzigen.
- Wat is dynamic inventory?
- Dus een systeem dat weet welke hosts of resources

## IaC Theorie 11 Network Automation Self Healing Networks
- Infrastructure as Code (IaC)
- Theorie 11: Ansible, Network Automation en Self-
- Healing Networks
- Ansible, Network Automation en Self-Healing
- Networks
- Netwerk moet er altijd zijn!
- Monitoring die wegvalt, failover die onverwacht activeert,…
- Een netwerkconfiguratie wijzigen kan de bereikbaarheid van veel hosts
- Waar staan we nu?
- Leerdoelen
- Na deze les kan je:
- Wat is network automation?
- Network automation = netwerkconfiguratie en
- Communicatie met netwerkcomponenten
- Ansible kan netwerkapparaten benaderen via verschillende paden:
- Python-omgeving hebben waarop Ansible-modules kunnen worden uitgevoerd.
- Fortinet, F5…
- Waarom netwerkautomation anders is

## IaC Theorie 12 Automation Controller AWX Operationalisatielaag
- Infrastructure as Code (IaC)
- Theorie 12: Automation Controller/AWX als
- Automation Controller/AWX als operationalisatielaag
- Waar staan we nu?
- Reeds gezien:
- Vandaag:
- Controller/AWX
- Wat doen we vandaag niet?
- Vandaag niet:
- Leerdoelen
- Waarom Automation Controller/AWX?
- Red Hat Ansible Automation Platform, gebaseerd op dezelfde
- AWX vervangt je Ansible-project niet
- Van één CLI-run naar beheerde uitvoering
- CLI-run:
- In dit ene commando zitten meerdere keuzes verborgen:
- AWX maakt deze keuzes expliciet via objecten
- Kernmodel: van CLI naar AWX-objecten

## IaC Theorie 13 PXE Magnum Opus Examen
- Infrastructure as Code (IaC)
- Theorie 13: PXE als zelfstudie, Magnum Opus
- PXE als zelfstudie, Magnum Opus en Examen
- PXE en bare metal provisioning als zelfstudie
- Kernidee
- Zelfstudie:
- Na deze zelfstudie kun je uitleggen
- Puntenverdeling: hoe word je beoordeeld?
- Volgens de ECTS-fiche:
- Tweede examenkans:
- Magnum Opus: wat moet je indienen?
- Magnum Opus: Git-repo afspraken
- Magnum Opus: README en PDF zijn niet hetzelfde
- README:
- Projectdocumentatie-PDF:
- Magnum Opus: Do’s en Don’ts
- Do’s:                                              Don’ts:
- Korte security-check

