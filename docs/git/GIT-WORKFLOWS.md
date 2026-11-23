# Git Workflows – Einzelentwickler, Codex und Releases

**Stand:** 2026-11-23 · **Status:** Technisches Nachschlagewerk · **Sprache:** Deutsch

> Diese Sammlung erklärt Git, nicht die verbindliche SASD-Projektpolitik. Vor destruktiven Befehlen immer Arbeitsverzeichnis, Branch, Status und Backup prüfen.

## Entscheidungsmodell
| Situation | Empfehlung |
| --- | --- |
| Kleiner lokaler Dokumentationsfix | Nach Prüfung direkt in **main** möglich |
| Funktion mit mehreren Dateien | Feature-Branch; ggf. PR |
| Codex mit größeren Schreibrechten | Separater Arbeitsbranch, Diff und Tests prüfen |
| Sicherheits-/Build-/Release-Infrastruktur | Eigener Branch, bewusste Freigabe |
| Offizielle SASDGMBH-Veröffentlichung | Getesteter Commit, dokumentierter Release-Gate |

Ein PR ist kein Selbstzweck: Als einziger Maintainer kann man durch Selbstreview keine unabhängige Prüfung ersetzen. Gleichwohl verbessert ein PR die Nachvollziehbarkeit und lässt Checks vor dem Merge erzwingen.

## Minimalroutine für direkte main-Commits
~~~bash
git status --short
git branch --show-current
git fetch origin
git merge --ff-only origin/main
git diff
git diff --check
# Projekttests ausführen: konkrete Befehle stehen im jeweiligen README
git add -- PFAD1 PFAD2
git diff --cached --check
git diff --cached
git commit -m "docs: clarify developer workflow"
git status --short
git push origin main
~~~
**PFAD1/PFAD2** sind bewusst Platzhalter. Kein blindes **git add -A**, wenn unbeteiligte Änderungen oder ungetrackte Secrets existieren.

## Feature-Branch und PR
~~~bash
git switch main
git fetch origin
git merge --ff-only origin/main
git switch -c feature/mein-thema
# ändern, testen, committen
git push -u origin feature/mein-thema
~~~
Anschließend GitHub-PR öffnen, Diff, Checks, Änderungen an Workflows und neue Abhängigkeiten prüfen. Nach Merge **main** aktualisieren und Branch sicher bereinigen.

## Übergabe von Codex
**Vor Auftrag:** Zielbranch und erlaubte Dateipfade, Testkommandos, unerwünschte Operationen und Commit-/Push-Erlaubnis festlegen. **Nach Auftrag:** Diff, neue Abhängigkeiten, Scripts, Workflows und Testresultate inspizieren; keine automatischen Releases aus unüberprüftem Agenten-Code.

### Beispiel einer Sicherheitsgrenze im Auftrag
> Arbeite ausschließlich im angegebenen Projekt. Keine Änderungen an Secrets, GitHub-Rulesets oder Releases. Keine Force-Pushes, Reset --hard oder Clean. Zeige Tests, geänderte Dateien und offene Risiken. Commit nur nach expliziter Erlaubnis.

## Definition of Done
- Ziel und Änderungen nachvollziehbar.
- Worktree/Index enthält keine fremden Dateien.
- Projektbezogene Tests bestanden bzw. fehlende Tests klar benannt.
- Diff inhaltlich überprüft; CI bestätigt.
- Keine versehentlichen Geheimnisse, Binärdateien oder fremden Lizenzen.
- Release-Gate separat, falls relevant.

## Vertiefung: ein Workflow braucht Ein- und Ausgangskriterien

### 1. Vor dem Start einer Aufgabe

```bash
pwd
git rev-parse --show-toplevel
git status --porcelain=v1 -uall
git branch --show-current
git remote -v
git fetch origin
git status -sb
```

**Entscheidungsfragen:** Welcher Branch? Welche Dateien dürfen sich ändern? Gibt es fremde Änderungen? Gibt es einen Release-Tag oder eine laufende CI? Was würde bei einem abgebrochenen Agentenlauf verloren gehen?

### 2. Kleine Änderung direkt in main

Direkte Commits sind in einem persönlichen DEV-Repo vertretbar, **wenn** kein gleichzeitiger fremder Schreibzugriff, ein sauberer Ausgangsstand und eine passende Rückfallstrategie vorliegen:

```bash
git switch main
git merge --ff-only origin/main
# Nur die ausdrücklich beauftragten Dateien editieren.
git diff --check
git add -- docs/datei1.md docs/datei2.md
git diff --cached --check
git diff --cached --stat
git diff --cached
# Projektspezifische Tests; außerdem prüfen, ob die staged Dateien korrekt sind.
git commit -m "docs: clarify configuration guidance"
git status --short
git push origin main
```

