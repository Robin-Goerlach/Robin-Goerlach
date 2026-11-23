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

Quellen: [Security Quickstart](https://docs.github.com/en/code-security/getting-started/quickstart-for-securing-your-repository), [Rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets), [Private Vulnerability Reporting](https://docs.github.com/en/code-security).
