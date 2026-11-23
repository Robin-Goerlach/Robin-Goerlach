# GitHub Actions – CI und Automatisierung sicher betreiben

**Stand:** 2026-11-23 · **Dokumenttyp:** GitHub-Nachschlagewerk · **Sprache:** Deutsch

> Menüs, Tarifumfang und API-Details können sich ändern. Die aktuelle Einstellung eines Repositories ist gesondert zu überprüfen; diese Seite nimmt keine Änderungen an anderen Repositories vor.

## Aufbau und Vertrauensgrenzen
Workflows stehen als YAML-Dateien unter **.github/workflows/**. Ein Workflow wird durch **events** ausgelöst und enthält **jobs**, **steps**, Berechtigungen und optional Secrets/Environments. Der Auslöser ist sicherheitsrelevant: Eine PR aus einem Fork ist nicht vertrauenswürdig wie ein Push auf **main**.

## CI-Beispiel und sichere Ausführung

Das frühere Minimalbeispiel wurde durch ein konkretes, geprüftes CMake-CI-Muster im Abschnitt **„Überarbeitetes CMake-CI-Beispiel“** ersetzt. Die Beschreibung der Mindestmaßnahmen folgt hier, bevor das vollständige YAML weiter unten erscheint.

## Rechte des GITHUB_TOKEN
- Repositoryweit **permissions: contents: read** oder bei Bedarf **permissions: {}**.
- Schreibrechte für einzelne Jobs gezielt erweitern; Testjobs benötigen gewöhnlich keine Schreibrechte.
- PATs möglichst vermeiden; für automatisierte Änderungen geeignete GitHub-App-Rechte prüfen.
- **persist-credentials: false** bei Checkout, wenn nach dem Checkout keine authentifizierten Git-Operationen notwendig sind.

## Angriffsflächen und Gegenmaßnahmen
| Risiko | Gegenmaßnahme |
| --- | --- |
| Externe Action wird verändert | Pinnen auf vollständige Commit-SHA, Updates kontrollieren |
| Untrusted PR-Code mit Secrets | Secrets/Schreibrechte aus PR-Testjobs fernhalten |
| pull_request_target + Checkout des PR-Head | Nicht tun; Privilegien und untrusted Code strikt trennen |
| Direkte Shell-Auswertung von PR-Titel | Eingaben nicht unescaped in Shell-Quelltext interpolieren |
| Self-hosted Runner auf fremden PRs | Isolieren bzw. für untrusted PRs vermeiden |
| Falsche Runner-/Cache-Vertrauensannahme | Artefakte und Caches als möglicherweise manipulierbar behandeln |
| Dauerhafte Cloud-Zugänge | Bei Eignung OIDC mit kurzlebigen Berechtigungen |

## Reviews vor Änderungen an Workflows
1. Prüfen, welcher **event**-Trigger die Datei ausführt.
2. Effektive Token-Rechte auf Workflow- und Job-Ebene bestimmen.
3. Herkunft und Pinning jeder Fremd-Action prüfen.
4. Testen, ob untrusted Quellcode jemals mit privilegierten Rechten läuft.
5. Veröffentlichungskontext (Tag, Environment, Genehmigung) separat prüfen.

## Troubleshooting
**Workflow läuft nicht:** Syntax, Pfad, branch filter, erforderliche Berechtigung, deaktivierte Actions. **PR-Checks fehlen:** event, Statusnamen, Runner-Verfügbarkeit. **Release scheitert:** Environment-Regeln, Token-Scopes, restriktives Ruleset, Triggerref und Tag.

## Nicht verwechseln
Ein grüner Workflow bestätigt ausschließlich die implementierten Checks. Er beweist weder sicheren Quellcode noch die Abwesenheit von kompromittierten Dependencies.

## Überarbeitetes CMake-CI-Beispiel mit vollständigem SHA-Pinning

Für ein **öffentliches C++-/CMake-Repository** mit GitHub-gehostetem Runner kann dieser Entwurf als Startpunkt dienen. **Nicht** ungeprüft in das Profil-Repository kopieren: Dort liegt kein CMake-Projekt. Die gezeigte Checkout-SHA wurde gegen GitHubs Tag `actions/checkout v6.0.0` geprüft; die Version muss später gepflegt werden.

```yaml
name: cxx-ci
on:
  push:
    branches: [main]
  pull_request:
permissions:
  contents: read

jobs:
  build-and-test:
    runs-on: ubuntu-24.04
    timeout-minutes: 20
    steps:
      - name: Checkout sources
        uses: actions/checkout@1af3b93b6815bc44a9784bd300feb67ff0d1eeb3 # v6.0.0
        with:
          persist-credentials: false

      - name: Configure
        run: cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug

      - name: Build
        run: cmake --build build --parallel 2

      - name: Run tests
        run: ctest --test-dir build --output-on-failure --no-tests=error
```

**Voraussetzungen:** CMakeLists.txt im Repo-Root, Tests per `enable_testing()`/`add_test` registriert und für die benötigten Compiler/Bibliotheken eine funktionierende Runner-Umgebung. `--no-tests=error` verhindert das falsche „grün“ bei null gefundenen Tests. Ein realer Produktworkflow sollte **Buildmatrix, Abhängigkeiten, Cache, Lint und Security-Checks** projektbezogen ergänzen; kein blindes Kopieren.

### Warum das vorherige Beispiel unzureichend war

Der ursprüngliche didaktische `git diff --check HEAD^ HEAD`-Job war nicht robust: Für den ersten Commit fehlt **HEAD^**; ein automatisch erzeugter Merge-Commit aus einem Pull Request kann andere Parent-Beziehungen haben. Er wurde bewusst durch einen fachlich passenden CI-Entwurf ersetzt. Für Markdown-Repositories bietet sich eine gesonderte, getestete Link-/Lint-Prüfung an.

## Trust Boundaries: Welche Daten gelten als fremd?

| Trigger | Typische Gefahr | Regel |
| --- | --- | --- |
| `push` auf main | Ein kompromittierter Maintainer-Account | Branch-Schutz und eingeschränkte CI-Rechte |
| `pull_request` | PR enthält untrusted Code | Keine Produktionssecrets, möglichst read-only |
| `pull_request_target` | Läuft in privilegiertem Basis-Kontext | PR-Code **nicht** mit privilegiertem Token ausführen |
| `workflow_run` | Folgeworkflow vertraut einem fremden Artefakt | Herkunft/Integrität und Rechte separat absichern |
| `workflow_dispatch` | Manuelle Auslösung mit Parametern | Eingaben validieren, Rechte und Environment prüfen |
| `schedule` | Periodische Jobs auf Default-Branch | Update-Robustheit, Berechtigungen, Runner-Kosten |

**Besonders gefährlich:** Ein Job mit `pull_request_target`, der den PR-Head auscheckt und anschließend dessen Skripte ausführt. Der Kontext kann höhere Berechtigungen und Secrets besitzen; diese Trennung darf nicht aufgeweicht werden.

## Least-Privilege-Design mit mehreren Jobs

```yaml
permissions:
  contents: read

jobs:
  test:
    permissions:
      contents: read
    # Tests: kein Veröffentlichungs-Token

  # Nur als schematische Skizze, keine direkt ausführbare Jobdefinition:
  # publish:
  #   if: github.ref_type == 'tag'
  #   permissions:
  #     contents: write
  #   environment: release
  #   needs: test
```

Das obere Beispiel ist **eine Designskizze**, kein lauffähiger Workflow: Jobschlüssel brauchen in echten Dateien `runs-on` oder `uses` und passende Steps. Eine verpflichtende Releasefreigabe erfordert die tatsächliche Konfiguration der **Environment Protection Rules** und Branch-/Tag-Gates; das Wort `environment: release` allein garantiert keine manuelle Genehmigung.

### OIDC statt langlebiger Cloud-Credentials

Bei Cloud-Zugriffen möglichst kurzlebige Tokens über OIDC nutzen. Dafür `id-token: write` gezielt am benötigten Job vorsehen und **Trust Policies beim Cloud-Anbieter** restriktiv auf Repository, Workflow, Ref und Environment ausrichten. `id-token: write` gewährt noch keinen direkten Cloud-Zugriff, ermöglicht aber die Ausgabe eines austauschbaren OIDC-Tokens.

## Detailprüfungen für Workflows

**Workflow-Datei:** Welche `on`-Events sind aktiv? Welche `permissions` und `secrets`? Werden externe Actions mit Commit-SHA gepinnt?  
**Runner:** GitHub-hosted oder selbstgehostet; welche Netzwerksegmente und lokale Persistenz?  
**Cache:** Kann ein untrusted PR einen von privilegierten Jobs später genutzten Cache manipulieren?  
**Artefakte:** Werden heruntergeladene PR-Artefakte ausgeführt, ungeprüft entpackt oder veröffentlicht?  
**Shell:** Werden `${{ github.event.* }}`-Werte direkt in Skript-Code interpoliert statt als sicher übergebene Umgebungsvariable behandelt?  
**Release:** Können andere Branches als der freigegebene Tag/Commit das Publish-Script erreichen?

## Diagnose in fünf Schritten

1. Workflow-Auslöser, Ref und tatsächlichen Commit SHA bestimmen.
2. Effektive `GITHUB_TOKEN`-Rechte pro Job überprüfen.
3. Fehlgeschlagenen Step öffnen, nicht nur den Gesamtstatus betrachten.
4. Reproduzierbarkeit mit lokalem Toolchain-Setup prüfen.
5. Vor jedem Retry untersuchen, ob Geheimnisse, externe Downloads oder veränderte Berechtigungen beteiligt sind.

## Abnahmekriterien für SASD-Projekte

- CI läuft auf dem vorgesehenen Branch/PR-Typ und der relevanten Commit-Revision.
- Keine untrusted Eingaben in privilegierten Jobs.
- Fremd-Actions sind auf geprüfte vollständige SHAs fixiert.
- Tests laufen wirklich; **0 Tests** gilt nicht als „bestanden“.
- Veröffentlichungsrechte sind getrennt und nicht für PR-Jobs verfügbar.
- Neue Dependabot-PRs durchlaufen dieselben Tests und Sicherheitsprüfungen.

Siehe auch: [Actions Security](../security/GITHUB-REPOSITORY-SECURITY.md), [Release-Gate](../security/GITHUB-RELEASE-SECURITY.md).

Quellen: [GitHub Actions Workflow Syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax), [Secure Use](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions), [pull_request_target](https://docs.github.com/en/actions/reference/security/securely-using-pull_request_target).
