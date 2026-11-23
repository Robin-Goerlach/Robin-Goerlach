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

Quelle: [GitHub Docs](https://docs.github.com/en).
