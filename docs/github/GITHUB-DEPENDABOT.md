# GitHub Dependabot – bekannte Risiken und kontrollierte Updates

**Stand:** 2026-11-23 · **Dokumenttyp:** GitHub-Nachschlagewerk · **Sprache:** Deutsch

> Menüs, Tarifumfang und API-Details können sich ändern. Die aktuelle Einstellung eines Repositories ist gesondert zu überprüfen; diese Seite nimmt keine Änderungen an anderen Repositories vor.

## Drei Mechanismen unterscheiden
1. **Dependency Graph:** GitHubs Sicht auf erkannte Abhängigkeiten.
2. **Dependabot Alerts/Security Updates:** Hinweise und ggf. PRs zu bekannten Schwachstellen.
3. **Dependabot Version Updates:** Geplante Versions-PRs aus **.github/dependabot.yml**.

Ein vorhandenes dependabot.yml aktiviert nicht automatisch jede Sicherheitsfunktion. Umgekehrt bedeutet ein leerer Dependency Graph **nicht**, dass ein Projekt keine Fremdbibliotheken nutzt.

## Minimales Update-Beispiel für GitHub Actions
~~~yaml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: monthly
    open-pull-requests-limit: 5
~~~
GitHub kann Action-Referenzen aktualisieren; bei Pinnen auf volle SHAs die Revisionen und Release Notes prüfen. **Version Updates** sind nicht dasselbe wie Alerts für verwundbare Abhängigkeiten.

## C/CMake: Grenzen der Erkennung
- Viele **find_package**-Aufrufe, manuelle Tarball-Downloads, lokale Bibliotheken, **FetchContent** und Git-Submodule werden nicht automatisch vollständig inventarisiert.
- Für C++-Projekte eine manuelle Dependency-Liste bzw. unterstützte Manifestformate und ggf. **Dependency Submission API** prüfen.
- CI muss Dependency-Update-PRs bauen, testen und gegen Lizenz-/ABI-/Kompatibilitätsanforderungen prüfen.
- Nicht eine schlecht unterstützte Abhängigkeit als geprüft markieren, nur weil ein grüner Dependabot-Status vorliegt.

## Review-Ablauf bei Alerts
1. Betroffenes Paket, Version, Datenquelle und tatsächlich erreichten Codepfad identifizieren.
2. Advisory-Schweregrad und Ausnutzbarkeit für das Projekt bewerten.
3. Fix-Version bestimmen; direkte und transitive Abhängigkeiten berücksichtigen.
4. Änderungen in eigenem Branch/PR testen, Compiler/Plattform-Matrix prüfen.
5. Update und Entscheidung dokumentieren; nötigenfalls Security-Advisory/Release planen.

## Für verschiedene Ökosysteme
| Projekt | Mögliche Quelle |
| --- | --- |
| C#/.NET | NuGet-Manifeste und Lockfile |
| Java | Maven/Gradle |
| JavaScript | npm/yarn/pnpm |
| GitHub Actions | workflow-Dateien |
| C++ | abhängig von vcpkg/Conan/anderen unterstützten Datenquellen |

**Warnung:** Nicht jedes Ökosystem wird von GitHub gleich umfassend erkannt. Vor dem Eintrag eines package-ecosystem die aktuell unterstützte Syntax und Voraussetzungen prüfen.

## Erweiterte C++-Analyse: was GitHub heute erkennt

**Präzisierung:** GitHub führt `vcpkg` inzwischen als eigenes Dependabot-`package-ecosystem` auf. Das erlaubt **bestimmte** Versionsupdates im Zusammenhang mit einem `vcpkg.json`-Manifest, zum Beispiel die Aktualisierung von `builtin-baseline`. Das bedeutet aber **nicht**, dass automatisch jede über CMake, `FetchContent`, `find_package`, Conan oder manuelle Downloads eingebundene Bibliothek erkannt wird.

