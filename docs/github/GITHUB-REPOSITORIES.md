# GitHub Repositories – Anlage, Wartung und Archivierung

**Stand:** 2026-11-23 · **Dokumenttyp:** GitHub-Nachschlagewerk · **Sprache:** Deutsch

> Menüs, Tarifumfang und API-Details können sich ändern. Die aktuelle Einstellung eines Repositories ist gesondert zu überprüfen; diese Seite nimmt keine Änderungen an anderen Repositories vor.

## Vor der Erstellung entscheiden
- **Eigentümer:** persönliches Konto oder Organisation; Übertragungen haben Auswirkungen auf Berechtigungen und Verweise.
- **Sichtbarkeit:** öffentlich oder privat. **Öffentlich** heißt: Code und Git-Historie sind weltweit einsehbar.
- **Lizenz:** bewusst auswählen; kein Repository ist allein wegen Sichtbarkeit frei weiterverwendbar.
- **Standardbranch:** **main** nach Teamkonvention.
- **Inhalt:** Quellcode, Tests, README, Lizenz, CONTRIBUTING, SECURITY, .gitignore, .gitattributes.
- **Schutzprofil:** DEV oder RELEASE; Dokumentation ersetzt keine tatsächliche Konfiguration.

## Repository-Einstellungen prüfen
| Bereich | Was konkret prüfen? |
| --- | --- |
| Settings → General | Beschreibung, Branch, Issues, Diskussionsfunktionen |
| Settings → Collaborators / Access | Personen- und App-Rechte |
| Settings → Branches / Rules → Rulesets | Löschung, Force-Push, PRs, Checks |
| Settings → Actions → General | GITHUB_TOKEN, zugelassene Actions, Fork-PR-Policy |
| Settings → Advanced Security | Meldeweg, Abhängigkeiten, Scanning |
| Security / Insights | Alerts, Advisories, Dependency Graph |

Die genaue Navigation variiert je nach UI und Berechtigung. Repository-Administratorrechte sind nicht gleichbedeutend mit der Möglichkeit der verbundenen GitHub-App, jede Einstellung per API auszulesen.

## Lokales Repository korrekt verbinden
~~~bash
git init -b main
git remote add origin git@github.com:OWNER/REPO.git
git remote -v
git fetch origin
git status -sb
~~~
Bei nicht leerem Remote vor einem Push die Historienbeziehung prüfen; nicht zwei unabhängige Histories mittels Force-Push vereinheitlichen.

## Profil-Repository und Standarddateien
Ein Repository mit dem Namen **OWNER/OWNER** zeigt dessen README auf der GitHub-Profilseite. Es ist eine gute **Dokumentationszentrale**, aber keine automatische Konfigurationsverteilung.

Ein separates Repository **OWNER/.github** kann bestimmte **Default Community Health Files** bereitstellen; es ersetzt keine selbst gepflegten Rulesets, Abhängigkeitskonfigurationen oder Private Vulnerability Reporting-Einstellungen. Persönliche und organisatorische .github-Defaults sind getrennt.

## Regelmäßige Pflege
**Monatlich:** Alerts, offene PRs, Releases, Versionsstände. **Vierteljährlich:** Apps, Tokens, Collaborators, Schutzregeln und Backup-Restore. **Bei Archivierung:** Secrets entfernen/widerrufen, abhängige CI-Flows und Release-Bereitstellung prüfen.

## Löschung / Transfer
Vor einer Löschung stets vollständigen Export des Git-Repositories plus benötigte Issues, PRs, Release-Dateien und Sicherheitskonfiguration vornehmen. **git clone --mirror** enthält nicht die vollständigen GitHub-Metadaten.

Quellen: [Repository-Erstellung](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository), [Profile README](https://docs.github.com/en/account-and-profile/how-tos/profile-customization/managing-your-profile-readme), [Default Community Health Files](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file).
