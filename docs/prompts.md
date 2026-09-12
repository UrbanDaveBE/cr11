# Starter-Prompts für die Arbeit mit KI-Agenten

Diese Prompts sind Vorlagen. Eckige Klammern `[...]` vor dem Abschicken
ersetzen. Alle Agenten lesen `PROJEKT.md` + `CLAUDE.md`/`AGENTS.md`
automatisch — der Projektkontext muss also nicht wiederholt werden,
es geht nur um die **konkrete Aufgabe**.

---

## 1. Feature umsetzen

> Setze Use Case [#X] aus PROJEKT.md um: „[Use-Case-Text]".
> Halte dich an die Architektur-Regeln (Abschnitt 4) und die
> Teststrategie (Abschnitt 6). Arbeite auf dem Branch `feature/[name]`.
> Fertig heißt: Unit- und Integrationstests geschrieben, alle Tests
> grün, PROJEKT.md-Status aktualisiert. Wenn dir Informationen fehlen,
> frag nach, statt zu raten.

## 2. Code reviewen (durch den *anderen* Agenten!)

> Reviewe den Branch `feature/[name]` gegen `main`. Prüfe:
> (1) Erfüllt der Code den Use Case aus PROJEKT.md?
> (2) Werden die Architektur-Regeln aus Abschnitt 4 eingehalten
> (insb. keine Framework-/DB-Imports in der Domain-Schicht)?
> (3) Gibt es Unit- und Integrationstests, und testen sie Verhalten
> statt Implementierung?
> (4) Sicherheits-Basics: keine Secrets, Eingabe-Validierung,
> parametrisierte DB-Zugriffe.
> Liste die Befunde nach Schweregrad. Ändere selbst nichts am Code.

## 3. Architektur-Entscheidung vorbereiten

> Wir müssen [Frage aus PROJEKT.md Abschnitt 8] entscheiden.
> Erstelle einen ADR nach `docs/decisions/000-template.md` mit
> 2–3 realistischen Optionen, Vor-/Nachteilen und einer Empfehlung.
> Entscheiden tun wir — du lieferst die Grundlage.

## 4. Debugging

> Der Test [Name] schlägt fehl mit: [Fehlermeldung einfügen].
> Finde die Ursache, bevor du etwas änderst. Erkläre mir die Ursache
> in zwei Sätzen, schlage dann den Fix vor. Kein Raten, kein
> „einfach nochmal probieren".

---

## Grundregeln für gute Prompts

- **Eine Aufgabe pro Prompt.** „Baue Auth + Kontakte + Pipeline" endet
  in Chaos.
- **Erfolgskriterium nennen** („Tests grün", „CI läuft"), sonst
  entscheidet der Agent selbst, wann er fertig ist.
- **Nachfragen erlauben** — der Satz „frag, statt zu raten" ist bei
  jedem Prompt Gold wert.
- **Fehlermeldungen komplett einfügen**, nicht umschreiben.
