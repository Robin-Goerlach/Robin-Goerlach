# Git Recovery – Fehler sicher rückgängig machen

**Stand:** 2026-11-23 · **Status:** Technisches Nachschlagewerk · **Sprache:** Deutsch

> Diese Sammlung erklärt Git, nicht die verbindliche SASD-Projektpolitik. Vor destruktiven Befehlen immer Arbeitsverzeichnis, Branch, Status und Backup prüfen.

## Die erste Regel: Diagnose vor Reparatur
~~~bash
pwd
git status --short
git branch --show-current
git log -8 --oneline --decorate
git reflog -10
git diff
git diff --cached
~~~
Vor riskanten Änderungen gegebenenfalls **git branch rescue/vor-reparatur HEAD** anlegen und nicht committete Arbeit separat sichern.

## Welcher Fehler liegt vor?
| Problem | Schon geteilt? | Meist passende Maßnahme |
| --- | --- | --- |
| Einzelne nicht gestagte Datei verwerfen | Nein | git restore -- PFAD (**löscht Änderungen**) |
| Datei aus Index nehmen, Änderungen behalten | Nein | git restore --staged -- PFAD |
| Letzten Commit korrigieren | Nein | git commit --amend (ändert Commit-ID) |
| Fehlerhaften Commit auf main neutralisieren | Ja | git revert COMMIT |
| Gelöschten Commit wiederfinden | Häufig | git reflog, dann gezielter rescue-Branch |
| Ungeprüfte ungetrackte Dateien löschen | – | **Nicht blind git clean -fd** |

## Revert ist für geteilte Historie meist sicherer
~~~bash
git switch main
git status --short
git revert <commit-sha>
git log -2 --oneline
~~~
Der Revert erzeugt einen neuen Commit. Bei Merge-Commits muss dessen Mainline-Elternbezug gesondert geklärt werden; nicht blind den Parameter **-m** verwenden.

## Restore und Reset unterscheiden
~~~bash
git restore --staged -- docs/beispiel.md
git restore --source=HEAD -- docs/beispiel.md
~~~
**git restore --source=HEAD** verwirft *lokale uncommittete Änderungen* an dieser Datei; nur nach Sichtung benutzen. **git reset --hard** überschreibt Index und Worktree; **git clean -fd** kann ungetrackte Inhalte dauerhaft entfernen. Beide fehlen bewusst aus der Standardroutine.

## Reflog und Backups
~~~bash
git reflog show HEAD
git fsck --no-reflogs --unreachable
git bundle create ../repo-backup.bundle --all
git bundle verify ../repo-backup.bundle
~~~
Ein Bundle sichert die angegebenen Git-Referenzen und deren Objekte, **nicht** GitHub-Issues, Rulesets, Releases, Wiki oder ungepushte Working-Tree-Dateien. Die Wiederherstellung muss praktisch getestet werden. Reflog ist lokal und nicht unbegrenzt verfügbar.

## Vorfall mit einem Secret
Token/Schlüssel **sofort widerrufen/rotieren**, Zugang und Zugriffslogs prüfen. Entfernen aus HEAD oder ein History-Rewrite allein machen ein bereits veröffentlichtes Secret nicht wieder geheim.

## Vertiefung: Wiederherstellung nach Fehlertyp

Der Ausgangspunkt ist immer **welcher Zustand ist betroffen**: Worktree, Index, lokaler Commit, bereits veröffentlichter Commit, Branchreferenz oder GitHub-Metadaten.

### Fall A – Datei wurde verändert, aber noch nicht gestaged

```bash
git diff -- docs/datei.md
git restore --source=HEAD -- docs/datei.md
```

Der zweite Befehl **verwirft die ungestagten Änderungen der Datei**; nur nach ausdrücklicher Entscheidung. Bei nicht gesicherten wichtigen Änderungen zuerst eine Kopie **außerhalb des Repositories** anlegen.

### Fall B – Datei gestaged, aber noch nicht committed

```bash
git diff --cached -- docs/datei.md
git restore --staged -- docs/datei.md
git status --short
```

Dadurch wird die Änderung lediglich aus dem **Index entfernt**, sie bleibt gewöhnlich im Worktree erhalten. Die beiden Operationen `--staged` und `--source=HEAD` deshalb nicht verwechseln.

### Fall C – ein Commit ist bereits in main veröffentlicht

```bash
git status --short
git log --oneline -5
git revert <commit-sha>
# Konflikte/Test/CI sorgfältig kontrollieren
git push origin main
```