Die aktuelle Übersicht ist [GitHub: Dependabot supported ecosystems](https://docs.github.com/en/code-security/reference/supply-chain-security/supported-ecosystems-and-repositories). Vor Aktivierung eines neuen Package-Ökosystems unterscheiden zwischen **Version Updates**, **Security Updates** und **Dependency-Graph-Abdeckung**.

### Beispiel: GitHub Actions plus vcpkg

```yaml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: monthly

  # Nur aktivieren, wenn das Projekt tatsächlich vcpkg.json verwendet
  # und GitHubs aktuelle vcpkg-Unterstützung den gewünschten Zweck erfüllt.
  - package-ecosystem: vcpkg
    directory: /
    schedule:
      interval: monthly
```

Dieses Beispiel ist **keine** universelle Konfiguration für das SASD-UI-Toolkit. Erst manifestbasierten Istzustand und Kompatibilität der verwendeten GitHub-Version prüfen. Im bestehenden Repository mit einer `.github/dependabot.yml` Änderungen **ergänzen**, nicht ohne Not überschreiben.

## Wie man die tatsächliche Abdeckung beweist

1. Liste der tatsächlichen Fremdkomponenten aus Build-Dateien, Lockfiles, Submodulen, Downloads und Build-Skripten erstellen.
2. **Insights → Dependency graph** und **Settings → Advanced Security** prüfen.
3. Pro Komponente feststellen: **im Graph vorhanden?** Welche Version und Quelle?
4. Alerts/Version-Updates als separate Zustände verifizieren.
5. Für fehlende Abdeckung ergänzende Tools oder manuelle Inventarisierung einsetzen.
6. In der Dokumentation **unbekannte Abdeckung** als offen, nicht als „keine Risiken“ kennzeichnen.

### Abhängigkeitsinventar (Muster)

| Komponente | Quelle | Tatsächliche Version | Im GitHub Graph? | Lizenz | Updateweg |
| --- | --- | --- | --- | --- | --- |
| CMake-Paket A | `find_package` | prüfen | prüfen | prüfen | manuell/Manager |
| Bibliothek B | `FetchContent` | Commit/Tag | prüfen | prüfen | eigener PR |
| Paket C | `vcpkg.json` | Manifest/Baseline | prüfen | prüfen | Dependabot nach Abdeckung |
| Actions | `.github/workflows` | SHA | prüfen | prüfen | Dependabot/Review |

## Security Alerts richtig priorisieren

Nicht nur CVSS-Schweregrad betrachten. Wichtige zusätzliche Fragen:
- Ist die gefährdete Funktion **wirklich erreichbar**?
- Wird das betroffene Modul in den Release-Artefakten eingebunden?
- Ist ein Exploit öffentlich bekannt und passt das Angriffsmodell?
- Welche unterstützten Versionen sind betroffen?
- Kann ein Update API/ABI, Datenformate oder Buildsystem brechen?
- Ist eine temporäre Mitigation erforderlich, bevor ein Fix verfügbar ist?

Bei einer als „nicht betroffen“ geschlossenen Alertmeldung **Begründung und Nachweis** dokumentieren. Veraltete grüne Dashboards sind kein Ersatz für die Prüfung neuer Upstream-Advisories.

## Bibliotheken außerhalb der GitHub-Erkennung

Für C++-Toolkits gegebenenfalls:
- explizite Dependency-Liste mit Herkunft/Version/Lizenz;
- deterministische Lock- und Baseline-Dateien;
- Build-/SBOM-Bericht;
- SCA-Scanner passend zum Paketmanager;
- Updates in isolierten Branches mit Testmatrix;
- regelmäßige Prüfung von Hersteller-/Projekt-Sicherheitsmeldungen.

## Warum Dependabot nicht alles automatisch lösen darf

Ein Update-PR kann eine kompromittierte Release-Version, neue transitive Abhängigkeiten oder Lizenzänderungen enthalten. Deshalb müssen auch automatisch erzeugte Updates **Diff-, Quellen-, Lizenz- und CI-Prüfung** durchlaufen. Nie alle Dependency-PRs allein aufgrund eines grünen Versionssignals blind mergen.

Quellen: [Dependabot Version Updates](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/dependabot-options-reference), [Dependabot Alerts](https://docs.github.com/en/code-security/dependabot/dependabot-alerts), [Supported Ecosystems](https://docs.github.com/en/code-security/reference/supply-chain-security/dependency-graph-supported-package-ecosystems).
