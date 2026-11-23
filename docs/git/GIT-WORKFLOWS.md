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
git fetch origin main
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

Quellen: [Git Workflows](https://git-scm.com/book/en/v2/Distributed-Git-Distributed-Workflows), [GitHub PRs](https://docs.github.com/en/pull-requests).
