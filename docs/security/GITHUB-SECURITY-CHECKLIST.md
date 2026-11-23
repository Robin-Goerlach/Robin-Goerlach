# GitHub Security Checklist – Persönliche Entwicklung und SASDGMBH

**Version:** 1.0 (Entwurf)  
**Stand:** 2026-11-23  
**Vorgeschlagener Ablageort:** `Robin-Goerlach/Robin-Goerlach/docs/security/GITHUB-SECURITY-CHECKLIST.md`  
**Geltungsbereich:** Persönliches GitHub-Konto, einzelne SASD-Repositories, GitHub Actions und offizielle Veröffentlichungen über SASDGMBH.

> **Ziel:** Einen möglichst sicheren, aber für einen Einzelentwickler praktikablen Workflow erreichen. Diese Liste ist eine **Prüfvorlage**, kein Auditbericht. Unmarkierte Punkte sind **nicht automatisch Mängel**. Funktionen können vom Repository-Typ, GitHub-Tarif und der jeweiligen Oberfläche abhängen.

## 0. Vorgehen, Status und Nachweise

**Status je Kontrollpunkt:** `[ ]` ungeprüft · `[x]` nachweislich erfüllt · `N/A` begründet nicht anwendbar · `OFFEN` zu bearbeiten · `RISIKO AKZEPTIERT` bewusste Abweichung mit Begründung.

Bei jeder Überprüfung festhalten:

| Feld | Eintrag |
| --- | --- |
| Repository / Geltungsbereich | |
| Eigentümer (persönlich / Organisation) | |
| Sichtbarkeit (öffentlich / privat) | |
| Schutzprofil (DEV / RELEASE) | |
| Prüfer / Prüfdatum | |
| Letzter überprüfter Commit (falls relevant) | |
| Ergebnisse / offene Punkte | |
| Belege (Link auf Settings, Workflow, PR, Commit, Screenshot) | |
| Abweichungen und Begründung | |
| Nächste Prüfung | |

**Regel:** Konfiguration nur als „erfüllt“ markieren, wenn sie tatsächlich überprüft wurde. Eine angelegte, aber deaktivierte Regel ist **kein aktiver Schutz**. Nicht erreichbare Einstellungen als „nicht verifiziert“ dokumentieren; nicht aus fehlenden API-Daten auf einen unsicheren Zustand schließen.

## 1. Bedrohungsmodell und Sicherheitsziele

- [ ] Kritische Werte identifiziert: GitHub-Konto, Quellcode, `main`, Releases, Pakete, Signaturschlüssel, SSH-Schlüssel, Tokens, CI-Secrets.
- [ ] Relevante Bedrohungen berücksichtigt: Kontodiebstahl, kompromittierte App, fehlgeleiteter Codex-Lauf, versehentlicher Force-Push, schädliche PRs, kompromittierte Abhängigkeiten, manipulierte GitHub Actions.
- [ ] Risiko nach Schutzprofil unterschieden: **DEV** (schnell und kontrolliert) und **RELEASE** (nachweisbar geprüft).
- [ ] Keine Zugangsdaten, Recovery-Codes, internen Netzpläne oder sensiblen Infrastrukturdetails in öffentlichen Checklisten oder Screenshots.

## 2. GitHub-Kontosicherheit – **P0**

Pfadbeispiel: **Profilbild → Settings → Password and authentication**; weitere Zugriffe unter **Settings → SSH and GPG keys / Developer settings / Applications / Sessions** (Bezeichnungen können variieren).

