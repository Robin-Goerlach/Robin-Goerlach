# Git Basics – vom Arbeitsverzeichnis zum Commit

**Stand:** 2026-11-23 · **Status:** Technisches Nachschlagewerk · **Sprache:** Deutsch

> Diese Sammlung erklärt Git, nicht die verbindliche SASD-Projektpolitik. Vor destruktiven Befehlen immer Arbeitsverzeichnis, Branch, Status und Backup prüfen.

## Das Modell verstehen
**Working tree:** sichtbare Dateien. **Index (Staging Area):** Inhalt für den nächsten Commit. **Lokales Repository:** Objektdatenbank und Commits unter **.git**. **Remote:** separates Repository (z. B. auf GitHub). **HEAD:** aktuell ausgecheckter Commit/Branchbezug.

Ein Commit ist ein Snapshot des getrackten Projektstands und verweist auf Elterncommits. Eine Datei kann gleichzeitig im Working Tree, im Index und in **HEAD** unterschiedliche Inhalte haben.

## Sicheres Übungslabor
~~~bash
LAB_DIR="$(mktemp -d)"
cd "$LAB_DIR"
git init -b main
git config user.name "Git Lernlabor"
git config user.email "lernlab@example.invalid"
printf 'Zeile 1\n' > notizen.txt
git status --short
git add notizen.txt
git diff --cached
git commit -m "docs: start learning lab"
printf 'Zeile 2\n' >> notizen.txt
git diff
git add notizen.txt
git diff --cached
git commit -m "docs: extend notes"
git log --oneline --decorate --graph --all
~~~
Die lokale Übungsidentität gilt nur für dieses Repository. Die Adresse **example.invalid** ist absichtlich nicht zustellbar.

## Vier unterschiedliche Vergleiche
| Befehl | Bedeutung |
| --- | --- |
| git status --short | Überblick über Index und Worktree |
| git diff | Worktree gegenüber Index |
| git diff --cached | Index gegenüber HEAD |
| git diff HEAD | Worktree gegenüber HEAD (einschließlich gestagter Unterschiede) |

## Datei-Lebenszyklus
**Untracked → staged → committed → modified → staged**. **.gitignore** betrifft nur ungetrackte Dateien; bereits versionierte Dateien werden dadurch nicht rückwirkend entfernt.

~~~bash
git ls-files
git show --stat HEAD
git log --follow --oneline -- notizen.txt
git blame notizen.txt
~~~

## Git, GitHub und Sicherheit
Git speichert Historie lokal; GitHub ergänzt Zugriffskontrolle, Reviews, Actions, Releases und Meldungen. Ein Git-Commit allein beweist weder Codequalität noch Security-Review. Geheimnisse gehören nie in die Historie.

## Übung mit Erfolgskriterien
Verändere **notizen.txt** ein drittes Mal. Zeige den Unterschied mit **git diff**, stage ihn und überprüfe **git diff --cached**. Erfolg: **git status** ist nach dem Commit leer und **git log -3 --oneline** zeigt drei Commits.

## Vertiefung: das Drei-Zustände-Modell

Ein häufiger Fehler ist die Vorstellung, `git add` würde eine Datei „festschreiben“. Tatsächlich passiert Folgendes:

```text
Arbeitsverzeichnis (Working Tree)         Index (Staging Area)          HEAD-Commit
  notizen.txt  ─────── git add ───────>     notizen.txt  ── commit ──>    notizen.txt
   bearbeitbar                                  Snapshot                   unveränderlich
```

Wenn nach `git add` erneut editiert wird, enthält der Index **nicht automatisch** die jüngste Fassung. Darum vor dem Commit immer sowohl `git diff --cached` als auch `git status` prüfen.

### Ein Experiment mit zwei Fassungen derselben Datei

Nur im zuvor angelegten Übungsrepository ausführen:

```bash
printf 'Version A\n' > zustand.txt
git add zustand.txt
printf 'Version B\n' > zustand.txt
git status --short
git diff -- zustand.txt
git diff --cached -- zustand.txt
git show :zustand.txt
```

Erwartet wird **AM** in `git status --short`: Die neue Datei ist im Index aufgenommen (**A**), im Worktree aber nochmals geändert (**M**). `git show :zustand.txt` zeigt Version A, die Datei auf der Festplatte Version B. **Jetzt nicht versehentlich committen**, sondern für die Übung `git add -- zustand.txt` ausführen.

### Identität, Verfasser und Ersteller

Git-Commits enthalten einen **Author** (ursprünglicher Verfasser) und einen **Committer** (wer den Commit angelegt hat). Bei Merge/Rebase können diese unterschiedlich sein. Die Angaben sind Metadaten und stellen **ohne kryptografische Signatur** keinen belastbaren Identitätsnachweis dar.

```bash
git show -s --format='commit=%H%nAuthor=%an <%ae>%nCommitter=%cn <%ce>' HEAD
git cat-file -p HEAD
git rev-parse --show-toplevel
git rev-parse --is-inside-work-tree
```

**Objektmodell:** Blobs speichern Dateiinhalt, Trees Verzeichniszustände, Commits verknüpfen Tree und Eltern-Commits. Branch-Namen sind nur Referenzen. `git gc` und Aufbewahrungsfristen machen verlorene, unreferenzierte Objekte langfristig nicht garantiert wiederherstellbar.

## Dateien gezielt nachverfolgen

```bash
git ls-files                  # versionierte Pfade
git status --untracked-files=all
git check-ignore -v -- path/to/file
git log --all --oneline -- path/to/file
git show HEAD:path/to/file
```

Mit `--` lassen sich Pfade eindeutig von Revisionen trennen; das ist vor allem bei ungewöhnlichen Dateinamen wichtig. `git show HEAD:pfad` liest die alte Datei, ohne sie in den Worktree zurückzuschreiben.

### `.gitignore` korrekt verstehen

```gitignore
# Lokale Entwicklungsartefakte
build/
.cache/
.env
*.swp
```

- Regeln gelten für **ungetrackte** Inhalte. Ein bereits versioniertes `.env` wird durch nachträgliches Ignorieren nicht versteckt.
- `.gitignore` ist kein Secret-Scanner und kein Ersatz für das Widerrufen bereits publizierter Zugangsdaten.
- Eine `.gitignore` im Repository und `git config --global core.excludesFile ...` für persönliche Editor-Dateien dienen unterschiedlichen Zwecken.
- Zum Identifizieren der konkreten Regel: `git check-ignore -v -- DATEI`.

## Lernkontrolle

1. Was unterscheidet Worktree, Index und HEAD?
2. Warum kann `git diff` leer sein, obwohl der nächste Commit Änderungen enthält?
3. Wie sieht man den Inhalt einer Datei aus dem letzten Commit, ohne Dateien zu überschreiben?
4. Was bewirkt `.gitignore` bei einer bereits getrackten Datei?
5. Warum sind Author- und Committer-Text allein kein Identitätsbeweis?

**Praxisnachweis:** `git log --oneline -3` zeigt nachvollziehbare Commits, `git status --short` ist nach Abschluss leer und alle Befehle liefen ausschließlich in einem lokalen Testrepository.

Quelle: [Git-Referenz und Grundlagen](https://git-scm.com/book/en/v2/Git-Basics-Getting-a-Git-Repository).
