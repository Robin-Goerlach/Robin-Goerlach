# GitHub Security Checklist – Persönliche Entwicklung und SASDGMBH

**Version:** 2.0 (ausführlich überarbeitete Prüfvorlage)  
**Stand:** 2026-11-23  
**Tatsächlicher Ablageort:** `Robin-Goerlach/Robin-Goerlach/docs/security/GITHUB-SECURITY-CHECKLIST.md`  
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

## 17. Praktische Audit-Playbooks (ohne Konfigurationsänderung)

Ein Audit besteht aus **Zugriff auf den tatsächlichen Zustand**, **Kontrolle der Wirksamkeit**, **Risikoentscheidung** und **Nachprüfung**. Ein auf GitHub veröffentlichtes Dokument kann all dies nicht allein durchführen.

### 17.1 Lokale Git-Bestandsaufnahme (P0/P1)

```bash
git remote -v
git branch -vv
git fetch origin
git status -sb
git log --oneline --decorate -10
git ls-remote --heads origin
git ls-remote --tags origin
```

**Erwartet:** `origin` ist das beabsichtigte Repository; der eigene Branch ist bekannt; `main` hat eine nachvollziehbare Historie. Diese Befehle zeigen **keine** GitHub-Rulesets, Security-Alerts oder Accountrechte. Für eine reine Lesekontrolle kann `git fetch` die lokalen Remote-Tracking-Referenzen aktualisieren, aber keine versionierten Worktree-Dateien überschreiben.

### 17.2 Optional: GitHub CLI lesend verwenden

Nur wenn `gh` lokal installiert, mit dem richtigen Account authentifiziert und für den betroffenen Scope ausreichend berechtigt ist:

```bash
gh auth status
gh repo view OWNER/REPO --json name,owner,visibility,defaultBranchRef
gh api repos/OWNER/REPO/rulesets
```

**Unterschied zwischen Daten und Schlussfolgerung:** Eine leere Ruleset-Liste sagt nichts Verlässliches über **klassische Branch-Protection** und nicht zugängliche Organisationsrichtlinien. Ein 403/404 bei einem nur mit Administrator-Token lesbaren Endpunkt bedeutet **„nicht verifizierbar mit diesen Rechten“**, nicht „deaktiviert“.

**Nicht** aus `gh auth status` komplette Token-Informationen oder Fehlerlogs mit Secrets in öffentliche Berichte kopieren.

### 17.3 Main-Branch-Schutz nachweisen

| Feld | Zu erfassende Information |
| --- | --- |
| Repository + Standardbranch | z. B. `OWNER/REPO`, `main` |
| Regeln | Restrict deletions, Block force pushes |
| Enforcement | **Active**, nicht nur konfiguriert |
| Zielbedingung | Default branch / Include / Exclude |
| Bypass | Rollen, Apps, Ausnahmen |
| Klassische Protection | Vorhanden/keine Zugriffsberechtigung/gesichtet |
| Funktionstest | Normale Pushes funktionieren weiterhin |
| Nachweis | Datum, Settings-Link/privater Screenshot |
| Nächste Prüfung | Bei Rechte-/Ruleset-Änderung |

**Kein** absichtlicher Force-Push oder Löschversuch auf `main` als Test.

### 17.4 Private Vulnerability Reporting nachweisen

1. Öffentliches Repository und Berechtigung zur Einstellung prüfen.
2. **Settings → Advanced Security** bzw. aktuelle Code-Security-Seite öffnen.
3. **Private vulnerability reporting**-Zustand lesen; **Enabled** nachweisen.
4. **Security → Advisories** aufrufen; für externe Forscher sollte nach Aktivierung ein vertraulicher Meldeweg möglich sein.
5. `SECURITY.md` auf korrekte, **tatsächlich existierende** Kontaktangaben prüfen.
6. Zuständigkeit für den Eingang und Bearbeitungs-/Antwortprozess klären.
7. Ergebnis mit Datum protokollieren – keine künstliche Test-Schwachstelle öffentlich posten.