- [ ] Zwei-Faktor-Authentifizierung (2FA) aktiv und aktuelle Methode geprüft.
- [ ] Nach Möglichkeit mindestens ein Passkey oder FIDO2-Sicherheitsschlüssel eingerichtet.
- [ ] Zweiter unabhängiger Anmelde- oder Wiederherstellungsweg vorhanden.
- [ ] GitHub-Recovery-Codes aktuell und **offline bzw. sicher außerhalb von GitHub** aufbewahrt; Wiederauffindbarkeit geprüft.
- [ ] Haupt-E-Mail-Konto separat stark geschützt (idealerweise ebenfalls phishingresistente MFA).
- [ ] Unbekannte bzw. alte Sitzungen und Geräte geprüft und bei Bedarf abgemeldet.
- [ ] SSH-Authentifizierungsschlüssel inventarisiert; ungenutzte Schlüssel entfernt.
- [ ] SSH-Schlüssel mit geeigneter Passphrase geschützt, sofern mit dem Einsatz vereinbar.
- [ ] Personal Access Tokens (PATs) geprüft: nur notwendige Berechtigungen, möglichst kurze Laufzeit, keine unkontrollierten langlebigen Tokens.
- [ ] OAuth-Apps und GitHub-App-Installationen geprüft; unnötige Berechtigungen und Altinstallationen entfernt.
- [ ] Deploy Keys geprüft: pro Repository nur wenn nötig; Schreibrechte möglichst vermeiden.
- [ ] Sicherer Ausweichweg bei Verlust des primären Rechners oder 2FA-Geräts dokumentiert – **ohne geheime Daten öffentlich abzulegen**.

**Hinweis:** 2FA schützt einen gestohlenen PAT oder privaten SSH-Schlüssel nicht nachträglich. Solche Zugänge erfordern eine eigene Berechtigungs- und Widerrufsprüfung.

## 3. Allgemeine Repository-Einstellungen – **P0/P1**

Pfadbeispiel: **Repository → Settings → General** sowie **Collaborators / Access**.

- [ ] Eigentümer, Sichtbarkeit und Zweck des Repositories stimmen mit dem Projektstatus überein.
- [ ] `main` (oder der gewählte Standardbranch) stimmt mit der Dokumentation überein.
- [ ] Personen mit Write-, Maintain- und Admin-Zugriff sind bekannt und erforderlich.
- [ ] Installierte GitHub Apps haben Zugriff nur auf benötigte Repositories, soweit konfigurierbar.
- [ ] Fork-/Issue-/PR-/Discussion-Einstellungen sind bewusst gewählt; keine Sicherheitsannahme allein aus deaktivierten Issues.
- [ ] Öffentliche README, `LICENSE`, `CONTRIBUTING.md` und ggf. `CODE_OF_CONDUCT.md` entsprechen dem Projektstatus.
- [ ] Keine vertraulichen Daten in Git-Historie, Releases, Wiki, Issues oder Actions-Logs bekannt.
- [ ] Archivierte/verwaiste Projekte werden bei Bedarf als read-only archiviert und vorhandene Secrets widerrufen.

## 4. Branch- und Tag-Schutz – **P0**

Pfad: **Repository → Settings → Rules → Rulesets**. Für öffentliche Repositories sind Rulesets auch mit GitHub Free verfügbar. Regeln sind erst wirksam, wenn die Durchsetzung auf **Active** steht.

### DEV-Profil: persönliches Beta-Repository

- [ ] Ruleset auf den tatsächlichen Default-Branch (`main`) ausgerichtet.
- [ ] **Restrict deletions** aktiviert.
- [ ] **Block force pushes** aktiviert.
- [ ] **Enforcement status: Active** kontrolliert.
- [ ] Bypass-Ausnahmen geprüft und begründet; keine unnötigen Ausnahmen.
- [ ] Direkte `main`-Commits sind – wenn bewusst gewünscht – weiterhin möglich.
- [ ] **Require pull request** und **Require status checks** sind nicht versehentlich erzwungen, solange direkte Commits ausdrücklich gewollt sind.
- [ ] Für größere Arbeiten mit Codex möglichst eigener Arbeitsbranch + optionaler Pull Request.
- [ ] Vor problematischen Änderungen existiert ein Rückweg (neuer Commit zum Revert, Tag, Git-Backup).

### RELEASE-Profil: SASDGMBH

