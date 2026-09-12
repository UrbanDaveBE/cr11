# CRM-Projekt

Ein CRM für [TBD: Zielgruppe]. Anforderungen, Architektur und Konventionen
stehen in [PROJEKT.md](PROJEKT.md).

## Voraussetzungen

- [TBD: Node-Version / Python-Version — ergänzen, sobald der Stack feststeht]
- Docker (für die Test-Datenbank und lokale Entwicklung)

## Setup

```bash
git clone <repo-url>
cd crm-projekt
cp .env.example .env   # danach Werte eintragen
# [TBD: Installationsbefehl, sobald der Stack feststeht]
```

## Starten

```bash
# [TBD]
```

## Tests

```bash
# Unit-Tests:          [TBD]
# Integrationstests:   [TBD — startet eine Test-Datenbank via Docker]
# Alle Tests:          [TBD]
```

Die Teststrategie (was wo getestet wird) steht in PROJEKT.md, Abschnitt 6.

## Struktur

```
apps/api         Backend
apps/web         Frontend
packages/shared  Geteilte Typen und API-Verträge
docs/decisions   Architektur-Entscheidungen (ADRs)
docs/prompts.md  Starter-Prompts für die Arbeit mit KI-Agenten
```

## Mitmachen

1. Aufgabe aus PROJEKT.md Abschnitt 2 wählen (klein schneiden!)
2. Branch: `feature/<kurzbeschreibung>`
3. Definition of Done aus PROJEKT.md Abschnitt 7 erfüllen
4. PR öffnen — Review durch eine andere Person bzw. einen anderen Agenten