Eine aktivierte Funktion ist noch keine Zusage über Antwortfristen oder Wartungsdauer.

### 17.5 Dependency-Sicht gegen reale Projektdateien prüfen

Für C++/CMake beispielhaft **nur lesend**:

```bash
git ls-files CMakeLists.txt '**/CMakeLists.txt' '*.cmake'   vcpkg.json conanfile.py conanfile.txt .gitmodules
git submodule status
git grep -nE 'FetchContent_|ExternalProject_|find_package\(' -- '*.cmake' CMakeLists.txt || true
```

Die Erkennung ist absichtlich **heuristisch**. Der `git grep`-Aufruf erfasst je nach Pfadmuster keine verschachtelten CMakeLists oder dynamischen Includes; auch nicht initialisierte Submodule können Sonderfälle erzeugen. Keine vollständige Materialinventur allein aus diesen Kommandos behaupten.

Eine prüfbare Tabelle enthält: **Dependency, tatsächliche Version/Commit, Herkunft, Lizenz, Nutzung im Release, im Dependency Graph sichtbar?, Updatepfad, letzter Audit**.

**C++-Sonderfall:** GitHub unterstützt bestimmte **vcpkg**-Version-Updates, aber nicht die vollständige Erkennung beliebiger CMake-/Conan-/FetchContent-Abhängigkeiten. Im Audit Version Updates, Security Updates und Dependency Graph als getrennte Funktionen behandeln.

### 17.6 Actions- und Agentenprüfung

Bei jedem Workflow:
- Welcher **Event-Trigger**? `push`, `pull_request`, `pull_request_target`, `workflow_run`, `workflow_dispatch`?
- Welche **globalen und jobbezogenen `permissions`**?
- Welcher Code wird ausgecheckt? Wird fremder PR-Code mit privilegierten Credentials ausgeführt?
- Welche Dritt-Actions werden verwendet, sind sie auf **vollständige Commit-SHA** gepinnt?
- Sind `secrets`, `environment`, OIDC und Runner vertrauenswürdig eingegrenzt?
- Ist der Build tatsächlich ausgeführt worden? **Wie viele Tests** liefen?
- Können Codex oder andere Agenten die Workflow-Dateien im Auftrag verändern?
- Gibt es eine klare Trennung zwischen CI und Publishing?

**Nachweis:** Workflow-Datei, Trigger/Commit, Run-URL, effektive Rechte (soweit einsehbar), Testanzahl, Abweichungen.

## 18. Kontrollregister: Status, Priorisierung und Risikoakzeptanz

Diese Checkliste beschreibt Kontrollen. Für tatsächliche Prüfungen sollte ergänzend ein **privates Kontrollregister** verwendet werden. Es darf nie vertrauliche Details in einem öffentlichen Profil-Repository preisgeben.

### 18.1 Statussemantik

| Status | Bedeutet | Was zusätzlich erforderlich ist |
| --- | --- | --- |
| `UNGEPRÜFT` | Kein hinreichender aktueller Nachweis | Prüftermin |
| `ERFÜLLT` | Kontrollziel wurde verifiziert | Datum, Geltungsbereich, Nachweis |
| `OFFEN` | Anforderung nicht erfüllt oder Lücke erkannt | Abhilfe, Owner, Frist |
| `N/A` | Im konkreten Kontext nicht anwendbar | Begründung |
| `NICHT VERIFIZIERBAR` | Fehlende Berechtigung oder technische Einsicht | Nachprüfung/Eskalation |
| `RISIKO AKZEPTIERT` | Bewusste Abweichung | Entscheider, Gründe, Ablaufdatum |

**Regel:** `UNGEPRÜFT` ist kein Fehlerbefund, aber **nicht** gleich `ERFÜLLT`. `RISIKO AKZEPTIERT` ist keine Sicherheitsfunktion.

### 18.2 Prüfvorgang mit Kontroll-ID