- [ ] Pull Requests vor Änderungen an `main` verpflichtend oder bewusst anders geregelt.
- [ ] Relevante CI-Checks vor Merge verpflichtend; Namen und Funktionsfähigkeit der Checks nachgewiesen.
- [ ] Anforderungen an Reviews sind für die tatsächlich verfügbaren Mitwirkenden realistisch; keine unerfüllbare zweite-Person-Regel bei nur einem Maintainer.
- [ ] Bypass-Rechte für Administratoren, Bots und Apps bewusst auf ein Minimum begrenzt.
- [ ] Release-Tags gegen unerwünschtes Überschreiben oder Löschen geschützt, soweit technisch und organisatorisch geeignet.
- [ ] Signierte Commits/Tags als zusätzlicher Schutz bewertet; nicht blind verpflichtend gemacht.

**Abgrenzung:** Ein Ruleset verhindert bestimmte Änderungen am geschützten Branch; es ist weder ein Ersatz für Git-Backups noch ein Schutz gegen jede kompromittierte Admin-Identität.

## 5. Sicherheitsmeldungen und Schwachstellenmanagement – **P0/P1**

Pfad: **Repository → Settings → Advanced Security** sowie **Repository → Security → Advisories**.

- [ ] Eine aussagekräftige `SECURITY.md` vorhanden (im Repository oder über die von GitHub unterstützten Standarddateien).
- [ ] `SECURITY.md` benennt unterstützte Versionen oder beschreibt, wie sie bestimmt werden.
- [ ] Ein tatsächlich funktionierender **vertraulicher Meldeweg** ist dokumentiert.
- [ ] Bei öffentlichen Repositories **Private vulnerability reporting** geprüft und nach Möglichkeit aktiviert.
- [ ] Nach Aktivierung ist der Meldeweg über **Security → Advisories** plausibel nachvollziehbar.
- [ ] Nicht vorhandene `security@...`-E-Mail-Adressen werden **nicht** als Kontakt angegeben.
- [ ] Zuständigkeit und Reaktionsablauf dokumentiert: Eingang prüfen → Schweregrad einschätzen → Fix privat vorbereiten → Tests → Veröffentlichung/Advisory.
- [ ] Für sicherheitsrelevante Releases: GitHub Security Advisory, CVE und betroffene Versionen bei Bedarf geprüft.
- [ ] Fehlerberichte enthalten keine Zugangsdaten oder aktiven Exploit-Details in öffentlichen Issues.

**Wichtig:** Eine Dokumentation im Profil-Repository aktiviert Private Vulnerability Reporting **nicht** in anderen Repositories. Die Einstellung ist für jedes betroffene Repository zu kontrollieren.

## 6. Secret Scanning und Schutz vertraulicher Daten – **P0/P1**

Pfad: **Repository → Settings → Advanced Security** bzw. **Security → Secret scanning**.

- [ ] Secret-Scanning-Verfügbarkeit und aktivierte Schutzfunktionen geprüft.
- [ ] Push Protection für die verwendeten Konten/Repositories geprüft; Schutz bei einem blockierten Push nicht leichtfertig übergangen.
- [ ] GitHub Secrets und Variables auf tatsächlich benötigte Einträge und Sichtbarkeit geprüft.
- [ ] Keine echten Zugangsdaten in Tests, Fixtures, `.env`-Dateien, Screenshots, Beispielkonfigurationen oder Actions-Logs.
- [ ] `.gitignore` und, wo sinnvoll, lokale Pre-Commit-Scanner oder Repository-Scans vorhanden.
- [ ] Bei geleaktem Secret **sofort widerrufen/rotieren**; das Entfernen aus dem aktuellen Commit reicht nicht, weil Git-Historie und Klone existieren können.
- [ ] Geheimnisrotation und Eskalationsweg für den Vorfall dokumentiert.

**Grenze:** Secret Scanning erkennt bestimmte Muster, nicht alle denkbaren Geheimnisse. Es ersetzt keine sichere Geheimnisverwaltung.

## 7. Abhängigkeiten und Lieferkette – **P1**

