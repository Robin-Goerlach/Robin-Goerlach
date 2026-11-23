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
Vor riskanten Änderungen gegebenenfalls **git branch rescue/2026-11-23 HEAD** anlegen und nicht committete Arbeit separat sichern.

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

Quellen: [Revert](https://git-scm.com/docs/git-revert), [Restore](https://git-scm.com/docs/git-restore), [Reflog](https://git-scm.com/docs/git-reflog), [Bundle](https://git-scm.com/docs/git-bundle).
