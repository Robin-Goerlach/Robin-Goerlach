# Git Merge und Rebase – Historie bewusst gestalten

**Stand:** 2026-11-23 · **Status:** Technisches Nachschlagewerk · **Sprache:** Deutsch

> Diese Sammlung erklärt Git, nicht die verbindliche SASD-Projektpolitik. Vor destruktiven Befehlen immer Arbeitsverzeichnis, Branch, Status und Backup prüfen.

## Unterschied
**Merge** verbindet Historien, ohne bestehende Commits umzuschreiben (bei echtem Merge entsteht ein Merge-Commit). **Fast-forward** bewegt nur einen Zeiger. **Rebase** spielt Commits auf einem neuen Ausgangspunkt erneut ab und erzeugt in der Regel **neue Commit-IDs**.

## Merge ohne Historienumschreibung
~~~bash
git switch main
git merge --ff-only feature/beispiel
~~~
Ist ein Fast-forward nicht möglich, **nicht** sofort mit **--no-ff** oder Rebase weitermachen: erst Branch-Historie und beabsichtigte Integrationsart klären.

## Rebase eines lokalen, unveröffentlichten Branches
~~~bash
git switch feature/beispiel
git fetch origin
git rebase origin/main
# Bei Konflikten: Dateien reparieren, dann:
git add PFAD
git rebase --continue
# Abbruch ohne Übernahme:
git rebase --abort
~~~
Rebase öffentlich gemeinsam genutzter Commits ist riskant, weil sich IDs ändern. Kein **git push --force** auf **main**.

## Konfliktbehandlung
~~~bash
git status
git diff --name-only --diff-filter=U
# Konfliktmarker <<<<<<<, =======, >>>>>>> in Dateien prüfen
git add PFAD
# Bei Merge:
git merge --continue
# Oder den begonnenen Merge verwerfen:
git merge --abort
~~~
Konflikte sind keine bloßen Marker-Probleme: Semantik und Tests der zusammengeführten Funktion kontrollieren. Während eines Rebase kann **ours/theirs** anders wirken als erwartet; nicht blind mit **-X theirs** lösen.

## Wann welche Strategie?
| Situation | Werkzeug |
| --- | --- |
| Eigene unabhängige Entwicklung zusammenführen | Merge oder PR-Merge |
| Lokalen unveröffentlichten Branch aktualisieren | Rebase möglich |
| Gemeinsame History nachvollziehbar erhalten | Merge bevorzugen |
| Fehler auf bereits gepushtem main korrigieren | **git revert**, nicht History Rewrite |
| Keine neuen Merge-Commits zulassen | **--ff-only** |

## Übung
In einem Wegwerf-Repository zwei Branches bearbeiten, denselben Satz in einer Datei ändern und einmal einen Konflikt provozieren. Abbruchbefehle testen und anschließend Konflikt bewusst lösen.

Quellen: [Merge](https://git-scm.com/docs/git-merge), [Rebase](https://git-scm.com/docs/git-rebase), [Konflikte](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging).