Pfad: **Repository → Settings → Advanced Security**, **Insights → Dependency graph**, **Security → Dependabot alerts**; Version-Updates über `.github/dependabot.yml`.

- [ ] **Dependency Graph** für unterstützte Abhängigkeiten aktiviert und seine tatsächliche Abdeckung geprüft.
- [ ] **Dependabot Alerts** aktiviert, sofern verfügbar.
- [ ] **Dependabot Security Updates** als Option bewertet und falls sinnvoll aktiviert.
- [ ] **Dependabot Version Updates** für erkannte Ökosysteme eingerichtet (z. B. `github-actions`, `nuget`, `maven`, `npm`).
- [ ] Pull Requests von Dependabot werden durch die normale CI geprüft.
- [ ] Kritische und hohe Alerts erhalten eine klare Bearbeitungspriorität.
- [ ] Externe Bibliotheken, Git-Submodule, FetchContent, vcpkg/Conan und manuelle Downloads sind auch dann inventarisiert, wenn GitHub sie **nicht** erkennt.
- [ ] Versionen und Quellen nach Möglichkeit reproduzierbar fixiert; Updates werden bewusst durchgeführt.
- [ ] Neue oder geänderte Abhängigkeiten inklusive Lizenz, Maintainer, Integrität und transitive Abhängigkeiten geprüft.
- [ ] Software Bill of Materials (SBOM) für veröffentlichte Bibliotheken als späterer Ausbaupunkt bewertet.

**C++/CMake-Sonderfall:** Aus aktivem Dependency Graph folgt **nicht**, dass GitHub automatisch jede `find_package`-, `FetchContent`- oder selbst eingebundene C++-Bibliothek versteht. Abdeckung gegen die offiziell unterstützten Manifest- und Submission-Verfahren prüfen.

## 8. GitHub Actions und Automatisierung – **P0/P1**

Pfad: **Repository → Settings → Actions → General**, außerdem `.github/workflows/*.yml`.

- [ ] `GITHUB_TOKEN` erhält möglichst nur Leserechte (`permissions: contents: read`), höhere Rechte ausschließlich im erforderlichen Job.
- [ ] Bei `actions/checkout` ist `persist-credentials: false` gesetzt, sofern keine spätere Git-Authentifizierung benötigt wird.
- [ ] Externe Actions möglichst auf vollständige Commit-SHA statt nur einen beweglichen Tag festgelegt; Update-Verfahren dokumentiert.
- [ ] Zugelassene Actions/Reusable Workflows und die Actions-Policy bewusst gewählt.
- [ ] Workflows aus Fork-Pull-Requests erhalten weder unnötige Schreibrechte noch produktive Secrets.
- [ ] `pull_request_target`, `workflow_run` und die Ausführung nicht vertrauenswürdigen PR-Codes besonders geprüft.
- [ ] Untrusted Eingaben aus PR-Titeln, Branch-Namen, Commit-Nachrichten usw. nicht ungeschützt in Shell-Befehle eingebaut.
- [ ] Actions von Drittanbietern auf Herkunft, Wartung, Berechtigungen und Vertrauenswürdigkeit geprüft.
- [ ] Selbstgehostete Runner nicht unkontrolliert für fremde PRs nutzbar; Netzwerk- und Secret-Zugriff berücksichtigt.
- [ ] Caches und Build-Artefakte als mögliche Supply-Chain-Angriffsfläche betrachtet.
- [ ] Für Cloud-Zugriffe OIDC bzw. kurzlebige Berechtigungen anstelle langlebiger Cloud-Secrets bewertet.
- [ ] Release-/Publish-Jobs erhalten eigene, eng begrenzte Berechtigungen und laufen nur in geeigneten vertrauenswürdigen Kontexten.
- [ ] Logs werden auf versehentliche Geheimnisveröffentlichung geprüft; keine Tokens im Debug-Modus ausgeben.

**Codex/Agenten-Zusatz:**

