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

## Vertiefung: welche Commit-Grafik entsteht?

```text
Ausgangslage:
A---B---C main
     \
      D---E feature

Merge-Commit:
A---B---C------M main
     \        /
      D-------E

Rebase des Feature-Branches auf main:
A---B---C main
         \
          D'---E' feature
```

Der zweite Graph verwendet **neue** Commit-Objekte `D'` und `E'`, obwohl die fachlichen Änderungen ähnlich sein können. Ein Rebase ist daher nicht nur „historisch aufräumen“, sondern kann bestehende Referenzen und Reviews beeinflussen.

### Ein Fast-forward ist eine Eigenschaft des Graphen

```bash
git merge-base main feature/beispiel
git log --left-right --graph --oneline main...feature/beispiel
git merge --ff-only feature/beispiel
```

Wenn `main` Vorfahr des Feature-Tips ist, reicht ein Verschieben des Branch-Zeigers. Andernfalls muss die Strategie ausdrücklich entschieden werden. `--ff-only` verändert bei Scheitern nicht heimlich die Historie.

### Konflikt sicher nachvollziehen

```bash
git status
git diff --name-only --diff-filter=U
git diff --check
git diff --ours -- PFAD
git diff --theirs -- PFAD
```

Diese Ansichten helfen bei unaufgelösten Konflikten. Je nach Merge/Rebase unterscheiden sich die Bedeutungen der Stufen 2 und 3; deshalb **immer** Kontext und Basis prüfen.

Ein Konfliktmarker ist keine abschließende Sicherheitsprüfung. Bei Code mit Bedingungen, SQL-Migrationen, Parsern oder CI-Workflows kann ein konfliktfreier Merge trotzdem logisch fehlerhaft sein.

### Drei Methoden, Konflikte zu vermeiden

1. Kleine, klar geschnittene Commits statt monatelanger Langläufer.
2. Große Umbenennungen und funktionale Änderungen getrennt durchführen.
3. Shared API-Änderungen früh dokumentieren und abhängige Projekte testen.

## Rebase: erlaubte und unerlaubte Situationen

| Situation | Einordnung |
| --- | --- |
| Eigene **noch nicht gepushten** Commits auf neuen Stand bringen | Gut kontrollierbar |
| Geöffnete PR mit nur einem Autor und bekannten Abnehmern | Mit Vorsicht, da Review-Commits wechseln |
| Bereits von Kollegen verwendeter Branch | Nur nach gemeinsamer Absprache |
| Geschütztes öffentliches **main** umschreiben | **Nicht empfohlen** |
| Fehler nach einem veröffentlichten Release korrigieren | Neuer Fix-/Revert-Commit und Patch-Release |

### Interactive Rebase (ausschließlich ungeteilte Historie)

```bash
git log --oneline -5
git rebase -i HEAD~3
```

Hier können Commits umgeordnet, zusammengefasst oder Nachrichten geändert werden. **Vorher** sicherstellen, dass drei Vorgänger existieren, der Branch nicht öffentlich geteilt ist und nichts Uncommittetes vorliegt. Zurückholen ggf. über `git reflog`; bei Unsicherheit vorab Rettungsbranch anlegen.

### Rebase nach Konflikt

```bash
git status
# Konflikte bewusst auflösen; Tests
git add -- PFAD
git rebase --continue
# Alternativ den gesamten Vorgang abbrechen:
git rebase --abort
```

`git rebase --skip` überspringt einen Commit und kann Änderungen stillschweigend verlieren. Nur nach Prüfung und dokumentierter Absicht nutzen.

## Cherry-pick – gezielt einen Commit übernehmen

```bash
git switch release/fix-test
git cherry-pick <commit-sha>
```

Nützlich für Backports auf unterstützte Versionen, jedoch nur in bewusst angelegten Branches. Konflikte und Abhängigkeiten prüfen. Der neue Commit kann eine andere SHA haben; nach erfolgreichem Cherry-pick Tests und Release-Gate vollständig durchlaufen.

```bash
git cherry-pick --abort
```

## Merge/Rebase nachweisen

**Vorher:** Branch-Tip und Diff. **Währenddessen:** Konflikte und Entscheidungen. **Nachher:** `git status` leer, erwartete Commits sichtbar, Tests bestanden, CI auf der tatsächlich gemergten Revision erfolgreich.

```bash
git log --graph --oneline --decorate --all -20
git diff --stat main...feature/beispiel
git status --short
```

Die Drei-Punkt-Diff-Notation vergleicht vom gemeinsamen Vorfahren; nicht mit `main..feature` im Log gleichsetzen.

## Übungsaufgabe
In einem Wegwerf-Repository zwei Branches mit derselben geänderten Zeile anlegen. Erst einen Konflikt mit `git merge` erzeugen und abbrechen; anschließend auf einem neuen Testbranch ein Rebase mit bewusst gelöster Konfliktstelle durchführen. Erfolg: Du kannst zeigen, warum sich Commit-IDs ändern.

Quellen: [Merge](https://git-scm.com/docs/git-merge), [Rebase](https://git-scm.com/docs/git-rebase), [Konflikte](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging).
