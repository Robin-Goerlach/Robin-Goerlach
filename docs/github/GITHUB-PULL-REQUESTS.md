# GitHub Pull Requests – Review als nachvollziehbarer Prozess

**Stand:** 2026-11-23 · **Dokumenttyp:** GitHub-Nachschlagewerk · **Sprache:** Deutsch

> Menüs, Tarifumfang und API-Details können sich ändern. Die aktuelle Einstellung eines Repositories ist gesondert zu überprüfen; diese Seite nimmt keine Änderungen an anderen Repositories vor.

## Wofür PRs gut sind
Ein Pull Request dokumentiert Änderungsvorschläge, Diskussionen, CI-Status und Merges. Er ist auch ohne mehrere Entwickler sinnvoll, wenn die Änderung groß oder riskant ist. Aber: **Ein Selbstreview ersetzt keinen unabhängigen Review**.

## Praktischer Ablauf
~~~bash
git switch main
git fetch origin
git merge --ff-only origin/main
git switch -c feature/new-toolkit-widget
# Änderung, lokale Tests, kontrolliertes Staging, Commit
git push -u origin feature/new-toolkit-widget
~~~
Auf GitHub **Compare & pull request** verwenden. Base muss **main**, Head der Feature-Branch sein. In der Beschreibung Problem, technische Änderungen, Testnachweis, Risiken und ggf. Rückrollplan erklären.

