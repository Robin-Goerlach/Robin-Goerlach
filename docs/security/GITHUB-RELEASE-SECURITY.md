# GitHub Release Security – geprüfte Veröffentlichungen und Vorfallreaktion

**Stand:** 2026-11-23 · **Status:** Handbuch / empfohlene Prüfverfahren

> Eine Dokumentation ist kein Sicherheitsnachweis. Unbekannte GitHub-Einstellungen bleiben **nicht verifiziert**, bis sie im jeweiligen Konto oder Repository überprüft wurden. Öffentliche Dateien dürfen keine Geheimnisse, Recovery-Codes oder internen Systeme enthalten.

## 1. Rollen und Geltungsbereich
**DEV:** schnelles Entwickeln, die Qualität einer Veröffentlichung aber niemals allein aus einem erfolgreichen Commit ableiten. **RELEASE:** kontrollierte Veröffentlichung über das dafür vorgesehene SASDGMBH-Repository; Eigentümer, Lizenz, Attribution, Signatur- und Release-Rechte ausdrücklich prüfen.

## 2. Release-Gates (P0)
| Gate | Nachweis / Entscheidung |
| --- | --- |
| Freizugebender Commit | Eindeutige vollständige Commit SHA |
| Version und Tag | Unveränderlicher, konsistenter Versionstag |
| Teststatus | Bestanden für genau diesen Commit; Ausnahmen dokumentiert |
| Security Alerts | Kritische Befunde bewertet/behoben |
| Lizenz/SBOM | Drittkomponenten und Nutzungsrechte überprüft |
| Build | Toolchain, Umgebung und Parameter dokumentiert |
| Artefakte | Checksums, Signatur/Attestation nach Bedarf |
| Freigabe | Verantwortlicher und Datum dokumentiert |
| Meldeweg | Funktionierende SECURITY.md / Private Vulnerability Reporting |
| Recovery | Patch- oder Widerrufsverfahren vorhanden |

Ein Release darf nur „Security Ready“ heißen, wenn die definierten Gates tatsächlich geprüft wurden. Automatische Signatur/Provenance ist **kein Ersatz** für Codeanalyse und Dependency-Review.

## 3. Release-Pipeline absichern
- Testjobs und Publish-Jobs getrennt, Veröffentlichung nur in vertrauenswürdigem Kontext.
- Schreibendes **GITHUB_TOKEN** nur am konkreten Publish-Job.
- Für externe Cloud-Dienste möglichst **OIDC/kurzlebige Berechtigungen** statt PATs mit langer Lebensdauer.
- Geschützte **Environments** und Approval nutzen, sofern für das Repository verfügbar und sinnvoll.
- Untrusted Fork-PRs können keine echten Deployment-Schlüssel erreichen.
- Drittanbieter-Actions und Builder-Images prüfen/pinnen.
- Vorabversionen klar als Pre-Release markieren.
- Veröffentlichte Tags und Artefakte nicht stillschweigend austauschen.

## 4. Provenance und Konsumentenvertrauen
**SHA-256** bestätigt Übereinstimmung mit einem bekannten Hash, aber nicht automatisch dessen Vertrauenswürdigkeit. Eine **Signatur** belegt die Nutzung des zugehörigen Schlüssels. **Artifact Attestations** können Build-Herkunft/Workflow/Commit dokumentieren. Alle drei Verfahren benötigen eine glaubwürdige, überprüfbare Veröffentlichungskette.

## 5. Sicherheitsvorfall während oder nach Release
1. Veröffentlichung stoppen und relevante Zeitpunkte/Commit-/Workflow-IDs sichern.
2. Geheimnisse oder Zugangsschlüssel bei Verdacht **sofort widerrufen/rotieren**.
3. Angriffsweg bestimmen: Code, CI, Runner, Dependency, Token oder Artefakt-Upload.
4. Betroffene Releases und Nutzer identifizieren; ggf. vertrauliche Koordination über Advisories.
5. Fix und Tests nachvollziehbar auf neuem Commit erstellen.
6. Patch-Version und Advisory/Release Notes veröffentlichen; betroffene Nutzer informieren.
7. Regeln, Berechtigungen und Recovery-Verfahren verbessern.

**Keine privaten Exploit-Details oder Tokens im öffentlichen Incident-Bericht veröffentlichen.**

## 6. Beispiel einer Release-Freigabevorlage
~~~text
Repository:
Version / Tag / Commit SHA:
Release-Owner:
CI-Run-Links:
Codeanalyse / offene Befunde:
Dependency- und Lizenzprüfung:
Build-Umgebung:
Artefakt-Checksummen/Signatur:
Dokumentation / bekannte Einschränkungen:
Freigabeentscheidung / Datum:
Patch-/Rollback-Plan:
~~~

## 7. Backup beachten
Ein Git-Mirror sichert **Commits, Branches und Tags**, nicht automatisch GitHub-Release-Dateien, PR-Diskussionen, Rulesets oder Secrets. Diese separat sichern, dokumentieren bzw. über die Plattform reproduzieren können.

Quellen: [GitHub Releases](https://docs.github.com/en/repositories/releasing-projects-on-github), [Artifact Attestations](https://docs.github.com/en/actions/concepts/security/artifact-attestations), [GitHub Security Advisories](https://docs.github.com/en/code-security/security-advisories), [OIDC](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-cloud-providers).
