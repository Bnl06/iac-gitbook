# Infrastructure as Code StudyBook

Zensical studiegids voor het mondeling examen **Infrastructure as Code**.

## Online publiceren

Deze repo gebruikt dezelfde standaard Zensical-aanpak als de voorbeeldbijlage:

- `mkdocs.yml` voor de volledige Zensical/Material-configuratie
- `docs/` als documentatiemap
- `.github/workflows/deploy-docs.yml` voor GitHub Pages
- `zensical build --clean` als build command

## Lokaal bekijken

```bash
pip install zensical # installeert Zensical
zensical serve # start de lokale preview op http://localhost:8000
```

## GitHub Pages

1. Push de repo naar GitHub.
2. Ga naar **Settings → Pages**.
3. Zet **Source** op **GitHub Actions**.
4. Push naar `main` of start de workflow manueel via **Actions**.


## Magnum Opus uitbreiding

Deze versie bevat extra Zensical-tabbladen voor het Vaultwarden Magnum Opus-project: architectuur, roles, templates, secrets/TLS/DNS-01, commands, bewijs, idempotentie, troubleshooting en mondelinge verdediging.
