# Git Branching – unabhängige Arbeitslinien

**Stand:** 2026-11-23 · **Status:** Technisches Nachschlagewerk · **Sprache:** Deutsch

> Diese Sammlung erklärt Git, nicht die verbindliche SASD-Projektpolitik. Vor destruktiven Befehlen immer Arbeitsverzeichnis, Branch, Status und Backup prüfen.

## Prinzip
Ein Branch ist eine verschiebbare Referenz auf einen Commit. **main** bezeichnet einen Branch; **origin/main** ist eine lokale Remote-Tracking-Referenz (kein Live-Status). **HEAD** zeigt die aktuelle Position an. Das Erstellen eines Branches kopiert das Repository nicht.

## Routine
~~~bash
git status --short
git switch main
git fetch --prune origin
git status -sb
git switch -c feature/verbesserte-dokumentation
# Änderungen, Tests, git add und git commit
git log --oneline --graph --decorate --all -15
~~~

## Lokale Arbeit auf main (Einzelentwickler)
~~~bash
git switch main
git fetch origin
git merge --ff-only origin/main
# ändern, prüfen, testen und committen
git push origin main
~~~
Dieser Ablauf setzt einen bereits sauberen Worktree und einen geeigneten Remote voraus. Ein Schutz-Ruleset gegen Löschen und Force-Push verhindert keine normalen direkten Commits.

## Feature-Branch lokal integrieren
~~~bash
git switch main
git merge --ff-only feature/verbesserte-dokumentation
git push origin main
git branch -d feature/verbesserte-dokumentation
~~~
**--ff-only** schlägt fehl, wenn die Historie divergiert; dann Ursache klären statt automatisch einen Rewrite oder Force-Push zu versuchen. Für PRs den Branch zuvor zum Remote pushen, etwa **git push -u origin feature/verbesserte-dokumentation**.

## Tracking, Verwechslungen und Löschung
~~~bash
git branch -vv
git branch --merged main
git branch --no-merged main
git remote show origin
~~~
**git branch -d** löscht nur den lokalen Zeiger und prüft die Merge-Beziehung; **git branch -D** erzwingt die Löschung und ist ein Ausnahmefall. Einen Remote-Branch löscht **git push origin --delete BRANCH**; niemals auf einem fremden oder geschützten Branch ohne explizite Freigabe ausführen.

## Übung
In einem Wegwerf-Repository zwei kleine Branches mit unterschiedlichen Dateinamen erstellen; beide ohne Konflikt nach **main** integrieren. Erfolg: **git log --graph --all** zeigt die nachvollziehbare Historie.

Quelle: [Git Branching](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell).
