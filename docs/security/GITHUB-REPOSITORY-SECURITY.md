# GitHub Repository Security – technischer Kontrollplan

**Stand:** 2026-11-23 · **Status:** Handbuch / empfohlene Prüfverfahren

> Eine Dokumentation ist kein Sicherheitsnachweis. Unbekannte GitHub-Einstellungen bleiben **nicht verifiziert**, bis sie im jeweiligen Konto oder Repository überprüft wurden. Öffentliche Dateien dürfen keine Geheimnisse, Recovery-Codes oder internen Systeme enthalten.

## 1. Sichtbarkeit, Zugriffsrechte und Branches (P0)
Repositoryinhaber, Schreibberechtigte und installierte Apps identifizieren. In **Settings → Rules → Rulesets** standardmäßig für **main** Löschung und Force-Push verhindern; Schutz muss auf **Active** stehen. Im persönlichen DEV-Profil dürfen normale direkte Commits erhalten bleiben. Für RELEASE zusätzliche PR-/CI-Gates. Klassische Branch-Protection-Regeln separat prüfen, da sie neben Rulesets wirken können.

Genaues Vorgehen: [GitHub Rulesets](../github/GITHUB-RULESETS.md).

## 2. Vertrauliche Meldungen (P0/P1)
Bei öffentlichen Repositories:
1. Eine passende **SECURITY.md** mit unterstützten Versionen und vertraulichem Meldeweg bereitstellen.
2. Unter **Settings → Advanced Security** den Status von **Private vulnerability reporting** prüfen.
3. Ggf. aktivieren; danach den **Security → Advisories**-Meldeweg prüfen.
4. Offene Meldungen privat bearbeiten; keine unveröffentlichten Schwachstellen in öffentliche Issues kopieren.
5. Keine nicht existierende oder unbetreute **security@...**-Adresse dokumentieren.

Ein **Hinweis im Profil-Repository** schaltet diesen Meldeweg in anderen Repositories **nicht** ein.

## 3. Geheimnisse und Abhängigkeiten (P0/P1)
| Bereich | Konkrete Prüfung |
| --- | --- |
| Secret Scanning | Verfügbarkeit, echte Aktivierung, offene Treffer |
| Push Protection | Konto-/Repo-Schutz, Umgang mit blockierten Pushes |
| Dependency Graph | Welche tatsächlichen Pakete werden erkannt? |
| Dependabot Alerts | Aktivierung und priorisierte Bearbeitung |
| Version Updates | Nur unterstützte Ökosysteme, normale CI für PRs |
| C++/CMake | find_package, Submodule, FetchContent, manuelle Downloads separat inventarisieren |

GitHub erkennt nicht zwingend jede Bibliothek, die ein C++-Projekt tatsächlich verwendet. Ein grünes Dashboard ist kein Beweis für vollständige Dependency-Abdeckung. Für Details siehe [Dependabot](../github/GITHUB-DEPENDABOT.md).

## 4. GitHub Actions (P0/P1)
- **GITHUB_TOKEN** auf Least Privilege; für normale CI meist **contents: read**.
- Fremde Actions nach Prüfung auf **volle Commit-SHAs** pinnen.
- Checkout-Credentials nicht unnötig im Git-Checkout belassen.
- Keine Secrets und keine Schreibrechte für untrusted PR-Ausführung.
- **pull_request_target** plus untrusted PR-Checkout ist besonders gefährlich.
- Self-hosted Runner vor fremdem PR-Code schützen.
- Release-Jobs getrennt absichern.
- Agenten/Codex nicht unbegrenzt Tokens oder Deploymentrechte überlassen.

Verfahren: [GitHub Actions](../github/GITHUB-ACTIONS.md), [Codex Workflow](../github/GITHUB-CODEX-WORKFLOW.md).