## Prüfpunkte vor Merge
- Diff auf unbeabsichtigte Dateien, Dateirechte, Secrets, generierte Binärdateien.
- Neue/aktualisierte Abhängigkeiten, Lizenzen, Downloadquellen, Shell-Skripte.
- Neue oder veränderte **.github/workflows/**, Token-Rechte und Trigger.
- Tests: was wurde tatsächlich ausgeführt und was nicht?
- API-/ABI-Kompatibilität, Dokumentation und Migrationshinweise.
- Checks gelten nur, wenn sie auf der relevanten Commit-Revision gelaufen sind.

## Merge-Strategien
| Strategie | Historie / Besonderheit |
| --- | --- |
| Merge commit | Erhält Branch-Topologie |
| Squash and merge | Ein Commit auf Base aus PR-Inhalt |
| Rebase and merge | Spielt Commits neu auf Base; IDs können sich ändern |

Eine verpflichtende PR-Regel allein reicht nicht: Bei RELEASE-Repositories zusätzlich erforderliche Statuschecks, Branch-Schutz und Review-Verantwortung festlegen. Nicht eine zweite Person voraussetzen, wenn faktisch nur ein Maintainer existiert; dann bewusstes Risiko und alternative Qualitätskontrollen dokumentieren.

## Konflikte / veralteter Branch
~~~bash
git switch feature/new-toolkit-widget
git fetch origin
git merge origin/main
# Konflikte lösen, Tests wiederholen, pushen
~~~
Oder lokalen Rebase bewusst nur dann, wenn History-Rewrite zulässig ist.

## PRs aus Forks
Fork-Beiträge sind fremder Code. CI bei PR-Ereignissen mit Least Privilege; keine Secrets in untrusted Jobs; kein riskantes **pull_request_target** mit ausgechecktem PR-Code. Nach Merge Hinweise für Contributors geben und Branch aufräumen.

## Vertiefung: PR-Reviews als technische Qualitätssicherung

### Vor dem Pull Request

Ein PR sollte ein Problem lösen, keinen ganzen Entwicklungstag abbilden. Für große Änderungen mehrere logisch getrennte PRs planen, sofern sich die Integrationsreihenfolge kontrollieren lässt.

```bash
git status --short
git branch --show-current
git fetch origin
git log --left-right --oneline origin/main...HEAD
git diff --stat origin/main...HEAD
git diff --check origin/main...HEAD
```

**Wichtig:** Diese Prüfung betrachtet die Git-Differenz. Ergänzend die **PR-Ansicht** auf GitHub anschauen, da dort Zielbranch, Reviewer, Automationen und CI abgebildet sind. Nach einem Rebase kann die Differenz anders wirken.

### Empfohlene PR-Beschreibung

```markdown
## Motivation
Welches konkrete Problem wird behoben?

## Änderung
Welche Komponenten und Verhaltensänderungen sind enthalten?

## Risiken
API/ABI, Security, Datenmigration, Parallelität, Lizenz, neue Abhängigkeiten.

## Nachweis
- [ ] Lokaler Build und Tests
- [ ] CI für aktuellen Head/PR-Merge-Commit erfolgreich
- [ ] Neue/aktualisierte Fremdkomponenten geprüft
- [ ] Dokumentation aktualisiert

## Rückfall
Wie wird bei einem Problem ein Korrektur- oder Revert-Commit erstellt?
```

### Welche Reviews besonders wichtig sind

| Datei oder Änderung | Zusätzliche Prüfung |
| --- | --- |
| `.github/workflows/` | Token-Scopes, Trigger, externe Actions, Secrets |
| `CMakeLists.txt` / Buildsystem | Fremd-Downloads, Buildoptionen, Tests |
| Parser / Dateiformat | Grenzfälle, Fehlerpfade, Fuzzing |
| Auth / Netzwerk / Firewall | Threat Model, sichere Defaults |
| Lizenz / Attribution | Rechte der übernommenen Inhalte |
| Releases / Packaging | Versionsreferenzen, Provenance |
| Dokumentation | Korrekte Links und ausführbare Befehle |

## Check-Status ist commitgebunden

Ein grünes Ergebnis vom **alten Commit** kann nach einer neuen Änderung überholt sein. Prüfen, auf welche SHA der Check gelaufen ist. Falls erforderlich „Require branches to be up to date“ oder Merge Queue passend konfigurieren; zusätzlich muss die CI den entsprechenden Auslöser unterstützen.

### Merge Queue – wichtiger Sonderfall

Bei aktivierter GitHub Merge Queue laufen Checks ggf. über **`merge_group`**. Ein Workflow, der ausschließlich `pull_request` enthält, kann bei einem verpflichtenden Check die Queue blockieren. Die konkrete Workflow-Datei entsprechend erweitern und mit echten Queue-Läufen validieren.

## Merge-Methoden bewusst auswählen

**Merge Commit:** Branch-Topologie bleibt sichtbar; vereinfacht manchmal die Nachverfolgung, macht den Graphen komplexer.  
**Squash:** ein logisch gebündelter Commit auf main; Einzelcommits aus dem Branch sind dort nicht separat vorhanden.  
**Rebase Merge:** flache Historie, neue Commit-IDs möglich; geeignet, wenn das Team diese Konsequenz versteht.

Die organisatorische Regel zur Merge-Methode gehört in den **SASD-Development-Standard**, nicht als unbewusste Vorgabe in dieses persönliche Nachschlagewerk.

## Solo-Maintainer ohne Review-Theater

- Für kleine risikoarme Änderungen direkter main-Workflow im DEV-Profil.
- Für komplexe Änderungen PR mit strukturierter Selbstkontrolle.
- Für Releases Pflichtchecks, Archiv der Freigabeentscheidung und ggf. externer Review bei besonderem Risiko.
- Nicht fälschlich den Eindruck eines unabhängigen Vier-Augen-Prinzips erwecken.

Eine erzwungene Review-Regel kann einen alleinigen Maintainer blockieren, wenn niemand berechtigt ist, die notwendige Freigabe zu erteilen. Das ist eine **Designentscheidung** der Repository-Regeln, kein Zeichen höherer Sicherheit.

## Nach dem Merge

```bash
git switch main
git fetch origin
git merge --ff-only origin/main
git status --short
git log --oneline --decorate -6
```

Danach prüfen: Wurde der erwartete PR gemergt? Ist der CI-Status auf dem Merge-Commit grün? Bleiben Release-Notes oder Folgetests? Erst anschließend Branch löschen.

Quellen zur Vertiefung: [Reviewing proposed changes](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests), [Merge Queue](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue).

Quellen: [About PRs](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests), [Merge Methods](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/about-pull-request-merges).
