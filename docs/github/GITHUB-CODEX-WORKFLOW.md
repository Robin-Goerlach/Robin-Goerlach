# GitHub und Codex – autonome Entwicklung sicher einrahmen

**Stand:** 2026-11-23 · **Dokumenttyp:** GitHub-Nachschlagewerk · **Sprache:** Deutsch

> Menüs, Tarifumfang und API-Details können sich ändern. Die aktuelle Einstellung eines Repositories ist gesondert zu überprüfen; diese Seite nimmt keine Änderungen an anderen Repositories vor.

## Ausgangspunkt
Codex ist ein Werkzeug zur Codebearbeitung, aber keine unabhängige Freigabeinstanz. Seine effektiven Rechte ergeben sich aus Arbeitsumgebung, erreichbaren Dateisystemen, GitHub-Authentisierung, Plugins, Secrets und den erlaubten Befehlen.

## Minimaler Auftrag (anpassbares Muster)
> Ziel: [konkrete Änderung]. Arbeite ausschließlich in [Repository/Branch]. Änderungen an Secrets, GitHub Actions, Branch-Schutz, Releases und Deployment-Infrastruktur sind untersagt, soweit sie nicht ausdrücklich Auftragsgegenstand sind. Keine Force-Pushes, History-Rewrites oder destruktiven Git-Befehle. Vor Abgabe: Diff, Tests, geänderte Dateien, neue Abhängigkeiten und offene Risiken dokumentieren. Direkter Push nur bei ausdrücklicher Erlaubnis.

## Standardphasen
| Phase | Mensch | Codex |
| --- | --- | --- |
| Scope | Ziel, Pfade, Branch, Risiko festlegen | Betroffenen Code lesen |
| Implementierung | Grenzen vorgeben | Kleine überprüfbare Commits |
| Verifikation | Prüfmaßstab festlegen | Tests/Static Checks ausführen |
| Abnahme | Diff, CI und Risiken prüfen | Ergebnisse und Einschränkungen melden |
| Veröffentlichung | Bewusst freigeben | Keine autonome produktive Release-Freigabe |

## Besonderheiten beim direkten main-Workflow
Kleine, dokumentierte Änderungen dürfen bewusst direkt auf **main** erfolgen. Bei großen autonomen Läufen besser einen Arbeitsbranch wählen, selbst wenn nur ein Entwickler am Projekt arbeitet. Das reduziert die Wahrscheinlichkeit, dass ein unbemerkter Fehler sofort im Default-Branch landet.

## Zusätzliche Sicherheitsfragen
- Hat der Agent Zugang zu **SSH-Schlüsseln**, Tokens oder Cloud-Secrets?
- Darf er **.github/workflows** oder **AGENTS.md** verändern?
- Kann ein externer README-/Issue-/PR-Text den Agenten zu abweichenden Anweisungen verleiten (**Prompt Injection**)?
- Werden Downloads, Git-Submodule oder CI-Skripte aus Fremdquellen ausgeführt?
- Ist ein Restore ohne History-Rewrite möglich?
- Sind Tests reproduzierbar und ihre Ausführung nachvollziehbar?

## Übergabebericht (Vorlage)
~~~text
Repository/Branch:
Basis-Commit:
Ziel:
Geänderte Dateien:
Neue/aktualisierte Abhängigkeiten:
Ausgeführte Prüfungen und Ergebnisse:
Nicht ausgeführte Prüfungen:
Security- / Lizenzrisiken:
Commits / Pushes:
Offene Punkte:
~~~

## Schutz ist mehrschichtig
GitHub-Rulesets verhindern bestimmte Ref-Operationen; Tests erkennen manche Fehler; Diff-Reviews erkennen unerwünschten Scope; Backup sichert Recovery; Least-Privilege begrenzt einen kompromittierten Agenten. Keine Einzelmaßnahme genügt.

Siehe auch [Git Workflows](../git/GIT-WORKFLOWS.md), [Git Recovery](../git/GIT-RECOVERY.md), [Security Checklist](../security/GITHUB-SECURITY-CHECKLIST.md).
