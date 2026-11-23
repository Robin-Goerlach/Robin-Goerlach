# Git Cheatsheet – sichere Befehlsübersicht

**Stand:** 2026-11-23 · **Status:** Technisches Nachschlagewerk · **Sprache:** Deutsch

> Diese Sammlung erklärt Git, nicht die verbindliche SASD-Projektpolitik. Vor destruktiven Befehlen immer Arbeitsverzeichnis, Branch, Status und Backup prüfen.

## Beobachten (ungefährlich)
~~~bash
git status -sb
git branch -vv
git remote -v
git log --oneline --decorate --graph --all -20
git diff
git diff --cached
git diff --check
git show --stat HEAD
git ls-files
git config --list --show-origin
~~~

## Aktualisieren (lokales Repository)
~~~bash
git fetch --prune origin
git switch main
git merge --ff-only origin/main
~~~
**fetch** verändert die aktuellen Arbeitsdateien nicht. **merge --ff-only** kann bei Divergenz abbrechen. Vorher auf sauberen Worktree achten.

## Änderungen einchecken
~~~bash
git add -- docs/datei.md
git diff --cached
git commit -m "docs: describe new topic"
git push origin main
~~~

## Branches und PRs
~~~bash
git switch -c feature/beispiel
git push -u origin feature/beispiel
git branch --merged main
git branch -d feature/beispiel
~~~

## Schnelle Fehlerdiagnose
| Symptom | Erste Prüfung |
| --- | --- |
| Push rejected / non-fast-forward | git fetch; git log --graph --all |
| Konflikte | git status; git diff --name-only --diff-filter=U |
| Falsche Identität | git var GIT_AUTHOR_IDENT; git config --show-origin --get user.email |
| Datei versehentlich gestaged | git restore --staged -- PFAD |
| Commit vermisst | git reflog -20 |
| Remote falsch | git remote -v; Repository-URL prüfen |

## Nicht ohne gezielte Entscheidung
**git reset --hard**, **git clean -fd**, **git push --force**, **git branch -D**, **git filter-repo**, **git rebase auf geteilten Branches**.

## Nächster Schritt
Erst die jeweilige Detailseite lesen, dann den Befehl mit **git status** und einem Backup- oder Testrepository absichern.

Quelle: [Git Reference](https://git-scm.com/docs).