## 5. Codeanalyse und Tests (P1)
Für C/C++ Eignung von CodeQL, compiler warnings, clang-tidy/cppcheck, AddressSanitizer, UBSan und (für Parser) Fuzzing prüfen. **Scan konfiguriert** und **Scan erfolgreich mit passender Abdeckung** sind unterschiedliche Zustände. Release nur nach projektspezifischer Testmatrix und nachvollziehbarem Commit.

## 6. Konkreter Prüfdurchlauf
| Schritt | Kontrolle | Soll DEV | Nachweis |
| --- | --- | --- | --- |
| 1 | Zugriffskreis | Nur benötigte Rechte | UNGEPRÜFT |
| 2 | main Ruleset | Lösch-/Force-Push-Schutz aktiv | UNGEPRÜFT |
| 3 | SECURITY.md / vertraulicher Meldeweg | Funktionsfähig | UNGEPRÜFT |
| 4 | Secret Scanning / Push Protection | Geprüft und genutzt | UNGEPRÜFT |
| 5 | Dependency-Abdeckung / Alerts | Bewusst konfiguriert | UNGEPRÜFT |
| 6 | Actions-Berechtigungen / Trigger | Minimal | UNGEPRÜFT |
| 7 | Codeanalyse / Tests | Projektspezifisch | UNGEPRÜFT |
| 8 | Backup und Restore | Praktisch erprobt | UNGEPRÜFT |

Nur den tatsächlich überprüften Status ändern. Bei nicht lesbarem GitHub-API-Endpunkt keine Vermutung über den Sicherheitszustand ableiten.

## Vertiefung: Security Review nach Bedrohungsfläche

### A. Kontrollen für jedes öffentliche Repository

| Kontrollgruppe | Minimalziel | Technischer Nachweis |
| --- | --- | --- |
| Eigentümer und Zugriff | Nur benötigte Maintainer/App-Rechte | GitHub Settings / berechtigte Rollen |
| Default-Branch | Lösch- und Force-Push-Schutz | Ruleset: Ziel + **Active** + Regeln |
| Vertrauliche Meldung | GitHub PVR oder anderer realer Kanal | Einstellungszustand + SECURITY.md |
| Secrets | Nicht in Code, Logs oder Artefakten | Scanner-Abdeckung + manuelle Prüfung |
| Abhängigkeiten | Erkannt und katalogisiert | Graph + Paketliste |
| Actions | Kein untrusted Code mit Release-Secrets | Trigger/Permissions/Runner-Review |
| Backup | Restore nachgewiesen | Testdatum + Wiederherstellungsergebnis |

### B. CodeQL bei C/C++ korrekt verstehen

GitHub unterstützt für C/C++ inzwischen auch **build-mode `none`** bei Default Setup. Das ist schnell zu aktivieren und erfordert keinen Build, kann aber andere Codeerfassung liefern als **`autobuild`** oder **`manual`**. Bei präzisen Build-Flags, Generierung oder speziellen Plattform-Makros ist die **tatsächlich analysierte Codebasis** entscheidend.

**Prüfablauf:**
1. Unter **Security → Code scanning** aktuellen Zustand und letzte erfolgreiche Analyse prüfen.
2. Bei neuem Setup `none` als möglichen Einstieg bewerten, nicht pauschal als unmöglich abtun.
3. Bei CMake-Systemen analysierte Quellen, Header, Präprozessorzweige, Plattformen und generierten Code prüfen.
4. Relevante Alerts triagieren und Ergebnisse versioniert nachvollziehbar halten.
5. Ergänzende `clang-tidy`, Compiler-Warnungen, Sanitizer und Fuzzing verwenden – CodeQL allein reicht nicht.

