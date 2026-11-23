# GitHub Rulesets – Branches und Tags schützen

**Stand:** 2026-11-23 · **Dokumenttyp:** GitHub-Nachschlagewerk · **Sprache:** Deutsch

> Menüs, Tarifumfang und API-Details können sich ändern. Die aktuelle Einstellung eines Repositories ist gesondert zu überprüfen; diese Seite nimmt keine Änderungen an anderen Repositories vor.

## Ziel
GitHub empfiehlt Schutzregeln für wichtige Branches. Entscheidend ist nicht, GitHub-Warnungen blind abzuarbeiten, sondern **versehentliche Löschungen/History-Rewrites zu blockieren**, ohne deinen Solo-Workflow unnötig zu erschweren.

## Empfehlung: persönliches DEV-Repository
Im Repository **Settings → Rules → Rulesets → New branch ruleset** (UI kann variieren):

| Feld | Empfohlener Wert | Warum? |
| --- | --- | --- |
| Ruleset-Name | main – minimal protection | Eindeutig |
| Enforcement | **Active** | Disabled/Evaluate schützt nicht zuverlässig |
| Target | Default branch: **main** | Nicht alle Branches blockieren |
| Restrict deletions | **Ein** | main nicht versehentlich löschen |
| Block force pushes | **Ein** | History nicht überschreiben |
| Require pull request | **Aus** | Direkte main-Commits bleiben möglich |
| Require status checks | Zunächst **Aus** | CI erfolgt zunächst nach Push |
| Restrict updates | **Aus** | Sonst riskant für direkten Push |
| Require signed commits | Optional, später | Nur nach funktionierender Signatur |
| Bypass | Möglichst leer | Keine unnötige Ausnahme |

**Achtung:** Rulesets können mit klassischen Branch-Protection-Regeln und anderen Rulesets gleichzeitig wirken. Ein korrektes DEV-Ruleset garantiert nicht, dass direktes Pushen funktioniert, wenn ein anderes Ruleset oder Branch Protection es blockiert.

## RELEASE-Profil (SASDGMBH)
- Schutz gegen Löschung und Force-Push wie oben.
- **PR-Pflicht** und erforderliche, stabil benannte CI-Checks erwägen/erzwingen.
- Bei nur einem Maintainer keine formale Pflicht zu fremder Zustimmung setzen, die niemand erfüllen kann. Fehlende unabhängige Reviews als bewusstes Risiko dokumentieren.
- Tags wie **v*** und Release-Berechtigungen gesondert kontrollieren.
- Bypass-Rollen sehr eng beschränken und im Review dokumentieren.

## Prüfen, ob eine Regel wirkt
1. In **Settings → Rules → Rulesets** Enforcement und Target kontrollieren.
2. Zusätzlich **Settings → Branches** auf klassische Branch Protection prüfen.
3. Ohne destruktive Test-Pushes in der GitHub-Oberfläche **Ruleset-Insights** bzw. den angezeigten Regelumfang ansehen.
4. CI-/PR-Anforderungen nur aktivieren, wenn die nötigen Checks zuverlässig existieren.
5. Status mit Datum und Repository in der [Checkliste](../security/GITHUB-SECURITY-CHECKLIST.md) vermerken.

## Was Rulesets nicht leisten
Sie sind kein Backup, kein Schadcode-Scanner und kein Schutz gegen kompromittierte Tokens mit anderen kritischen Rechten. Die Eingaben/Seite zum Erstellen können standardmäßig **Disabled** vorauswählen. **Active** vor dem Speichern bewusst prüfen.

## Typische Fehlkonfigurationen
**Require PR** bei direktem main-Workflow: Push wird blockiert. **Require status checks** mit nicht existierendem Check: Merge kann blockieren. **Restrict updates**: kann auch normale Updates untersagen. **Bypass für Admins**: Schutz gilt nicht zwingend für alle.

## Technische Entscheidung: gezielte minimale Schutzregeln statt Sicherheits-Theater

Es gibt mehrere Mechanismen, die **parallel** wirken können:

1. **Klassischer Branch Protection Rule:** ältere, einzelne Branch-Regel.
2. **Branch Ruleset:** Regelsatz mit Zielauswahl und möglicher Durchsetzung.
3. **Tag Ruleset:** kontrolliert bestimmte Tag-Operationen.
4. **Organisationsregeln:** können zusätzliche Grenzen setzen.
5. **GitHub-Rechte und Apps:** legen unabhängig davon fest, wer überhaupt pushen darf.

**Wichtig:** Die API-Liste mit `[]` zeigt nur, dass über den abgefragten Repository-Ruleset-Endpunkt keine Rulesets zurückgegeben wurden; daraus folgt **nicht**, dass keinerlei klassischer Branchschutz existiert oder dass Konto/Organisation komplett ungeprüft sind.