`revert` bewahrt die gemeinsame Historie. Beim Revert eines **Merge-Commits** muss das Elternverhältnis ausdrücklich bestimmt werden; keine pauschale `-m 1`-Empfehlung.

### Fall D – Branch gelöscht oder `HEAD` verstellt

```bash
git reflog -20
git show --no-patch <reflog-commit-sha>
git branch rescue/wiederhergestellt <reflog-commit-sha>
git log --oneline -5 rescue/wiederhergestellt
```

Reflogs sind lokal und zeitlich begrenzt; sie ersetzen kein Backup. Vorher anhand von Datum, Commit-Meldung und Diff sicherstellen, dass die gesuchte Revision korrekt ist.

### Fall E – Datei ist nicht versioniert

Git kann ungetrackte, endgültig gelöschte Dateien im Allgemeinen **nicht** wiederherstellen. Prüfe Editor-Local-History, Dateisystem-Snapshots, Backup oder Papierkorb. **git clean -fd** ist deshalb in der Standardroutine tabu.

## Stash – für vorübergehende Änderungen, nicht als Backup

```bash
git status --short
git stash push -m "temp: wip before update"
git stash list
git stash show --stat stash@{0}
git stash apply stash@{0}
```

`apply` behält den Stash-Eintrag; `pop` entfernt ihn bei Erfolg. Stash kann Konflikte verursachen. `-u` schließt ungetrackte Dateien mit ein und kann **vertrauliche Dateien** in Git-Objekte aufnehmen; nur bewusst einsetzen.

## Ein echtes Git-Backup erstellen und prüfen

Für ein bereits existierendes Git-Repository:

```bash
git status --short
BACKUP_DIR="$(mktemp -d)"
git bundle create "$BACKUP_DIR/repository.bundle" --all
git bundle verify "$BACKUP_DIR/repository.bundle"
git bundle list-heads "$BACKUP_DIR/repository.bundle"
```

Die Bundle-Datei befindet sich im **temporären** Verzeichnis; für einen echten Backup-Prozess muss sie zusätzlich auf dauerhaftes, unabhängiges und geeignet gesichertes Medium übertragen werden. Ein Bundle umfasst nur die mit den Refs erreichbaren Git-Objekte. Worktree-Änderungen und große Teile der GitHub-Plattformdaten fehlen.

**Wiederherstellungstest** in einem separaten Verzeichnis:

```bash
RESTORE_DIR="$(mktemp -d)"
git clone "$BACKUP_DIR/repository.bundle" "$RESTORE_DIR/restore"
git -C "$RESTORE_DIR/restore" log -1 --oneline
git -C "$RESTORE_DIR/restore" branch -a
git -C "$RESTORE_DIR/restore" tag -l
```

Bei Bundles ohne zur Default-Ref passende HEAD-Zuordnung muss beim Klonen ggf. ausdrücklich ein Branch ausgewählt werden; Tags und Branches nach dem Import prüfen. **Achtung:** Temporäre Verzeichnisse können automatisch gelöscht werden.

## GitHub-Metadaten gesondert sichern

Ein Mirror oder Bundle exportiert **nicht** alle:
- Issues und Diskussionen;
- PR-Reviews, Check-Run-Protokolle und Release-Downloads;
- Repository-Secrets, Rulesets, App-Berechtigungen oder Environments;
- GitHub Projects, Wiki (ggf. eigenes Git-Repository), Attestations.

Für wichtige Inhalte Exportverfahren und Aufbewahrungsfristen dokumentieren; Secrets nicht im Klartext archivieren.

## Security-Incident ist kein normaler Undo-Vorgang

Bei öffentlich gewordenen Zugangsdaten:
1. Secret **sofort sperren/rotieren**.
2. Logs und Berechtigungen auf Missbrauch prüfen.
3. Historie, Issues, Artifacts und Forks auf weitere Kopien prüfen.
4. History-Rewrite/Cache-Bereinigung nur als **zusätzliche** Maßnahme nach Koordination, keinesfalls als Ersatz für Rotation.
5. Neue gültige Credentials nur sicher übergeben.

## Übung mit Erfolgskriterien
In einem Wegwerf-Repo absichtlich eine Datei verändern, stagen, unstage, wieder stagen und committen. Danach den Commit `git revert`-basiert neutralisieren. Anschließend ein Bundle erzeugen und die Historie in einem neuen Ordner überprüfen. **Kein `reset --hard` erforderlich.**

Quellen: [Revert](https://git-scm.com/docs/git-revert), [Restore](https://git-scm.com/docs/git-restore), [Reflog](https://git-scm.com/docs/git-reflog), [Bundle](https://git-scm.com/docs/git-bundle).
