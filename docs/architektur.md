# Architektur im Detail

> Dieses Dokument beschreibt ausführlich, was in PROJEKT.md Abschnitt 4
> skizziert ist. Es richtet sich an Menschen **und** KI-Agenten: Wer hier
> etwas ändern will, braucht eine Team-Entscheidung und einen ADR
> (`docs/decisions/`).

---

## 1. Gesamtbild

```
Browser → apps/web (GUI)
              │  HTTP/JSON
              ▼
┌──────────── apps/api (Backend) ────────────────────────────┐
│ API-Schicht        HTTP-Routen, Validierung, Auth          │
│                      │                                     │
│                      ▼                                     │
│ Anwendungsschicht    Use Cases, Orchestrierung             │
│                      │                                     │
│                      ▼                                     │
│ Domain               Geschäftslogik, reine Funktionen      │
│                      ▲                                     │
│                      │  (implementiert Interfaces)         │
│ Persistenz           Repositories, ORM, DB-Zugriff         │
└────────────────────────────────────────────────────────────┘
              │
              ▼
        PostgreSQL
```

**Die eine Regel, die alles zusammenhält (Dependency Rule):**
Abhängigkeiten (Imports) zeigen nur **nach innen** — API → Anwendung →
Domain. Die Domain importiert niemals etwas aus einer äußeren Schicht.
Die Persistenz ist die einzige Ausnahme im Pfeilbild: Sie *implementiert*
Interfaces, die die Anwendungsschicht definiert — dadurch zeigt auch
diese Abhängigkeit nach innen (Dependency Inversion).

---

## 2. apps/web — das Frontend (steht außerhalb der Schichten)

Das GUI ist **keine** der vier Schichten. Es ist ein eigener Client, der
das Backend ausschließlich über dessen HTTP-API anspricht — wie jede
andere externe Anwendung auch.