Quelle: [CodeQL for compiled languages](https://docs.github.com/en/code-security/how-tos/find-and-fix-code-vulnerabilities/manage-your-configuration/codeql-for-compiled-languages).

### C. Bedrohungsflächen bei unterschiedlichen SASD-Produkten

**Terminal-UI-Toolkit (C++):** Terminal-Escape-Sequenzen, Steuerzeichen, fehlerhafte UTF-8-Eingaben, Speicher-/Pufferfehler, Größe/Resize, Eingabeparsing, Ressourcenfreigabe.  
**Firewall-/Routing-Appliance:** Verifizierung von Konfigurationsänderungen, Least Privilege, atomare Änderungen mit Rollback, sichere Verwaltung ohne permanenten Webserver, Logs/IDS-Integrität.  
**API-Server:** Authentisierung/Autorisierung, Inputvalidierung, Rate-Limits, Secret-Verwaltung, sichere Defaults, Audit-Logging.  
**Parser/Crawler:** Bösartige Eingabedaten, Ressourcenerschöpfung, SSRF, Deserialisierung, Dateipfade, externe Inhalte.

Diese Fragen definieren **Testideen**, keine Behauptung, dass ein Projekt solche Schwachstellen besitzt.

### D. SECURITY.md und Private Vulnerability Reporting

**SECURITY.md** dokumentiert, wie Nutzer Sicherheitsprobleme vertraulich melden und welche Versionen betreut werden. **Private Vulnerability Reporting** ist eine eigenständige GitHub-Einstellung je öffentliches Repository. Ein README-Link ersetzt den aktivierten Meldeweg nicht.

Ein präziser Text könnte lauten:

> Bitte melde vermutete Sicherheitslücken über die Funktion „Report a vulnerability“ im Bereich **Security → Advisories**, sofern sie für dieses Repository aktiviert ist. Veröffentliche Details nicht als öffentliches Issue.

**Achtung:** Diesen Text **erst verwenden**, wenn die Funktion im konkreten Repository nachweislich aktiv ist. Alternativ muss ein tatsächlich überwachter vertraulicher Kanal benannt werden. Keine erfundenen `security@`-Adressen.

### E. Pull-Request-Angriffe und Branchschutz

Ein PR von einem Fork ist eine fremde Codeänderung, nicht nur ein harmloser Textvorschlag. Gefährliche Stellen:
- Buildskript zieht ausführbaren Code aus externen Quellen.
- Actions job bekommt schreibendes GITHUB_TOKEN.
- PR-Head wird mit `pull_request_target` oder einem privilegierten Folge-Workflow ausgeführt.
- Kompromittierter Cache oder Artefakt wird in einem Release-Job erneut verwendet.
- Schadcode wird durch ein gültig signiertes Commit eingebracht.

Mindestmaßnahmen: `permissions: contents: read` für Tests, Full-SHA-Pinning von Fremd-Actions, untrusted Eingaben nicht in Shell-Code interpolieren und Release-Gates separat konfigurieren.

### F. Was das Security-Dashboard nicht beweist

- „0 Alerts“ kann bedeuten, dass der Scanner **nichts** abdeckt.
- „PVR configured“ sagt nichts über zeitnahe Reaktion der Maintainer aus.
- „Tests passed“ kann nur eine kleine Plattformmatrix oder null Tests bedeuten.
- „Branch Protection“ kann über Bypass-Permissions umgangen werden.
- „Verified commit“ bestätigt eine Signaturkonstellation, nicht eine unabhängige Sicherheitsprüfung.

## Prüfprotokoll für ein Repository

```text
Repository / Sichtbarkeit:
Prüfdatum / Prüfer:
Schutzprofil DEV|RELEASE:
Default-Branch / aktive Regeln:
Berechtigungen / App-Review:
Security Policy / PVR:
Secret Scanning / Push Protection:
Dependency Graph / Coverage-Lücken:
Actions-Trigger / effektive Token-Rechte:
Code Scanning / getestete Plattformen:
Backups / Restoretest:
Offene Befunde (P0/P1/P2):
Risikoakzeptanz: wer, warum, bis wann?
Nächste technische Aktion:
```

**Keine** Tokenwerte, Recovery-Codes oder internen Netzwerkdetails in öffentliche Nachweise aufnehmen.

Quellen: [Security Quickstart](https://docs.github.com/en/code-security/getting-started/quickstart-for-securing-your-repository), [Rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets), [Private Vulnerability Reporting](https://docs.github.com/en/code-security).