```text
Kontroll-ID: GH-SEC-001
Prüfobjekt: OWNER/REPO
Profil: DEV|RELEASE
Kontrollziel: Branch main nicht löschbar/force-pushbar
Priorität: P0
Status: UNGEPRÜFT|ERFÜLLT|OFFEN|N/A|NICHT VERIFIZIERBAR|RISIKO AKZEPTIERT
Ist-Zustand:
Nachweismethode:
Nachweisort (geschützt, falls sensibel):
Prüfdatum:
Prüfer:
Befund / Auswirkung:
Gegenmaßnahme:
Verantwortlicher:
Frist / Ablauf der Ausnahme:
Nächster Review:
```

### 18.3 Prioritätsentscheidung (kein starres CVSS-Mapping)

**P0:** Gefahr für Accountkontrolle, Release-Integrität, Produktions-Secrets, unautorisierte Schreibrechte oder kritische Sicherheitsfunktionen vor Veröffentlichung.  
**P1:** Risiken, die die Erkennung und Eindämmung von Schwachstellen oder die Wartbarkeit beeinträchtigen.  
**P2:** Verbesserungen der Verteidigungstiefe, zusätzlichen Nachweisführung oder langfristigen Automatisierung.

Die Priorität richtet sich nach **Exposition und Wirkung**. Für einen öffentlich erreichbaren API-Server ist z. B. eine fehlende Authentisierung unter Umständen P0, unabhängig davon, ob sie im generischen Dokument unter „Codeanalyse“ steht.

### 18.4 Ausnahmen sind befristet

```text
Abweichung:
Warum ist die empfohlene Maßnahme derzeit nicht umsetzbar?
Risiko und Auswirkungen:
Ausgleichsmaßnahmen:
Entscheider und Datum:
Spätestes Neubewertungsdatum:
Technischer Nachweis, dass die Ausnahme nicht größer geworden ist:
```

Bei ernstem verbleibendem Risiko darf ein RELEASE nicht allein durch einen formalen „akzeptiert“-Vermerk freigegeben werden; Freigabeverantwortung und Konsequenzen sind getrennt zu prüfen.

## 19. Incident-Response-Ablauf für GitHub-Projekte

### 19.1 Verdacht: Account-/Token-Kompromittierung

**Eindämmung vor Bereinigung:** betroffene Credentials sperren/rotieren, neue authentisierte Zugriffe aus sauberer Umgebung vorbereiten, aktive Sessions und installierte Apps kontrollieren. Erst dann Commits, Workflow-Logs, Tags, Releases und geklonte Repositories untersuchen.

**Nicht** zuerst die Historie kosmetisch säubern: Bereits veröffentlichte Schlüssel müssen als kompromittiert betrachtet werden.

### 19.2 Verdacht: kompromittierter CI-Workflow

- Release- und Deployment-Jobs vorsorglich stoppen.
- Workflow-Revision, Trigger, Runner, Token-Scopes und genutzte Artefakte sichern.
- Fremd-Actions, Build-Downloads, Caches und Folgeworkflows überprüfen.
- Mit minimalen Rechten in sauberer Umgebung reproduzieren.
- Betroffene Veröffentlichungen und Repositories identifizieren.

### 19.3 Vertraulich gemeldete Schwachstelle

1. Eingang intern einordnen, Kontaktweg respektieren.
2. Ausnutzbarkeit und betroffene unterstützte Versionen untersuchen.
3. Fix privat planen, Tests/Fuzzing/Regressionsprüfungen ausführen.
4. Neue sichere Version erstellen.
5. Advisory, Upgrade-Anleitung und ggf. CVE abgestimmt publizieren.
6. Nachbereitung und Kontrollverbesserung dokumentieren.

### 19.4 Unerwünschter Commit auf main

Bei normalem Fehler ohne kompromittierte Berechtigungen: `git revert` auf geteiltem `main` bevorzugen. Bei vermutetem Angriff **nicht** nur einen Revert ausführen, sondern kompromittierte Schlüssel/Apps, CI, Releases und externe Integrationen prüfen.

### 19.5 Backup und Wiederherstellung

