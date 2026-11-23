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

## Vertiefung: Remote-Tracking und Synchronisationsfehler

**`origin/main` ist kein Live-Zugriff auf GitHub.** Es ist der lokale, zuletzt abgerufene Verweis auf den Remote-Branch. Deshalb vor einer Aussage wie „mein main ist aktuell“ zuerst `git fetch origin` verwenden.

```bash
git fetch --prune origin
git branch -vv
git log --left-right --graph --oneline HEAD...origin/main
git rev-list --left-right --count HEAD...origin/main
```

Das letzte Kommando zeigt zwei Zahlen: lokal- und remote-exklusive Commits. Ein **Fast-forward** ist nur möglich, wenn die Historie entsprechend linear fortgesetzt werden kann.

### Praktischer Entscheidungsbaum bei Push-Ablehnung

1. **Nicht Force-Pushen.** `git status` und `git branch --show-current` kontrollieren.
2. `git fetch origin` ausführen.
3. `git log --graph --oneline --decorate --all -20` und `git rev-list --left-right --count HEAD...origin/main` ansehen.
4. Wenn **main** nicht divergiert: `git merge --ff-only origin/main`.
5. Wenn sowohl lokal als auch remote neue Commits existieren: Inhalts- und Integrationsstrategie klären (Merge oder geplanter Rebase eigener, ungeteilter Commits).
6. Tests erneut ausführen, dann normal pushen. Bei erneutem Fehler prüfen, ob Rulesets oder Rechte statt Historie die Ursache sind.

### Sicher mit mehreren Aufgaben parallel: Git Worktrees

`git worktree` bietet separate Arbeitsverzeichnisse im **selben** Repository, ohne die Git-Historie doppelt zu klonen.

```bash
git worktree list
git switch main
git worktree add -b feature/docs-review ../repo-docs-review main
git worktree list --porcelain
```

Diese Befehle **nur** aus einem passenden Repository heraus ausführen und das Nachbarverzeichnis vorher prüfen. In einem Worktree kann ein Branch normalerweise nur einmal gleichzeitig ausgecheckt sein. Zum Aufräumen zunächst alle Änderungen sichern und dann:

```bash
git worktree remove ../repo-docs-review
git branch -d feature/docs-review
```

`git worktree remove` wird bei ungesicherten Änderungen regulär verweigert. Keine `--force`-Variante benutzen, solange Inhalt unklar ist.

### Detached HEAD und Experimente

```bash
git switch --detach <commit-sha>
git status -sb
# Zum Zurückwechseln:
git switch main
```

Im detached HEAD zeigen neue Commits nicht automatisch auf einen normalen Branch. Wenn dort Arbeit entstanden ist, **vor dem Verlassen** einen neuen Branch anlegen: `git switch -c rescue/meine-arbeit`.

### Branch-Namen und Aufbewahrung

- `feature/...`, `fix/...`, `docs/...` bezeichnen Absicht, nicht einen Sicherheitsstatus.
- Branches nicht als dauerhaftes Backup verwenden; unabhängige Remotes/Bundles und Restoretests einplanen.
- Release-Tags sind von kurzlebigen Arbeitsbranches zu unterscheiden.
- `git branch -d` kann einen Branch löschen, wenn Git die Arbeit als integriert betrachtet; vorher im Zweifel Diff und Commits prüfen.

## Laboraufgabe (ohne Produktionsremote)

Erstelle ein lokales Bare-Repository als Test-Remote:

```bash
LAB="$(mktemp -d)"
git init --bare "$LAB/remote.git"
git clone "$LAB/remote.git" "$LAB/one"
git -C "$LAB/one" switch -c main
git -C "$LAB/one" config user.name "Lab"
git -C "$LAB/one" config user.email "lab@example.invalid"
printf 'start\n' > "$LAB/one/file.txt"
git -C "$LAB/one" add file.txt
git -C "$LAB/one" commit -m "lab: start"
git -C "$LAB/one" push -u origin main
git clone -b main "$LAB/remote.git" "$LAB/two"
```

Nun in beiden Klonen jeweils **unterschiedliche** Commits anlegen und einen Push-Konflikt kontrolliert beobachten. Erfolg: Du verstehst die Ablehnung eines nicht-fast-forward Pushes und behebst sie ohne `--force`. Die Adresse **example.invalid** ist absichtlich nicht zustellbar.

Quelle: [Git Branching](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell).
