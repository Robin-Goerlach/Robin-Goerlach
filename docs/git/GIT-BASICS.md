# Git Basics – vom Arbeitsverzeichnis zum Commit

**Stand:** 2026-11-23 · **Status:** Technisches Nachschlagewerk · **Sprache:** Deutsch

> Diese Sammlung erklärt Git, nicht die verbindliche SASD-Projektpolitik. Vor destruktiven Befehlen immer Arbeitsverzeichnis, Branch, Status und Backup prüfen.

## Das Modell verstehen
**Working tree:** sichtbare Dateien. **Index (Staging Area):** Inhalt für den nächsten Commit. **Lokales Repository:** Objektdatenbank und Commits unter **.git**. **Remote:** separates Repository (z. B. auf GitHub). **HEAD:** aktuell ausgecheckter Commit/Branchbezug.

Ein Commit ist ein Snapshot des getrackten Projektstands und verweist auf Elterncommits. Eine Datei kann gleichzeitig im Working Tree, im Index und in **HEAD** unterschiedliche Inhalte haben.

## Sicheres Übungslabor
~~~bash
mkdir -p /tmp/git-lernlabor && cd /tmp/git-lernlabor
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
**Untracked → staged → committed → modified → staged**. **.gitignore** betrifft gewöhnlich nur ungetrackte Dateien; bereits versionierte Dateien werden dadurch nicht rückwirkend entfernt.

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

Quelle: [Git-Referenz und Grundlagen](https://git-scm.com/book/en/v2/Git-Basics-Getting-a-Git-Repository).
