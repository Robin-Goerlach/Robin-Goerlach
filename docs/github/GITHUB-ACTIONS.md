# GitHub Actions – CI und Automatisierung sicher betreiben

**Stand:** 2026-11-23 · **Dokumenttyp:** GitHub-Nachschlagewerk · **Sprache:** Deutsch

> Menüs, Tarifumfang und API-Details können sich ändern. Die aktuelle Einstellung eines Repositories ist gesondert zu überprüfen; diese Seite nimmt keine Änderungen an anderen Repositories vor.

## Aufbau und Vertrauensgrenzen
Workflows stehen als YAML-Dateien unter **.github/workflows/**. Ein Workflow wird durch **events** ausgelöst und enthält **jobs**, **steps**, Berechtigungen und optional Secrets/Environments. Der Auslöser ist sicherheitsrelevant: Eine PR aus einem Fork ist nicht vertrauenswürdig wie ein Push auf **main**.

## Minimales CI-Beispiel (nur Dokumentation)
~~~yaml
name: docs-check
on:
  push:
    branches: [main]
  pull_request:
permissions:
  contents: read
jobs:
  whitespace:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          persist-credentials: false
      - name: Check whitespace
        run: git diff --check HEAD^ HEAD
~~~
**Wichtige Einschränkung:** Das ist nur ein *didaktisches Beispiel* und **nicht** sofort als Workflow für jedes Repository geeignet: **HEAD^** existiert nicht zwingend bei Initial-Commits; außerdem kann ein PR-Merge-Commit anders aussehen. Für Produktion Ereignisse/Referenzen korrekt vergleichen und Action-Versionen nach Prüfung auf vollständige unveränderliche Commit-SHAs pinnen. Hier wird nichts automatisch in .github/workflows installiert.

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

Quellen: [GitHub Actions Workflow Syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax), [Secure Use](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions), [pull_request_target](https://docs.github.com/en/actions/reference/security/securely-using-pull_request_target).
