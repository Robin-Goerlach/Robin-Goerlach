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

## Vertiefung: tägliche Routine nach Risikostufe

### Grün – lesende Diagnose

```bash
git rev-parse --show-toplevel
git status --porcelain=v1 -uall
git branch --show-current
git log -8 --oneline --decorate
git diff --stat
git diff --cached --stat
git remote -v
git show --no-patch --format=fuller HEAD
```

Diese Befehle verändern normalerweise keine Versionierung oder den Inhalt der Arbeitsdateien.

### Gelb – kontrollierte Schreiboperationen

```bash
git fetch --prune origin
git switch -c docs/beispiel
git add -- docs/beispiel.md
git diff --cached --check
git diff --cached
git commit -m "docs: explain a feature"
git push -u origin docs/beispiel
```

Die Kommandos setzen beabsichtigte Dateipfade und einen gültigen Remote voraus. `git add -p` ist oft besser als ein pauschales Stage-All.

### Rot – nur mit expliziter Absicht

| Befehl | Risikogrund |
| --- | --- |
| `git reset --hard` | Kann Worktree- und Indexänderungen verwerfen |
| `git clean -fd` | Kann ungetrackte Dateien/Verzeichnisse löschen |
| `git push --force` | Kann gemeinsame Branch-Historie überschreiben |
| `git branch -D` | Löscht lokale Branchreferenz ohne Merge-Schutz |
| `git rebase -i` | Ändert Commit-IDs |
| `git tag -f` | Verschiebt Tag-Referenz; gefährlich für Releases |

**Merksatz:** Ein vermeintlich kaputter Remote-Branch wird **nicht** durch Force-Push „repariert“. Erst `fetch`, `log --graph`, `status` und Rulesets prüfen.

## Fragen und passende Kommandos

| Frage | Befehl |
| --- | --- |
| Welche Dateien landen im nächsten Commit? | `git diff --cached --stat` |
| Was wurde noch nicht gestaged? | `git diff` |
| Welcher Commit ist ausgecheckt? | `git rev-parse HEAD` |
| Woher kommt dieser Config-Wert? | `git config --show-origin --get SCHLÜSSEL` |
| Wieviele Commits fehlen gegenüber origin/main? | `git rev-list --left-right --count HEAD...origin/main` (nach fetch) |
| Welche Datei ist ignoriert? | `git check-ignore -v DATEI` |
| Welche Branches zeigen wohin? | `git branch -avv` |
| Wie finde ich verlorene Referenzen? | `git reflog` |
| Wie zeige ich die Datei aus einem Commit? | `git show HEAD:PFAD` |
| Was ist in einem Tag enthalten? | `git show --no-patch TAG` |
| Wie prüfe ich staged Whitespace? | `git diff --cached --check` |
| Wie teste ich eine bereits gebaute Software? | **Projektspezifischer** Testbefehl, z. B. `ctest --test-dir build --output-on-failure` |

## Einseitige Diagnose bei einem abgelehnten Push

```bash
git status -sb
git branch -vv
git fetch origin
git log --graph --decorate --oneline --all -20
git rev-list --left-right --count HEAD...origin/main
```

Danach klären: **Nicht-fast-forward**, fehlende Rechte, Branch-Ruleset, signierte Commits erforderlich oder CI-Gate? Die Fehlermeldung nicht durch History-Rewrite übergehen.

## Minimaler Abnahme-Satz

**Vor Commit:** `git status`, `git diff --check`, staged diff, lokale Tests.  
**Vor Push:** Branch und Remote, erwartete Commit-Liste.  
**Nach Push:** GitHub Actions/Checks auf **exakt** dem gepushten Commit kontrollieren.  
**Vor Release:** Security- und Release-Gate separat dokumentieren.

Weiterlesen: [Git Recovery](GIT-RECOVERY.md), [Git Workflows](GIT-WORKFLOWS.md).

Quelle: [Git Reference](https://git-scm.com/docs).
