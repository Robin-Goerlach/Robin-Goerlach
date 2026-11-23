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

Quellen: [Rulesets erstellen](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository), [Rulesets-Regeln](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets), [Rulesets verwalten](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/managing-rulesets-for-a-repository).
