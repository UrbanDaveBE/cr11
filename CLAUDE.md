# Anweisungen für Claude Code

**Lies zuerst `PROJEKT.md` im Repo-Root.** Dort stehen Anforderungen,
Architektur, Tech-Stack, Teststrategie und Konventionen. Diese Datei hier
ergänzt sie nur um harte Arbeitsregeln. Bei Widersprüchen gilt `PROJEKT.md`.

## Harte Regeln

1. **Keine Architektur- oder Stack-Entscheidungen im Alleingang.**
   Wenn eine Aufgabe eine solche Entscheidung erzwingt, schlage sie vor
   (als ADR in `docs/decisions/`) statt sie still zu treffen.
2. **Kein Code ohne Tests.** Die Regeln aus PROJEKT.md Abschnitt 6 gelten:
   Domain-Logik → Unit-Tests, Use Cases → Integrationstests gegen echte
   Test-DB. CI muss grün sein.
3. **Kleine Aufgaben, kleine Commits.** Genau ein Use Case bzw. ein klar
   abgegrenzter Teil davon pro Branch. Branch-Namen: `feature/…`,
   `fix/…`, `chore/…`.
4. **Secrets niemals committen.** Konfiguration über Umgebungsvariablen;
   neue Variablen in `.env.example` mit leerem Wert dokumentieren.
5. **Geteilte Verträge leben in `packages/shared`.** Keine Typen für
   API-Requests/-Responses in `apps/` duplizieren.
6. **PROJEKT.md aktuell halten.** Wenn sich durch deine Arbeit Status
   oder Entscheidungen ändern, aktualisiere die Datei im selben Commit.
7. **Ehrlich berichten.** Wenn Tests fehlschlagen oder du etwas
   übersprungen hast: sagen, nicht verschweigen. Fertig heißt geprüft.

## Kontext, den du nicht raten sollst

- Anforderungen: PROJEKT.md Abschnitt 2
- Architektur-Regeln: PROJEKT.md Abschnitt 4
- Getroffene Entscheidungen: `docs/decisions/`

Wenn etwas Unklares besteht, das in diesen Dateien nicht beantwortet ist:
Rückfrage stellen statt raten.
