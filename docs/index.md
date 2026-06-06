---
title: Infrastructure as Code StudyBook
description: Zensical studiegids voor het mondeling examen Infrastructure as Code.
hide:
  - navigation
  - toc
---

# Infrastructure as Code — StudyBook

!!! abstract "Doel van deze studiegids"

    Deze Zensical-site helpt je om de volledige theorie voor het **mondeling examen Infrastructure as Code** te begrijpen, uit te leggen en toe te passen.  
    De focus ligt niet op blind vanbuiten leren, maar op **mondeling kunnen redeneren met voorbeelden**.

<div class="grid cards" markdown>

-   :material-school:{ .lg .middle } __Start met studiepad__

    ---

    Volg een duidelijke volgorde om de theorie slim te verwerken en te herhalen.

    [:octicons-arrow-right-24: Studiepad openen](exam/studiepad.md)

-   :material-chat-question:{ .lg .middle } __Mondelinge vragen__

    ---

    Oefen met examenvragen en voorbeeldantwoorden die je vlot kunt uitleggen.

    [:octicons-arrow-right-24: Vragen oefenen](exam/mondelinge-vragen.md)

-   :material-code-braces:{ .lg .middle } __Ansible voorbeelden__

    ---

    Bekijk codevoorbeelden met uitleg bij elke belangrijke command of taak.

    [:octicons-arrow-right-24: Code bekijken](examples/ansible-code-met-uitleg.md)

-   :material-clipboard-check:{ .lg .middle } __Checklist__

    ---

    Controleer of je Magnum Opus, AWX, cloud, roles, secrets en automation begrijpt.

    [:octicons-arrow-right-24: Checklist openen](exam/magnum-opus-checklist.md)

</div>

## Hoe gebruik je deze site?

1. Lees per hoofdstuk eerst de **kernzin**.
2. Probeer de uitleg hardop na te vertellen alsof je op het mondeling zit.
3. Bekijk de voorbeelden en let op de `# uitleg` bij commands.
4. Test jezelf via de mondelinge vragen.
5. Herhaal de checklist vlak voor het examen.

!!! tip "Mondeling antwoorden"

    Begin je antwoord altijd met een korte definitie, geef daarna een praktisch voorbeeld en eindig met waarom dit nuttig is in automatisatie.

## Lokaal openen

```bash
pip install zensical # installeert Zensical lokaal
zensical serve # start de lokale website op http://localhost:8000
```

## Online publiceren

Deze repo bevat een GitHub Actions workflow die bij elke push naar `main` automatisch bouwt met Zensical en publiceert naar GitHub Pages.
