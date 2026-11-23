# GitHub – Repositories, Automatisierung und Zusammenarbeit

**Stand:** 2026-11-23 · **Dokumenttyp:** GitHub-Nachschlagewerk · **Sprache:** Deutsch

> Menüs, Tarifumfang und API-Details können sich ändern. Die aktuelle Einstellung eines Repositories ist gesondert zu überprüfen; diese Seite nimmt keine Änderungen an anderen Repositories vor.

GitHub ergänzt Git um Hosting, Code-Reviews, Issues, Projektverwaltung, Automatisierung, Releases und Security-Werkzeuge. Die Git-Grundlagen stehen in [../git/README.md](../git/README.md); die Sicherheitscheckliste in [../security/GITHUB-SECURITY-CHECKLIST.md](../security/GITHUB-SECURITY-CHECKLIST.md).

## Kapitel
| Dokument | Thema |
| --- | --- |
| [GITHUB-REPOSITORIES.md](GITHUB-REPOSITORIES.md) | Erstellung, Sichtbarkeit, Repository-Lebenszyklus |
| [GITHUB-ACTIONS.md](GITHUB-ACTIONS.md) | CI/CD, Berechtigungen und Workflow-Sicherheit |
| [GITHUB-PULL-REQUESTS.md](GITHUB-PULL-REQUESTS.md) | Reviews und kontrolliertes Zusammenführen |
| [GITHUB-RULESETS.md](GITHUB-RULESETS.md) | Branch-/Tag-Schutz ohne unnötige Reibung |
| [GITHUB-RELEASES.md](GITHUB-RELEASES.md) | Releases, Tags, Artefakte und Nachweise |
| [GITHUB-DEPENDABOT.md](GITHUB-DEPENDABOT.md) | Updates, Alerts und Abdeckungsgrenzen |
| [GITHUB-CODEX-WORKFLOW.md](GITHUB-CODEX-WORKFLOW.md) | Praktische Grenzen für KI-Agenten |

## Entscheidung: zwei Schutzprofile
**DEV:** Solo-Entwicklung, direktes Committen nach **main** erlaubt; Lösch- und Force-Push-Schutz empfohlen. **RELEASE:** offizielle SASDGMBH-Veröffentlichung mit reproduzierbarem Build, Nachweisen und bewusstem Freigabeprozess. Eine PR-Pflicht darf nicht durch eine unmöglich erfüllbare Pflicht zu einer zweiten Person blockiert werden.

## Die drei Ebenen nicht vermischen
1. **Git:** lokale Commits, Branches, Tags.
2. **GitHub Repository Settings:** Schutzregeln, Berechtigungen, Actions, Code Security.
3. **Projektpolitik:** Konventionen und Anforderungen etwa im SASD-Development-Standard.

Eine Dokumentationsdatei im Profil-Repository überträgt keine GitHub-Settings auf andere Repositories.

## Vom GitHub-Menü zur sicheren Entwicklungsroutine

Die Kapitel sind absichtlich in der Reihenfolge aufgebaut, in der ein Repository entsteht, bearbeitet, geprüft und veröffentlicht wird:

```text
Repository erstellen
  → Zugriffsrechte und Security Policy
  → Branch-/Tag-Rulesets
  → Branch oder direkter main-Workflow
  → Pull Request / Code Review
  → GitHub Actions und Dependabot
  → Release-Gate und Artefakte
  → Monitoring, Advisory, Patch, Recovery
```

### Lernpfad für ein öffentliches C++-Projekt

1. [Repositories](GITHUB-REPOSITORIES.md): Owner, Visibility, Lizenz und Projektstatus prüfen.
2. [Rulesets](GITHUB-RULESETS.md): minimalen Schutz für `main` planen; direkte Commits nicht unbeabsichtigt blockieren.
3. [Actions](GITHUB-ACTIONS.md): Rechte, Trigger, Runner, Tests und SHA-Pinning verstehen.
4. [Dependabot](GITHUB-DEPENDABOT.md): Abhängigkeitsgraph mit tatsächlicher CMake-/vcpkg-Nutzung vergleichen.
5. [Pull Requests](GITHUB-PULL-REQUESTS.md): bei größeren Änderungen und Codex Branch-/Review-Ablauf üben.
6. [Codex Workflow](GITHUB-CODEX-WORKFLOW.md): Scope und Beweislage für autonome Entwicklung.
7. [Releases](GITHUB-RELEASES.md): Build, Artefakte und Freigabe dokumentieren.

### Abgrenzung zu Git und Security

| Frage | Beste Anlaufstelle |
| --- | --- |
| „Was tut `git merge`?“ | [Git-Handbuch](../git/README.md) |
| „Wie erzwinge ich erforderliche CI-Checks?“ | [GitHub Rulesets](GITHUB-RULESETS.md) |
| „Was darf mein Codex-Auftrag verändern?“ | [Codex Workflow](GITHUB-CODEX-WORKFLOW.md) |
| „Wie belege ich den Security-Status?“ | [Security Checklist](../security/GITHUB-SECURITY-CHECKLIST.md) |
| „Wie releasen wir offiziell?“ | [Release Security](../security/GITHUB-RELEASE-SECURITY.md) |

## Checklisten und Beispiele sind keine wirksame Konfiguration

Ein Dokument mit **`Require status checks = ja`** erzwingt keinen einzigen Check. Erst ein tatsächlich aktives Ruleset tut das. Ein `SECURITY.md`-Text ist keine Garantie, dass Private Vulnerability Reporting eingeschaltet ist. Ein grünes CI-Ergebnis kann null Tests bedeuten.

**Jede Empfehlung braucht einen überprüfbaren Endzustand:** Repository, Datum, Check-URL/Commit-SHA, erfasste Konfiguration, Einschränkungen und gegebenenfalls den nächsten Reviewtermin.

## Regelmäßige Pflege

- **Bei GitHub-UI-/Actions-Versionwechseln** die verlinkten offiziellen Dokumentationsseiten prüfen.
- **Monatlich** Abhängigkeiten und Security Alerts sichten.
- **Vor jedem größeren Codex-Auftrag** Tokenrechte und Aufgaben-Scope begrenzen.
- **Vor jedem offiziellen Release** das gesonderte Release-Protokoll nutzen.
- **Nach Sicherheitsvorfällen** die Lessons Learned in das entsprechende Kapitel einarbeiten, ohne vertrauliche Vorfallsdetails öffentlich zu veröffentlichen.

Quelle: [GitHub Docs](https://docs.github.com/en).
