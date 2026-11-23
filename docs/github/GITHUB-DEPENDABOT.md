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

Quellen: [Dependabot Version Updates](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/dependabot-options-reference), [Dependabot Alerts](https://docs.github.com/en/code-security/dependabot/dependabot-alerts), [Supported Ecosystems](https://docs.github.com/en/code-security/reference/supply-chain-security/dependency-graph-supported-package-ecosystems).