## Schritt-für-Schritt-Plan für ein persönliches Repo

### Vor der Einrichtung
- Repository und Eigentümer auf GitHub kontrollieren.
- Prüfen, ob `main` wirklich Default-Branch ist.
- Unter **Settings → Rules → Rulesets** und **Settings → Branches** mögliche bestehende Regeln anschauen.
- Sicherstellen, dass aktuelle Arbeiten gespeichert und ein normaler Push möglich sind.

### Ruleset erstellen
1. **Settings → Rules → Rulesets → New branch ruleset** öffnen.
2. Name `main – minimal protection`.
3. Bei **Enforcement** `Active` wählen; `Disabled` ist keine Durchsetzung. Falls `Evaluate` angeboten wird, ist dies nur eine Auswertung, keine harte Blockade.
4. **Target branches**: Default-Branch bzw. `main` auswählen; Auswahl vor dem Speichern erneut prüfen.
5. **Restrict deletions**: aktivieren.
6. **Block force pushes**: aktivieren.
7. **Require pull request**, **Restrict updates**, **Require status checks**: für das bewusste DEV-Direkt-push-Profil zunächst **aus**.
8. Bypass-Liste nur bei begründetem Bedarf; zusätzliche Ausnahmen reduzieren den Schutz.
9. Speichern und anschließend Status/Target sowie aktive Regeln erneut ansehen.

Die UI kann Pflichtfelder anders gruppieren. Ob ein Schalter im persönlichen Tarif/Repository verfügbar ist, anhand der aktuellen Oberfläche und [GitHub-Dokumentation](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets) kontrollieren.

### Nach der Einrichtung: ohne destruktiven Test
- Ruleset-Liste auf **Active** prüfen.
- Zielbranch/Include-/Exclude-Regeln lesen.
- In den Ruleset-Insights mögliche Blockaden nach einem normalen Test-Commit erkennen.
- Mit einem **harmlosen** Dokumentationscommit testen, dass reguläre Pushes weiterhin erlaubt sind.
- **Niemals** durch einen echten Force-Push oder Branch-Löschversuch auf `main` testen.

## Statuschecks, PRs und Merge Queue

Wenn ein RELEASE-Repository `Require status checks` nutzt, müssen die Check-Namen stabil sein und die Check-Quelle ggf. korrekt festgelegt werden. Ein umbenannter oder nicht ausgelöster Workflow kann Merges verhindern. Ein Workflow, der nur auf `push` nach **main** reagiert, liefert nicht zwingend den Check, der **vor** dem Merge gefordert wird.

Bei Merge Queue zusätzlich prüfen, ob die CI `merge_group`-Events abdeckt, wenn dieser Mechanismus aktiviert ist. Sonst wartet die Queue möglicherweise auf Checks, die nie laufen.

## Review-Regeln für eine Person

Eine PR-Pflicht ohne **Pflicht zu fremdem Review** ist technisch nicht identisch mit Vier-Augen-Prinzip. Sinnvolle Zusatzmaßnahmen:
- Prüfliste für Diff, Tests, Abhängigkeiten, Workflows und Signaturen.
- Dokumentierte Freigabe vor Releases.
- CI-Prüfung auf dem zu mergenden Commit.
- Bei Hochrisikoänderungen eine tatsächliche externe Codeprüfung organisieren, statt formale Selbstfreigabe als unabhängig auszugeben.

## Tags und gefährliche Ausnahmen

Tag-Rulesets können verhindern, dass veröffentlichte `v*`-Tags nachträglich gelöscht oder verschoben werden. Sie sollten vor einer Release-Automatisierung auf deren Auswirkungen geprüft werden. Schreibende Bots, Release-Actions oder Admin-Bypass können in Konflikt mit den Regeln geraten. **Bypass-Rechte sind Teil des Bedrohungsmodells, keine bloße Komfortoption.**

## Dokumentation einer Entscheidung (Vorlage)

```text
Repository:
Repository-Typ: DEV | RELEASE
Default-Branch:
Ruleset-Name / Ziel / Enforcement:
Restrict deletions:
Block force pushes:
Require PR:
Require status checks (Namen):
Restrict updates:
Bypass-Rollen:
Klassische Branch Protection:
Geprüft am / durch:
Offene Abweichungen und nächste Prüfung:
```

**Sicherheitsgewinn:** Schutz vor unbeabsichtigter Ref-Löschung und erzwungener Änderung. **Nicht abgedeckt:** Schadcode in legitimen Commits, gestohlene Zugangsdaten für andere Berechtigungen, Fehler in Actions, Abhängigkeitsrisiken und fehlende Offsite-Backups.

Quellen: [Rulesets erstellen](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository), [Rulesets-Regeln](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets), [Rulesets verwalten](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/managing-rulesets-for-a-repository).
