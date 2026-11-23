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

Quellen: [About PRs](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests), [Merge Methods](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/about-pull-request-merges).