- [ ] Dokumentiert, welche Verzeichnisse, Befehle, Branches und Releases Codex autonom verändern darf.
- [ ] Vor Merge/Release wird die **Differenz tatsächlich geprüft**; ein grüner Testlauf allein beweist keine sichere Änderung.
- [ ] Codex hat keine unnötigen Produktions- oder Release-Secrets in seiner Ausführungsumgebung.
- [ ] Kritische Operationen (Secrets, Rulesets, Releases, Branch-Löschungen) sind von alltäglicher Codegenerierung getrennt.

## 9. Statische Analyse, Tests und Build-Sicherheit – **P1/P2**

Pfad: **Repository → Security → Code scanning**.

- [ ] CodeQL oder vergleichbare Codeanalyse für unterstützte Programmiersprachen bewertet.
- [ ] Bei C/C++ der tatsächlich verwendete CodeQL-Buildmodus (`none`, `autobuild` oder manuell) und seine Abdeckung geprüft.
- [ ] Fehlgeschlagene oder überhaupt nicht laufende Analysen nicht als „Code Scanning aktiv und wirksam“ gewertet.
- [ ] Compiler-Warnungen und statische Analyse (z. B. clang-tidy, cppcheck) passend zum Projekt eingerichtet.
- [ ] Tests decken gefährliche Grenzfälle ab: Parser, Strings, Dateiformate, Eingaben, Rechte, Fehlerpfade und Recovery.
- [ ] C/C++-Projekte: AddressSanitizer/UndefinedBehaviorSanitizer in geeigneter Testumgebung bewertet.
- [ ] Fuzzing für sicherheitsrelevante Parser und Netzwerkfunktionen erwogen.
- [ ] Reproduzierbarkeit der Builds und verwendete Toolchain-Versionen dokumentiert.

## 10. Offizielle Releases und SASDGMBH – **P0 vor öffentlicher Veröffentlichung**

- [ ] Repository-Eigentümer, Rechtsträger und Lizenz für **das konkrete Release** geprüft.
- [ ] Nicht angenommen, dass die Lizenz des persönlichen Beta-Repositories automatisch für die Organisation gilt; Rechte und Lizenztexte explizit dokumentiert.
- [ ] Release-Commit entspricht einem überprüften, erfolgreichen CI-Stand.
- [ ] Release-Tag ist eindeutig; Herkunft des veröffentlichten Artefakts auf Commit/Workflow zurückführbar.
- [ ] Veröffentlichte ZIPs, Binaries und Pakete werden aus einer kontrollierten Build-Pipeline erzeugt.
- [ ] Release-Berechtigungen (Token, Environment, Approval) getrennt von allgemeinen Build-Berechtigungen.
- [ ] Release Notes, bekannte Einschränkungen, Security Policy und unterstützte Versionen vorhanden.
- [ ] Prüfsummen, Signaturen und/oder Build-Provenance/Attestations je nach Produkt und Aufwand bewertet.
- [ ] Veröffentlichungen sind nachvollziehbar versioniert; ein Zurückziehen oder Ersetzen ist als Prozess dokumentiert.
- [ ] Vor Veröffentlichung ein kurzes Security-Readiness-Review durchgeführt und dokumentiert.

## 11. Backup, Wiederherstellung und Vorfallsreaktion – **P0/P1**

- [ ] Wichtige Repositories einschließlich **aller relevanten Branches und Tags** regelmäßig unabhängig von GitHub gesichert.
- [ ] Ein vollständiger Wiederherstellungstest (nicht nur Backup-Dateien ansehen) mindestens einmal durchgeführt.
- [ ] Kritische GitHub-Metadaten identifiziert: Issues, PR-Diskussionen, Releases, Actions-Konfiguration, Wiki, Projektboards und **nicht im Git enthaltene Einstellungen**.
- [ ] Für nicht durch Git gesicherte Artefakte ein geeigneter Export-/Dokumentationsweg festgelegt.
- [ ] Bei verdächtigem Commit: Änderungen stoppen → Zugriff prüfen → schädliche Änderung identifizieren → gegebenenfalls revertieren → CI und Release-Artefakte überprüfen.
- [ ] Bei kompromittiertem Token/SSH-Schlüssel: sofort widerrufen/rotieren → Zugriffe und Logs prüfen → weitere Berechtigungen bewerten.
- [ ] Bei veröffentlichter Schwachstelle: privat koordinieren und betroffene Nutzer über einen geeigneten Advisory-/Release-Kanal informieren.
- [ ] Wiederanlauf nach kompromittiertem Entwicklungsrechner oder GitHub-Konto beschrieben.