- Darf: Daten anzeigen, Eingaben sammeln, clientseitige
  Plausibilitätsprüfungen (z. B. „Pflichtfeld leer") für gute UX.
- Darf nicht: Geschäftsregeln als einzige Stelle durchsetzen. Alles, was
  das Frontend prüft, prüft das Backend noch einmal — das Frontend ist
  manipulierbar.
- Verträge: Request-/Response-Typen kommen aus `packages/shared`,
  nie selbst neu definieren.

---

## 3. API-Schicht (≈ Controller)

**Aufgabe:** Der Türsteher. Übersetzt HTTP in Methodenaufrufe der
Anwendungsschicht und deren Ergebnisse zurück in HTTP.

**Verantwortlich für:**
- Routen (`POST /contacts`), Request-Parsing, Formvalidierung
  („ist das überhaupt eine E-Mail-förmige Zeichenkette?")
- Authentifizierung/Autorisierung (Token prüfen, Rechte prüfen)
- Fehler der inneren Schichten in HTTP-Statuscodes übersetzen
  (DomainError → 422, NotFound → 404)

**Darf nicht:**
- Geschäftslogik enthalten (kein `if deal.amount > ...` auf fachlicher
  Ebene)
- Direkt auf die Datenbank oder das ORM zugreifen
- Die Persistenz-Schicht importieren

**Tests hier:** dünn. Wer richtig validiert, zeigt sich in den
Integrationstests (API gegen Test-DB). Eigene Unit-Tests nur für
nichttriviale Mapping-/Fehlerübersetzungslogik.

**MVC-Einordnung:** entspricht dem klassischen Controller — „dumm, nur
Orchestrierung". Der Unterschied: Er orchestriert nicht mal den fachlichen
Ablauf, das tut schon die Anwendungsschicht. Er orchestriert nur HTTP.

---

## 4. Anwendungsschicht (Use Cases / Application Services)

**Aufgabe:** Pro Use Case aus PROJEKT.md Abschnitt 2 gibt es hier genau
eine Stelle im Code. Sie orchestriert den **fachlichen Ablauf**: laden →
Regeln anwenden lassen → speichern → Nebeneffekte anstoßen.

**Verantwortlich für:**
- Transaktionsgrenzen
- Aufruf von Repositories (über Interfaces!) und Domain-Objekten
- Nebeneffekte: E-Mails, Events, Audit-Log

**Darf nicht:**
- Geschäftsregeln selbst formulieren — das ist Domain-Sache. Faustregel:
  Ein `if`, das eine Fachregel prüft („Deal ohne Betrag darf nicht
  gewonnen werden"), gehört nicht hierher. Ein `if`, das den Ablauf
  steuert („wenn Speichern fehlschlägt, rollback"), schon.
- HTTP kennen (keine Request-/Response-Objekte, keine Statuscodes)
- Das ORM oder SQL importieren

**Beispiel:**

```python
# Use Case: Kontakt anlegen
def create_contact(cmd: CreateContactCommand,
                   contacts: ContactRepository,   # Interface!
                   companies: CompanyRepository,  # Interface!
                   mailer: Mailer) -> Contact:    # Interface!
    company = companies.get(cmd.company_id)        # laden
    contact = Contact.create(cmd, company)         # Regeln: Domain
    contacts.save(contact)                          # speichern
    mailer.send_welcome(contact)                    # Nebeneffekt
    return contact
```

**Tests hier:** Integrationstests (siehe PROJEKT.md Abschnitt 6) — der
Use Case läuft gegen die echte Test-Datenbank. Das ist die wichtigste
Testebene des Projekts.

**MVC-Einordnung:** der „Ablauf-Teil" klassischer Services.

---

## 5. Domain (Geschäftslogik)

**Aufgabe:** Die eigentlichen Regeln des Fachgebiets, konzentriert an
einem Ort. Entities, Value Objects, Domain-Services.

**Verantwortlich für:**
- Regeln wie „Kontakt braucht gültige E-Mail", „Deal wechselt nur mit
  Betrag in die Stufe ‚gewonnen'", „eine Firma mit offenen Deals kann
  nicht gelöscht werden"
- Zustandsübergänge und Invarianten

**Darf nicht (das ist der härteste Punkt des ganzen Dokuments):**
- Irgendetwas importieren, das HTTP, DB, ORM, Framework oder Infrastruktur
  betrifft
- Von der Außenwelt wissen (keine Sessions, keine aktuellen Nutzer, keine
  URLs)
- I/O machen (keine Dateien, kein Netzwerk, keine DB)

Domain-Code ist damit eine Sammlung reiner Funktionen und Klassen, die
Daten hereingibt und Ergebnisse/Fehler herausbekommt.

**Warum so streng?** Weil genau das die Teststrategie ermöglicht: Jede
Regel ist in Millisekunden unit-testbar, ohne Datenbank, ohne Server,
ohne Mocks. Und weil Regeln so unabhängig vom Framework bleiben —
Frameworks wechseln, Geschäftsregeln nicht.

**Tests hier:** Unit-Tests, viele, schnell. Regel: keine Domain-Logik
ohne Unit-Test.

**MVC-Einordnung:** der „Regel-Teil" klassischer Services bzw. das, was
in MVC oft diffus im „Model" versammelt ist.

---

## 6. Persistenz (Repositories)

**Aufgabe:** Der einzige Ort, der die Datenbank kennt. Übersetzt zwischen
Domain-Objekten und Tabellen.

**Verantwortlich für:**
- Implementierung der Repository-Interfaces, die die **Anwendungsschicht
  definiert** (nicht umgekehrt!)
- ORM-Mapping, SQL, Migrationen

**Darf nicht:**
- Geschäftslogik enthalten (kein fachliches `if` — nur speichern/laden)
- Von der API-Schicht importiert werden (die geht über die
  Anwendungsschicht)

**Dependency Inversion in einem Satz:** Nicht der Use Case hängt vom
Datenbankcode ab, sondern der Datenbankcode vom Interface des Use Cases.
Praktischer Effekt: Anwendungsschicht + Domain laufen und kompilieren,
auch wenn noch gar keine Datenbank existiert.

**Tests hier:** werden durch die Integrationstests der Anwendungsschicht
mit abgedeckt (echte Test-DB via Docker). Keine Mocks der eigenen
Datenbank — niemals.

---

## 7. Zuordnung zur klassischen MVC-/Layered-Welt

| Klassisch | Hier | Bemerkung |
|---|---|---|
| View | `apps/web` | eigener Prozess, spricht HTTP |
| Controller | API-Schicht | noch dünner als klassisch: nur HTTP |
| Services | **aufgeteilt:** Anwendungsschicht (Ablauf) + Domain (Regeln) | die zentrale Änderung |
| DAO/Repository | Persistenz | aber Interface gehört der Anwendungsschicht |

**Warum die Service-Aufteilung?** In klassischen Layern wachsen Services
mit der Zeit zu „fat services": Ablauf und Fachregeln vermischen sich,
jeder Regel-Test braucht eine Datenbank. Die Aufteilung zwingt Regeln an
einen Ort, an dem sie trivial testbar sind. Preis: eine Schicht mehr,
mehr Dateien. Für ein CRUD-Tool ohne Regeln wäre das Overkill — ein CRM
mit Pipeline-Logik, Berechtigungen und Validierungen hat genug Regeln,
dass sich die Trennung lohnt.

---

## 8. Durchgang durch alle Schichten (Beispiel)

`POST /contacts { "email": "a@b.c", "company_id": "…" }`

1. **API:** JSON parsen, Format prüfen, Auth-Token prüfen → ruft
   `create_contact(cmd, …)`
2. **Anwendung:** Firma via `CompanyRepository`-Interface laden →
   `Contact.create(...)` aufrufen → `contacts.save(...)` →
   `mailer.send_welcome(...)`
3. **Domain:** `Contact.create` prüft E-Mail-Gültigkeit und Firma-vorhanden
   → wirft `DomainError` oder liefert ein valides `Contact`-Objekt
4. **Persistenz:** `PostgresContactRepository.save()` schreibt die Zeile
5. **API:** übersetzt Ergebnis → `201 Created` (+ JSON), oder
   `DomainError` → `422`

Fehlerfall „ungültige E-Mail": Schritt 3 wirft, Schritte 4 und 5
(Speichern) passieren nie, die API übersetzt in 422 — und ein Unit-Test
der Domain hat genau diesen Fall bereits ohne jede Datenbank geprüft.
