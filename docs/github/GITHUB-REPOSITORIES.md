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

## Vertiefung: Repository-Lebenszyklus als Architekturentscheidung

### A. Ein neues Projekt anlegen

Vor dem ersten Commit sollten mindestens diese Entscheidungen feststehen:

| Entscheidung | Beispiel | Warum wichtig? |
| --- | --- | --- |
| Owner | Person oder Organisation | Rechtliche und technische Verantwortung |
| Visibility | public/private | Vertraulichkeitsgrenze |
| License | konkret definierte Lizenz | Nutzungsrechte und Abhängigkeiten |
| Default Branch | main | CI- und Ruleset-Ziel |
| Schutzprofil | DEV oder RELEASE | Direkt-Push versus erzwungenes Gate |
| Meldeweg | GitHub Private Vulnerability Reporting | Vertrauliche Schwachstellenkoordination |
| Build-Anleitung | README + CI | Reproduzierbarkeit |
| Release-Verantwortung | namentlich/rollenbasiert | Notfallentscheidungen |

Eine **öffentlich sichtbare** Repository-Historie enthält möglicherweise auch aus Git entfernte Altdateien; ein späterer Wechsel auf `private` macht veröffentlichte Kopien nicht rückwirkend privat.

### B. Vom persönlichen Beta-Repository zur Organisation

Bei einer späteren SASDGMBH-Veröffentlichung gibt es zwei verschiedene Modelle:

**Repository übertragen:** Die bestehende Historie und das Repository wechseln zu einem anderen Owner; GitHub-Einstellungen, Berechtigungen, URLs, Rulesets, Paketnamen, Releases, Secrets und externe Integrationen müssen im Zielkontext überprüft werden.

**Neues Organisationsrepository anlegen:** Der offizielle Veröffentlichungsstand wird bewusst als eigenes Produkt fortgeführt. Hier muss dokumentiert werden, wie Commit-Historie, Attribution, Lizenzen, Versionsnummern und Release-Artefakte zusammenhängen. Keine stillschweigende rechtliche Gleichsetzung mit dem persönlichen Repository.

**Nicht** eine technische Umbenennung oder Übertragung als Beweis für die Übertragung von Urheber- oder Verwertungsrechten betrachten. Diesen Punkt separat rechtlich klären, wenn relevant.

### C. Community-Dateien und Standard-Repository

Für eine allgemeine GitHub-Präsenz:
- `README.md`: Einstieg, Zweck, Build, Support.
- `LICENSE`: verbindlicher Lizenztext, wenn eine Lizenz gewollt ist.
- `CONTRIBUTING.md`: Entwicklungs-/PR-/Issue-Konventionen.
- `CODE_OF_CONDUCT.md`: Regeln bei offener Zusammenarbeit.
- `SECURITY.md`: unterstützte Versionen und **echter vertraulicher Kontaktweg**.
- `CODEOWNERS`: automatische Review-Anfragen für bestimmte Pfade (ersetzt kein Branch-Ruleset).

**Profil-Repository `OWNER/OWNER`:** eigene öffentliche Profilseite.  
**Defaults-Repository `OWNER/.github`:** mögliche zentrale Community-Health-Defaults.  
**Normales Projekt-Repository:** verwaltet seine tatsächlich wirksamen Settings **selbst** (oder via geeigneter Organisationsverwaltung).

### D. Forks und Drittbeiträge

Ein Fork ist ein eigenständiges Repository mit möglicherweise anderem Eigentümer. PRs aus Forks sind **nicht vertrauenswürdiger Code**. CI auf Fork-PRs braucht minimal berechtigte Tokens und darf keine Produktionsecrets zur Verfügung stellen. Bei fremden Beiträgen Lizenz/Attribution und Herkunft kontrollieren.

### E. Archiv, Löschung und Wiederherstellung

**Archivieren** reduziert weitere Bearbeitung, ist aber keine Backup-Strategie. Vor einer Archivierung:
1. offene Security Advisories, Support-/Release-Zusagen und Deprecation-Hinweise prüfen;
2. Token-, Deploy-Key- und Actions-Abhängigkeiten inventarisieren;
3. Quellcode, Tags, Releases und relevante Metadaten unabhängig sichern;
4. öffentliche README um Archivstatus und ggf. Nachfolgeprojekt ergänzen;
5. Restore-Test durchführen.

### F. Sicherheitsüberprüfung als Routine

**Bei Erstellung:** Owner, Visibility, Lizenz, Zugriff, Ruleset, SECURITY.md, CI-Grundstruktur.  
**Monatlich:** Alerts, Updates, offene PRs.  
**Vierteljährlich:** Apps, Rechte, Tokens, Backups.  
**Vor Release:** eindeutig geprüfter Commit, CI, Abhängigkeiten, Lizenz, Security-Meldeweg und Artefakt-Herkunft.

## Konkrete Prüfbefehle (nur lesend)

```bash
git remote -v
git branch -vv
git ls-remote --heads origin
git ls-remote --tags origin
git log --oneline --decorate --graph -12
```

Diese Befehle zeigen **Git-Referenzen**, nicht alle GitHub-Plattformdaten. Rulesets, Advisories, Releases und App-Rechte separat in der Weboberfläche oder mit passenden autorisierten Schnittstellen prüfen.

## Typische Fehlannahmen

- **„Repository öffentlich = Open Source“**: nein, ohne passende Lizenz gelten nicht automatisch die gewünschten Nutzungsrechte.
- **„Profil-README verteilt Settings“**: nein.
- **„Clone/Mirror ist vollständiges Backup“**: nein, GitHub-Metadaten und Secrets fehlen.
- **„Adminrechte im Web = jede API-App kann alles lesen“**: nein; Apps haben eigene Berechtigungen.
- **„Ein leerer Dependabot-Report bedeutet keine Dependencies“**: nein, erst Erkennungsumfang verifizieren.

Quellen: [Repository-Erstellung](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository), [Profile README](https://docs.github.com/en/account-and-profile/how-tos/profile-customization/managing-your-profile-readme), [Default Community Health Files](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file).