## 12. Konkreter Standard nach Repository-Klasse

| Maßnahme | Persönliches DEV-Repo | SASDGMBH RELEASE-Repo |
| --- | --- | --- |
| 2FA und Recovery | Pflicht auf Kontoebene | Pflicht bei allen berechtigten Konten |
| `main` löschen/Force-Push | Blockiert | Blockiert |
| Direkt in `main` committen | Erlaubt, wenn bewusst | Möglichst nicht |
| PR-Pflicht | Optional | Bevorzugt verpflichtend |
| CI | Durchführen und Fehler prüfen | Vor Merge/Release erforderlich |
| `SECURITY.md` | Ja | Ja |
| Private Vulnerability Reporting | Bei öffentlichem Repo | Bei öffentlichem Repo |
| Dependency Graph / Alerts | Nach Möglichkeit ja | Ja, Abdeckung geprüft |
| Secret Scanning / Push Protection | Prüfen und nutzen | Prüfen und nutzen |
| CodeQL / statische Analyse | Je nach Sprache/Projekt | Eignung vor Release prüfen |
| Unnötige Action-Schreibrechte | Vermeiden | Strikt vermeiden |
| Freigabe von Release-Artefakten | Einfach, aber bewusst | Dokumentiert |
| Backup | Ja | Ja, inklusive Release-Metadaten |

## 13. Erster Prüfdurchlauf: SASD-UI-Toolkit

**Stand dieser Vorlage:** Der nachstehende Zustand ist **nicht als erledigt attestiert**; nur die bereits bekannten Sachverhalte sind als Ausgangspunkt vermerkt. Die GitHub-Anbindung lieferte am 23.11.2026 für `Robin-Goerlach/SASD-UI-Toolkit` **keine Repository-Rulesets** (`[]`). Klassische Branch-Protection oder andere Einstellungen können davon nicht abgeleitet werden.

| Priorität | Maßnahme | Aktueller Prüfstatus |
| --- | --- | --- |
| P0 | `main`-Ruleset: Löschen + Force-Push blockieren, Active | **OFFEN / zu prüfen** |
| P0 | Privaten Schwachstellenmeldeweg in den Einstellungen erneut verifizieren | **ZU VERIFIZIEREN** |
| P0 | 2FA, Recovery und GitHub-App-Berechtigungen auf Kontoebene prüfen | **ZU VERIFIZIEREN** |
| P1 | Dependency Graph aktivieren/prüfen | **ZU VERIFIZIEREN** |
| P1 | Dependabot Alerts aktivieren/prüfen | **ZU VERIFIZIEREN** |
| P1 | Secret Scanning und Push Protection verifizieren | **ZU VERIFIZIEREN** |
| P1 | Actions-Workflows auf Rechte, SHA-Pinning, unsichere Trigger prüfen | **ZU VERIFIZIEREN** |
| P1 | CodeQL C/C++ testen und Wirksamkeit bewerten | **GEPLANT** |
| P1 | Backup und Restore für Quellcode/Metadaten | **ZU VERIFIZIEREN** |
| P2 | Freigabe- und Releaseprozess für Übernahme nach SASDGMBH festlegen | **GEPLANT** |

**Empfohlene Reihenfolge:** Konto absichern → `main` vor Löschung/Force-Push schützen → Meldeweg und Secret-/Dependabot-Einstellungen kontrollieren → Actions prüfen → Codeanalyse und Releases vertiefen.

## 14. Reviewrhythmus und Änderungsdokumentation

