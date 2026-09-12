# CRM-Projekt — Gemeinsame Projektgrundlage

> Diese Datei ist die **einzige Quelle der Wahrheit** für Anforderungen,
> Architektur und Konventionen. Alle KI-Agenten (Claude Code, Kimi/pi)
> arbeiten auf Basis dieses Dokuments. Änderungen daran werden im Team
> abgesprochen und per Commit festgehalten — niemals nur mündlich.
>
> Status jedes Abschnitts: 🟡 Platzhalter | 🟢 entschieden | 🔴 offen

---

## 1. Ziel & Scope 🟡

**Was ist das Produkt in einem Satz:**
> [TBD: z. B. „Ein CRM für kleine Agenturen, das Kontakte, Deals und
> Kommunikationshistorie an einem Ort bündelt."]

**Zielgruppe:** [TBD]

**Nicht-Ziele** (genauso wichtig! Was bauen wir bewusst *nicht*?):
- [TBD: z. B. kein E-Mail-Versand, keine Mobile App in v1, kein Billing]

---

## 2. Use Cases / Anforderungen 🟡

Format: *Als [Rolle] möchte ich [Was], um [Warum].*
Priorität: **M** = Muss (MVP), **S** = Soll (v1), **K** = Kann (später)

| # | Prio | Use Case | Status |
|---|------|----------|--------|
| 1 | M | Als Nutzer möchte ich mich registrieren und einloggen, damit meine Daten geschützt sind. | 🟡 |
| 2 | M | Als Nutzer möchte ich Kontakte anlegen, bearbeiten und löschen. | 🟡 |
| 3 | M | Als Nutzer möchte ich Kontakte zu Firmen gruppieren. | 🟡 |
| 4 | M | Als Nutzer möchte ich Notizen/Aktivitäten an Kontakten festhalten. | 🟡 |
| 5 | S | Als Nutzer möchte ich Deals mit Stufen (Pipeline) verwalten. | 🟡 |
| 6 | S | Als Nutzer möchte ich über alles suchen können. | 🟡 |
| 7 | K | … | 🟡 |

**Regel:** Ein Use Case gilt erst als umgesetzt, wenn die Tests nach
Abschnitt 6 existieren und grün sind.

---

## 3. Tech-Stack 🟡

**Grundprinzip:** Mainstream wählen. KI-Agenten produzieren auf weit
verbreiteten Stacks (viel Trainingsmaterial, stabile APIs) deutlich
besseren Code als auf Nischen-Technologien. Das ist für euch wichtiger
als technische Eleganz.

| Bereich | Kandidaten | Entscheidung | Begründung |
|---------|-----------|--------------|------------|
| Sprache | TypeScript / Python | [TBD] | |
| Backend | NestJS / Fastify / FastAPI | [TBD] | |
| Frontend | React (Next.js) / Vue | [TBD] | |
| Datenbank | PostgreSQL | [TBD] | |
| ORM/DB-Zugriff | Prisma / Drizzle / SQLAlchemy | [TBD] | |
| Auth | Auth.js / eigene JWT-Lösung | [TBD] | |
| Unit-Tests | Vitest / Jest / pytest | [TBD] | |
| Integrationstests | Supertest + Test-DB / Testcontainers | [TBD] | |
| CI | GitHub Actions | [TBD] | |
| Hosting | [TBD] | [TBD] | |

> Empfehlung als Diskussionsgrundlage: TypeScript durchgehend (Front- und
> Backend teilen sich Typen in `packages/shared`), PostgreSQL, GitHub
> Actions. Aber: entscheidet das gemeinsam.

---

## 4. Architektur 🟡

**Grundprinzip:** Modularer Monolith zuerst. Keine Microservices —
die braucht ihr mit 3 Personen nicht, sie verdreifachen nur den
Betriebsaufwand und erschweren den Agenten den Überblick.

**Schichten (Dependency Rule: Abhängigkeiten zeigen nur nach innen):**

```
API-Schicht (HTTP-Routen, Validierung, Auth)
      │
      ▼
Anwendungsschicht (Use Cases, Orchestrierung)
      │
      ▼
Domain (Geschäftslogik, reine Funktionen, keine Framework-Imports)
      ▲
      │
Persistenz (Repositories, ORM, DB-Zugriff)
```

- Domain-Code importiert **nichts** aus Framework, DB oder HTTP.
  → Dadurch ist er mit einfachen Unit-Tests testbar.
- Persistenz ist hinter Interfaces versteckt, die die Anwendungsschicht
  definiert.
- [TBD: Architekturdiagramm ergänzen, sobald Use Cases stehen]

---

## 5. Repo-Struktur (Monorepo) 🟡

```
crm-projekt/
├── apps/
│   ├── api/            # Backend
│   └── web/            # Frontend
├── packages/
│   └── shared/         # Geteilte Typen, Validierungsschemas, Konstanten
├── docs/
│   └── decisions/      # ADRs (s. Abschnitt 7)
├── PROJEKT.md          # Diese Datei
├── CLAUDE.md           # Verweist auf PROJEKT.md
├── AGENTS.md           # Verweist auf PROJEKT.md
└── README.md           # Setup, Start, Tests — für Menschen
```

**Konventionen:**
- Ordner und Pakete klein mit Bindestrich.
- `packages/shared` ist der einzige Ort, den Front- und Backend gemeinsam
  importieren. API-Verträge (Request/Response-Typen) leben dort.
- Zuständigkeiten: [TBD — wer „besitzt" welchen Bereich?]

---

## 6. Teststrategie 🟡

**Testpyramide — viele unten, wenige oben:**

| Ebene | Was | Wie viele | Werkzeug |
|-------|-----|-----------|----------|
| Unit | Domain-Logik, reine Funktionen, Validierung. Keine DB, kein Netzwerk, kein Framework. | Viele, schnell (< 1 s gesamt am Anfang) | [TBD] |
| Integration | API-Endpunkte gegen eine **echte Test-Datenbank** (z. B. via Testcontainers/Docker). Prüft: Request → Use Case → DB → Response. | Pro Use Case mindestens 1–3 | [TBD] |
| E2E | Kritische User-Flows durchs UI (Login, Kontakt anlegen). | Sehr wenige (3–5) | [TBD: Playwright o. ä.] |

**Regeln:**
1. Kein neuer Use Case ohne Integrationstest. Keine Domain-Logik ohne
   Unit-Test.
2. Tests laufen in der CI bei jedem Push; grün ist Pflicht fürs Mergen.
3. Mocks nur an den Rändern (externe APIs, Zeit). Nie die eigene
   Datenbank mocken — dafür sind Integrationstests da.
4. Ein Test testet Verhalten, nicht Implementierung (keine Tests auf
   private Methoden oder interne Aufrufreihenfolgen).

---

## 7. Arbeitsweise mit KI-Agenten 🟡

**Geteilter Kontext:**
- Diese Datei + `CLAUDE.md` / `AGENTS.md` (die hierauf verweisen) sind der
  gemeinsame Kontext aller Agenten. Wer Kontext nur mündlich/im Chat
  gibt, erzeugt Wissen, das die anderen nicht haben.
- Architektur- und Stack-Entscheidungen werden als kurze ADRs
  (*Architecture Decision Records*, eine Datei pro Entscheidung) in
  `docs/decisions/` festgehalten: Kontext, Entscheidung, Alternativen,
  Konsequenzen.

**Arbeitsablauf pro Aufgabe:**
1. Aufgabe klein schneiden: genau ein Use Case oder ein klar abgegrenzter
   Teil davon pro Agenten-Session.
2. Eigener Branch pro Aufgabe, keine parallelen Agenten auf demselben
   Ordner (Konfliktvermeidung durch die Zuständigkeiten aus Abschnitt 5).
3. Der Agent liefert: Code + Tests + grüner CI-Lauf.
4. Review: ein **anderer** Agent (oder Mensch) reviewt den PR — nicht der
   Agent, der ihn geschrieben hat.
5. Mergen nur bei grüner CI.

**Definition of Done:**
- [ ] Use Case aus Abschnitt 2 referenziert
- [ ] Unit- + Integrationstests vorhanden und grün
- [ ] PROJEKT.md/ADRs aktualisiert, falls Entscheidungen gefallen sind
- [ ] Keine Secrets im Code, `.env` in `.gitignore`, `.env.example`
      mit leeren Werten committed

---

## 8. Offene Entscheidungen 🔴

| # | Frage | Optionen | Wer klärt's | Bis wann |
|---|-------|----------|-------------|----------|
| 1 | Sprache/Stack (Abschnitt 3) | TS vs. Python | alle | [TBD] |
| 2 | Multi-Tenancy: wie werden Kunden/Teams getrennt? | Schema pro Tenant vs. tenant_id-Spalte | [TBD] | [TBD] |
| 3 | Auth: selbst bauen vs. Library | | [TBD] | [TBD] |
| 4 | Deployment-Ziel | | [TBD] | [TBD] |
