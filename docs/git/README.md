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
