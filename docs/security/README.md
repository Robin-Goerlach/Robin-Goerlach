# Security – GitHub-Konto, Repositories und Releases

**Stand:** 2026-11-23 · **Status:** Handbuch / empfohlene Prüfverfahren

> Eine Dokumentation ist kein Sicherheitsnachweis. Unbekannte GitHub-Einstellungen bleiben **nicht verifiziert**, bis sie im jeweiligen Konto oder Repository überprüft wurden. Öffentliche Dateien dürfen keine Geheimnisse, Recovery-Codes oder internen Systeme enthalten.

## Einstieg und Kapitel
| Dokument | Was es bietet |
| --- | --- |
| [GITHUB-SECURITY-CHECKLIST.md](GITHUB-SECURITY-CHECKLIST.md) | 17-teilige wiederverwendbare Prüfliste für DEV und RELEASE |
| [GITHUB-ACCOUNT-SECURITY.md](GITHUB-ACCOUNT-SECURITY.md) | 2FA/Passkeys, Recovery, SSH, PATs und Apps |
| [GITHUB-REPOSITORY-SECURITY.md](GITHUB-REPOSITORY-SECURITY.md) | Schutzregeln, Meldungen, Abhängigkeiten und CI-Risiken |
| [GITHUB-RELEASE-SECURITY.md](GITHUB-RELEASE-SECURITY.md) | Release-Gates, Attestations und Incident Response |

## Bedrohungsmodell
Zu schützen sind **GitHub-Konto, Code-Historie, Default-Branches, Release-Artefakte, Signatur-/Authentisierungsschlüssel, CI-Secrets und Backup-/Recovery-Wege**. Mögliche Fehlerquellen: kompromittierte Anmeldeinformationen, zu breit berechtigte Apps, Agenten-Fehlaktionen, absichtliche Pull-Request-Manipulation, anfällige Abhängigkeiten und unkontrollierte Publikations-Jobs.

## Priorität
**P0:** Kontozugriff, Wiederherstellung, ungewollte Pushes, Secrets und vertraulicher Schwachstellenkanal.
**P1:** Dependency- und Code-Scanning, Actions-Härtung, Sicherheitsprozess.
**P2:** Ausbau von Provenance, Fuzzing und Automatisierung.

## DEV ≠ RELEASE
Persönliche DEV-Repos können direkte main-Commits erlauben; mindestens Lösch- und Force-Push-Schutz lohnt sich. Offizielle RELEASE-Repos benötigen stärkere Prüfbarkeit, reproduzierbare Builds und klare Freigaben. Beide Profile brauchen Kontosicherheit, Secret-Management und Backup.

## Kein blindes Häkchensetzen
Die Checkliste enthält **keinen Nachweis**, dass eine Funktion aktiv ist. Eine Warnung oder fehlender API-Zugriff allein beweist keine Schwachstelle. Für jede technische Einstellung GitHub-Oberfläche und, soweit möglich, API-Daten prüfen.

## Vertrauliche Schwachstellenmeldung
GitHub **Private Vulnerability Reporting** ist ein Kanal für externe Sicherheitsmeldungen, sofern im jeweiligen Repo aktiviert. Erst nach der tatsächlichen Aktivierung als funktionsfähig dokumentieren. Keine unbetreute security@-Adresse veröffentlichen.

Offizielle Quellen: [Security Features](https://docs.github.com/en/code-security/getting-started/github-security-features), [GitHub Rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets).