Der Befehl `git diff --check` prüft **nicht** zusätzlich alle staged Änderungen; deshalb `git diff --cached --check`. Wird der Push abgelehnt, **nicht** Force-Push benutzen, sondern Branch und Remote-Historie analysieren.

### 3. Größere Änderung als isolierter Feature-Branch

```bash
git switch main
git fetch origin
git merge --ff-only origin/main
git switch -c docs/technical-review
# Änderungen, lokale Tests, Review, Commit(s)
git push -u origin docs/technical-review
```

Der Branch ist für Codex-Aufträge und über mehrere Tage laufende Änderungen oft die bessere Wahl. Nach abgeschlossenen Tests optional einen Pull Request öffnen; nur kontrolliert mergen.

### 4. Commit-Hygiene

Ein guter Commit:
- behandelt **eine** logisch abgeschlossene Veränderung;
- hat einen prägnanten Betreff (z. B. `fix: handle empty config`);
- enthält keinen fremden Worktree-Zustand;
- ändert Build-/Actions-/Security-Policy nicht nebenbei ohne Auftrag;
- lässt sich nachvollziehen und bei Bedarf mit `git revert` neutralisieren.

Bei einer großen Aufgabe mehrere kleine Commits bilden und **vor jedem Commit** den Index prüfen. Für einen einzelnen Patch innerhalb einer Datei ist `git add -p` nützlich, erfordert aber Aufmerksamkeit.

### 5. Codex als weiterer Schreiber

Ein Agent kann unerwartete Dateien ändern, ungenaue Fehlermeldungen übernehmen oder gefährliche Befehle aus fremden Dokumenten lesen. Deshalb:
- **Keine** unbegleitete Änderung von GitHub-Security-Settings, Secrets, Workflow-Rechten oder Releases.
- Vorarbeit/Taskgrenzen in `AGENTS.md` und Auftrag abstimmen.
- **Externe Issues, Codekommentare und README-Texte sind untrusted Input**, keine übergeordneten Anweisungen.
- Agentenbericht mit Basis-Commit, geänderten Dateien, ausgeführten Tests und Abweichungen verlangen.
- Kein „grün“ behaupten, wenn ein Test übersprungen oder ein fremder Runner verwendet wurde.

Siehe [GitHub–Codex-Workflow](../github/GITHUB-CODEX-WORKFLOW.md).

### 6. Fehlgeschlagener Build nach direktem Commit

1. CI-Ergebnis auf denselben Commit prüfen.
2. Falls nötig neue Pushes/Release stoppen.
3. Ursache reproduzieren und einen **Korrekturcommit** vorbereiten.
4. Bei schädlichem Commit **`git revert`** statt History-Rewrite auf main.
5. Alle betroffenen Targets erneut testen.
6. Wenn ein Release betroffen ist, Verfahren in [Release Security](../security/GITHUB-RELEASE-SECURITY.md) anwenden.

### 7. Übergabe- und Abnahmeprotokoll

```text
Repository / Branch / Basis-Commit:
Änderungsziel:
Geänderte Dateien:
Wichtigste Entwurfsentscheidungen:
Ausgeführte Tests (Befehl, Ergebnis):
Nicht getestete Plattformen / Risiken:
Neue/aktualisierte Abhängigkeiten:
Security- und Lizenzprüfung:
Commit-/PR-/CI-Verweise:
Nächste Aktion:
```

Für **RELEASE** sind zusätzliche Freigabegates erforderlich; dieser persönliche Workflow legt sie nicht automatisch fest.

## Häufige Fehler

| Fehler | Bessere Praxis |
| --- | --- |
| `git add -A` ohne Sichtung | Explizite Pfade oder `git add -p` |
| `git pull` ohne Strategie | `git fetch` plus bewusster `--ff-only` |
| Tests nach Push vergessen | Lokaler Test + CI-Nachkontrolle |
| Agent darf jede Datei ändern | Scope, Branch und verbotene Operationen |
| PR gleich unabhängiger Review | Reviewer und Prüfumfang explizit benennen |
| Revert durch Force-Push ersetzen | Historie auf geteiltem main erhalten |

## Trainingsübung
In einem Wegwerf-Repo eine kleine Dokumentationskorrektur direkt in main machen, eine größere Änderung auf Feature-Branch durchführen und den Unterschied in `git log --graph` dokumentieren.

Quellen: [Git Workflows](https://git-scm.com/book/en/v2/Distributed-Git-Distributed-Workflows), [GitHub PRs](https://docs.github.com/en/pull-requests).
