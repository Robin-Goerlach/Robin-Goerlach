# Git – Grundlagen, Arbeitsabläufe und Fehlerbehebung

**Stand:** 2026-11-23 · **Status:** Technisches Nachschlagewerk · **Sprache:** Deutsch

> Diese Sammlung erklärt Git, nicht die verbindliche SASD-Projektpolitik. Vor destruktiven Befehlen immer Arbeitsverzeichnis, Branch, Status und Backup prüfen.

Git ist ein **verteiltes Versionsverwaltungssystem**; GitHub ist ein möglicher Hosting-Dienst. Die meisten Befehle funktionieren daher auch offline und mit anderen Remote-Servern.

## Kapitel

| Thema | Dokument | Schwerpunkt |
| --- | --- | --- |
| Grundlagen | [GIT-BASICS.md](GIT-BASICS.md) | Worktree, Index, Repository, Commits |
| Einrichtung | [GIT-CONFIGURATION.md](GIT-CONFIGURATION.md) | Identität, Editor, SSH, Git-Konfiguration |
| Branches | [GIT-BRANCHING.md](GIT-BRANCHING.md) | Branches, Tracking, sichere Bereinigung |
| Alltag | [GIT-WORKFLOWS.md](GIT-WORKFLOWS.md) | Direkte main-Commits und Feature-Branches |
| Historie | [GIT-MERGE-REBASE.md](GIT-MERGE-REBASE.md) | Merge, Rebase, Konflikte |
| Versionen | [GIT-TAGS.md](GIT-TAGS.md) | Tags und Release-Referenzen |
| Wiederherstellung | [GIT-RECOVERY.md](GIT-RECOVERY.md) | Revert, Restore, Reflog und Backup |
| Kurzreferenz | [GIT-CHEATSHEET.md](GIT-CHEATSHEET.md) | Diagnose- und Alltagsbefehle |

## Lernweg
1. Ein **eigenes Wegwerf-Repository** für Beispiele anlegen.
2. Zuerst **GIT-BASICS**, **GIT-CONFIGURATION** und **GIT-WORKFLOWS** durcharbeiten.
3. Danach Merge/Rebase in einem **Testbranch** üben.
4. Vor destruktiven Aktionen **GIT-RECOVERY** lesen.

## Arbeitsregeln
- Vor Änderungen: **git status**, **git branch --show-current** und **git diff**.
- Vor einem Commit: **git diff --check**, anschließend **git diff --cached**.
- Niemals ohne ausdrückliche Absicht fremde Änderungen übernehmen.
- **git reset --hard**, **git clean -fd**, Force-Push und History-Rewrites gelten als gefährlich.
- Git ist kein vollständiges Backup für GitHub-Issues, PR-Kommentare, Rulesets oder Secrets.

## Quellen
- [Pro Git (offizielles frei verfügbares Buch)](https://git-scm.com/book/en/v2)
- [Git-Referenz](https://git-scm.com/docs)
- [GitHub Git Handbook](https://docs.github.com/en/get-started/using-git/about-git)

## Kompetenzstufen und Lernpfad

### Stufe 1: Sicher im Alltag (ca. 60–90 Minuten Übung)
1. [Basics](GIT-BASICS.md): Worktree, Index und Commits in einem Wegwerf-Repo unterscheiden.
2. [Configuration](GIT-CONFIGURATION.md): Identität und Zeilenenden kontrollieren.
3. [Cheatsheet](GIT-CHEATSHEET.md): fünf diagnostische Befehle auswendig nutzen.

**Erfolgskriterium:** Du erkennst ohne Raten, welche Dateien der nächste Commit enthalten wird und ob ein Push den richtigen Remote betrifft.

### Stufe 2: Zusammenarbeit und autonome Änderungen
1. [Branches](GIT-BRANCHING.md): Remote-Tracking und parallele Worktrees.
2. [Merge/Rebase](GIT-MERGE-REBASE.md): Konflikte kontrolliert lösen.
3. [Workflows](GIT-WORKFLOWS.md): Unterschiede zwischen direktem main und Feature-PR kennen.

**Erfolgskriterium:** Du kannst einen fehlgeschlagenen Push ohne Force-Push behandeln und einen ungeteilten Branch ohne Datenverlust umstrukturieren.

### Stufe 3: Veröffentlichungen und Rückfallstrategien
1. [Tags](GIT-TAGS.md): Annotated Tags und vertrauenswürdige Release-Referenzen.
2. [Recovery](GIT-RECOVERY.md): Revert, Reflog und Bundle-Restore.
3. [GitHub Release](../github/GITHUB-RELEASES.md) und [Release Security](../security/GITHUB-RELEASE-SECURITY.md).

**Erfolgskriterium:** Du kannst eine veröffentlichte Fehländerung durch einen neuen Commit neutralisieren und ein lokales Bundle in ein neues Verzeichnis klonen.

## Wichtige Unterschiede zu anderen Git-Clients

- **Git for Windows**: eigene Git-Binaries, Credential-Manager, Zeilenenden und Konfiguration.
- **WSL/Linux**: POSIX-Pfade, Dateirechte, Shell-Verhalten und SSH-Agent.
- **GitHub Desktop/IDE**: abstrahieren die Befehle, aber nicht deren Auswirkungen.
- **GitHub Repository Settings**: werden **nicht** durch `git config` oder lokale Branches verwaltet.

### Typische Migrationsfallen

Ein auf Windows erstellter Branch kann unter WSL Dateirechte, Zeilenenden und ausführbare Bits anders behandeln. In gemischten Projekten **.gitattributes**, `git ls-files --eol`, `git diff --summary` und `git config --show-origin` prüfen. Vor Massenrenormalisierung einen eigenen Branch erstellen.

## Didaktisches Prinzip

Alle gefährlichen Befehle werden nur in **Wegwerf-Repositories** erklärt; in produktiven Projekten erst Status, Diff, Backup und Rückfallmöglichkeit prüfen. Beispiele sind keine Aufforderung, die SASD-Repositories ungeprüft zu verändern.

## Pflege und Quellen

Die Quellen sind bevorzugt offizielle Git-Handbuch- und Referenzseiten. Prüfe bei gravierenden Versionswechseln die lokale Ausgabe von `git --version` und die zur Installation passende Dokumentation. Besonders `pull`-, Rebase- und Credential-Standards sind bewusst projektabhängig.