- **Bei Anlage eines Repositories:** DEV-/RELEASE-Profil festlegen und P0 einmal vollständig abarbeiten.
- **Vor jeder offiziellen Veröffentlichung:** Release-Check durchführen; P0- und relevante P1-Abweichungen bewusst entscheiden.
- **Monatlich:** neue Dependabot-/Code-Scanning-/Secret-Scanning-Alerts sichten; ungenutzte Tokens und Apps im Blick behalten.
- **Vierteljährlich:** Kontozugriffe, GitHub Apps, PATs, SSH Keys, Rulesets und Backup-Wiederherstellung auf Aktualität prüfen.
- **Nach Incident oder GitHub-Funktionsänderung:** Checkliste aktualisieren und Auswirkungen auf andere Repositories bewerten.

### Optionales Prüfprotokoll

| Datum | Repository | Kontrollpunkt | Status | Nachweis | Entscheidung / nächste Aktion |
| --- | --- | --- | --- | --- | --- |
| YYYY-MM-DD | owner/repo | Beispiel: `main`-Ruleset | UNGEPRÜFT | | |

## 15. Wichtige Grenzen: Was sich *nicht* zentralisiert

- Das persönliche Profil-Repository `Robin-Goerlach/Robin-Goerlach` ist für eine **öffentliche Dokumentationssammlung** geeignet; es vererbt **keine** Branch-Rulesets, Actions-Berechtigungen, Dependabot-Konfiguration oder Private-Vulnerability-Reporting-Einstellungen an andere Repositories.
- Für bestimmte **Default Community Health Files** wie `SECURITY.md` sieht GitHub ein separates öffentliches Repository mit dem speziellen Namen **`.github`** unter dem betreffenden Konto bzw. der Organisation vor. Verwendet wird eine solche Default-Datei nur, soweit ein Ziel-Repository keine eigene Datei desselben Typs hat.
- `Robin-Goerlach/.github` und `SASDGMBH/.github` hätten **unterschiedliche Geltungsbereiche**. Solche Standarddateien sind kein Ersatz für die eigentliche technische Konfiguration.
- Nicht jede im UI sichtbare Sicherheitsfunktion ist in jedem Repository/Plan mit denselben Möglichkeiten verfügbar. Den konkreten Zustand immer prüfen.

## 16. GitHub-Dokumentation / Nachschlagequellen

Offizielle Dokumentation (Funktionsumfang und Menünamen können sich ändern):

1. [GitHub Profile README](https://docs.github.com/en/account-and-profile/how-tos/profile-customization/managing-your-profile-readme)
2. [Default Community Health Files](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file)
3. [Securing your account with 2FA](https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa)
4. [2FA recovery methods](https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/configuring-two-factor-authentication-recovery-methods)
5. [Preventing unauthorized account access](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/preventing-unauthorized-access)
6. [GitHub rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)
7. [Rules available for rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)
8. [Private Vulnerability Reporting](https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities/configure-vulnerability-reporting/configure-for-a-repository)
9. [Quickstart for securing a repository](https://docs.github.com/en/code-security/getting-started/quickstart-for-securing-your-repository)
10. [Dependency Graph supported ecosystems](https://docs.github.com/en/code-security/reference/supply-chain-security/dependency-graph-supported-package-ecosystems)
11. [GitHub Actions settings](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository)
12. [Securely using `pull_request_target`](https://docs.github.com/en/actions/reference/security/securely-using-pull_request_target)
13. [CodeQL compiled-language build options](https://docs.github.com/en/code-security/reference/code-scanning/codeql/build-options-for-compiled-languages)
14. [Secret Scanning coverage](https://docs.github.com/en/code-security/reference/secret-security/secret-scanning-scope)
15. [OIDC for cloud providers](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-cloud-providers)

---

**Nächster Arbeitsschritt:** Die Checkliste zunächst als Dokument freigeben und danach Punkt für Punkt anhand der tatsächlichen GitHub-Einstellungen durchgehen. Das reine Anlegen der Markdown-Datei ändert **keine** GitHub-Sicherheitseinstellungen.