**Quellcode:** `git bundle` oder Git-Mirror plus unabhängiger Speicher.  
**GitHub-Metadaten:** Issues, PRs, Releases, Download-Assets, Projekte und relevante Einstellungen separat.  
**Secrets:** niemals im Klartext als Teil eines öffentlichen oder ungeschützten Backups archivieren.  
**Nachweis:** Wiederherstellung in eine isolierte Umgebung mit geprüftem Commit, Branches, Tags und benötigten Artefakten.

## 20. Review-Agenda je Monat, Quartal und Release

| Intervall | Aktivität | Erfolgskriterium |
| --- | --- | --- |
| Bei Repo-Anlage | Owner/Visibility, Lizenz, Branchschutz, PVR, Berechtigungen | Alle P0-Punkte bewertet |
| Monatlich | Dependabot, Security Alerts, Actions, PRs | Keine ungeklärten kritischen Befunde |
| Vierteljährlich | 2FA-Recovery, Tokens/Apps, Rulesets, Backups | Rechte und Restoretest dokumentiert |
| Vor offiziellem Release | Commit/Tag, CI, Analyse, SBOM/Lizenz, Artefakte | Gateprotokoll freigegeben |
| Nach Incident | Schlüsselsperrung, Ursachenanalyse, Lessons Learned | Wiederholung erschwert, Nachkontrolle terminiert |
| Nach GitHub-Funktionsänderung | Einstellungen und Beispiele auf Aktualität prüfen | Dokumentationsstand bestätigt |

## 21. Freigabeentscheidung: bewusst klein anfangen

Für öffentliche persönliche DEV-Repositories sind **direkte main-Commits nicht automatisch unsicher**, wenn Basisstand, Diff und Tests kontrolliert werden. Minimaler zusätzlicher Nutzen: aktives Ruleset für Löschung/Force-Push, sichere Kontoanmeldung, begrenzte Tokenrechte und ein vertraulicher Security-Meldeweg.

Für offizielle SASDGMBH-Releases muss die Freigabe stattdessen nachweisbar sein: getestet, zurückverfolgbar, rechtlich zugeordnet und mit einem realen Patch-/Incident-Prozess verbunden. **Eine zweite menschliche Prüfung nicht vortäuschen**, solange du alleiniger Maintainer bist.

## 22. Dokumentenpflege

- Dieses Dokument ist ein **lebendes Prüfschema**, kein statischer Zertifizierungsbericht.
- GitHub-Menünamen, Planumfang und Unterstützungslisten mindestens bei relevanten Änderungen erneut mit offiziellen Quellen vergleichen.
- Konkrete **Ist-Zustände** gehören in ein separates, datiertes Register; nicht in pauschale Beispieltexte.
- Die persönliche Sammlung unter `Robin-Goerlach/Robin-Goerlach` definiert keine automatischen Einstellungen für `SASDGMBH`-Repositories.
- Verbindliche Anforderungen des **SASD-Development-Standard** haben getrennte Versions- und Freigabeprozesse.

**Quellen zur Vertiefung:**
- [GitHub Rulesets und Durchsetzung](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)
- [Private Vulnerability Reporting](https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities/configure-vulnerability-reporting/configure-for-a-repository)
- [GitHub Secure Use of Actions](https://docs.github.com/en/actions/reference/security/secure-use)
- [Dependency Graph Ecosystems](https://docs.github.com/en/code-security/reference/supply-chain-security/dependency-graph-supported-package-ecosystems)
- [Dependabot Ecosystems](https://docs.github.com/en/code-security/reference/supply-chain-security/supported-ecosystems-and-repositories)
- [CodeQL for Compiled Languages](https://docs.github.com/en/code-security/how-tos/find-and-fix-code-vulnerabilities/manage-your-configuration/codeql-for-compiled-languages)

---

**Arbeitsweise:** Diese Prüfliste ist auf GitHub veröffentlicht. Reale Sicherheitseinstellungen und Prüfresultate werden gesondert kontrolliert; die Markdown-Datei verändert sie nicht. Vertrauliche Nachweise gehören nicht in das öffentliche Profil-Repository.
