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

## Vertiefung: Release-Sicherheit ist eine Kette von Vertrauensentscheidungen

Ein Produkt ist nur so vertrauenswürdig wie seine Verbindung zwischen **Quellcode**, **Freigaberevision**, **Build**, **Artefakten**, **Verteilung** und **Patchprozess**.

```text
Verifizierter Quellstand
    ↓
CI mit eindeutigem Commit und definierten Rechten
    ↓
Test- und Security-Gates (Protokoll)
    ↓
Build mit definierter Toolchain + Abhängigkeiten
    ↓
Prüfsummen / Signatur / Provenance
    ↓
Release mit Tag, Notes, Downloads
    ↓
Beobachtung, Advisory, Updates
```

### 1. Freigabe durch technische Evidenz

| Signal | Belastbare Aussage | Was es **nicht** aussagt |
| --- | --- | --- |
| CI erfolgreich | Definierte Jobs liefen erfolgreich | Alle Tests/Plattformen abgedeckt |
| Commit-Signatur | Signatur passt zu einem Schlüssel | Code ist korrekt oder reviewt |
| SHA-256 | Datei stimmt mit referenzierter Hashsumme überein | Quelle ist vertrauenswürdig |
| Attestation | Build-Herkunft und Workflow wurden dokumentiert | Build-/Quellcode ohne Schwachstelle |
| SBOM | Aufgelistete Komponenten sind dokumentiert | Inventar ist vollständig |
| Reproducible Build | Gleiche Inputs können identischen Output ergeben | Sicherheitsgarantie des Outputs |

### 2. Artefaktprüfung für Konsumenten

Bei bereitgestellten Artefakten beispielsweise:

```bash
sha256sum DATEI.zip
# Bei eingerichteten GitHub Artifact Attestations:
gh attestation verify DATEI.zip -R OWNER/REPO
```

`gh attestation verify` setzt passende vorhandene Attestations, eine aktuelle GitHub CLI und die Vertrauensanforderungen des Konsumenten voraus. Fehlt die Attestation, darf daraus **kein erfolgreich verifizierter Build** behauptet werden.

### 3. Dependency-/Lizenz-Gate

Vor jeder öffentlichen Binärlieferung:
- direkter/transitiver Paketbestand inventarisiert;
- Herkunft und Versionen fixiert, soweit technisch sinnvoll;
- alle Lizenzpflichten/Attribution für veröffentlichte Formate geprüft;
- Security Advisories mit betroffenen Versionsständen bewertet;
- bei Bedarf SBOM als CycloneDX oder SPDX erzeugt und ihre Abdeckung erklärt;
- Abweichungen transparent dokumentiert.

Weder ein README-Lizenzhinweis noch ein einheitlicher Open-Source-Header ersetzen eine konkrete Lizenzprüfung neuer Dependencies.

### 4. Reproduzierbarkeit in der Praxis

Für C++ bzw. .NET nicht nur Commit SHA merken, sondern auch:
- Compiler/SDK/CMake-Version;
- Paketmanager/Lockfile/Baseline;
- Betriebssystem und Architektur;
- Build-Flags, Optionen und relevante Umgebungsvariablen;
- verwendete Submodule und extern heruntergeladene Tools;
- Testmatrix und genaue CI-Run-ID.

„Reproduzierbar“ streng genommen erst nach erfolgreichem **unabhängigen Rebuild** behaupten, nicht allein weil eine CI-Datei existiert.

### 5. Private Sicherheitsvorfälle und Koordination

Ein vertrauenswürdiger Meldeweg ist nur der erste Teil eines Incident-Prozesses. Für eingehende Sicherheitsmeldungen:

1. Eingang privat bestätigen, Klassifizierung und Eigentümer festlegen.
2. Ausnutzbarkeit, betroffene Versionen und Exposition prüfen.
3. Exploit-Details vertraulich halten und Fix in kontrolliertem Kontext vorbereiten.
4. Regressionstests und gegebenenfalls CodeQL/Sanitizer/Fuzzing ergänzen.
5. Patchversion + Advisory/Release Notes koordinieren.
6. Nutzer über Update-Notwendigkeit und gegebenenfalls Mitigations informieren.
7. Lessons Learned veröffentlichen, aber keine noch ausnutzbaren Geheimnisse.

### 6. Sicherheitsrelevante Rücknahme

Einmal veröffentlichte Binaries und Git-Tags können bereits gespiegelt, gecacht oder heruntergeladen sein. **Löschen** entfernt daher die reale Gefährdung nicht zuverlässig. Neue Version, klarer Advisory-Status und ggf. Revocation bei signierten Artefakten sind notwendige Bestandteile des Reaktionsplans.

### 7. Zuständigkeiten und Freigabe

Auch bei einem Einzelentwickler unterscheidet man **Implementierung**, **technische Kontrolle** und **Veröffentlichungsentscheidung**. Ein zweiter Mensch ist nicht automatisch vorhanden; fehlende unabhängige Prüfung bleibt ein bewusst zu bewertendes Risiko. Für kritische Änderungen kann externe Expertise sinnvoll sein.

## Release-Auditprotokoll – kurze Pflichtfelder

```text
Produkt und Version:
Source-Repo / Commit-SHA / Tag:
Owner und Lizenznachweis:
Erstellungszeitpunkt:
CI-Run und Tests:
Statische Analyse / bekannte Schwachstellen:
SBOM / Paketquellen:
Build-Umgebung / Toolchain:
ZIP/Binary-Hashes:
Signaturen / Attestation (falls vorhanden):
Freigabeentscheidung:
Veröffentlichter Sicherheitsmeldeweg:
Supportete Versionen / Rückfallplan:
```

Keine vertraulichen Tokens oder nicht veröffentlichte Advisory-Details in öffentliche Dateien übernehmen.

Quellen: [GitHub Releases](https://docs.github.com/en/repositories/releasing-projects-on-github), [Artifact Attestations](https://docs.github.com/en/actions/concepts/security/artifact-attestations), [GitHub Security Advisories](https://docs.github.com/en/code-security/security-advisories), [OIDC](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-cloud-providers).
