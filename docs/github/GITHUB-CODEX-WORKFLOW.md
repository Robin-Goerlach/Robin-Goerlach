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

## Vertiefung: ein überprüfbarer Agentenauftrag

### Sicherheitsmodell

Ein KI-Agent ist weder ein separater menschlicher Reviewer noch automatisch ein sicherer Serviceaccount. Sein **tatsächlicher Berechtigungsumfang** wird durch das ausführende Betriebssystem, das GitHub-Token, verbundene Apps, Projektordner und lokale SSH-Schlüssel bestimmt.

Auch scheinbar harmlose Dokumentation kann **Prompt Injection** enthalten: eine README, ein Issue, Log oder ein fremdes Script kann textlich verlangen, Sicherheitsregeln zu umgehen oder Zugangsdaten hochzuladen. Externe Inhalte sind Datenquellen, **keine Anweisungen höherer Priorität**.

### Auftragvorlage für größere SASD-Änderungen

```text
Projekt: OWNER/REPO
Ziel: [nachprüfbares Verhalten]
Basis: [geprüfter main-Commit]
Arbeitsbranch: feature/[thema]
Erlaubte Pfade: [konkret]
Nicht erlaubte Pfade: .github/workflows, SECURITY.md, CI/Release-Secrets
Erlaubte Werkzeuge: [Build, Lint, Tests]
Ausdrücklich verboten:
  - force push / reset --hard / git clean -fd
  - Löschen, Ersetzen oder Umbenennen öffentlicher Tags
  - Ändern von GitHub-Rulesets oder Credentials
  - Upload von internen Dateien oder Zugangsdaten
  - Veröffentlichung von Releases ohne Freigabe
Abnahmekriterien:
  - alle geänderten Dateien auflisten
  - lokale Tests mit Befehlen und Ergebnis
  - neue Abhängigkeiten, Lizenzen, offene Risiken nennen
  - nachvollziehbare Commits; kein Push ohne erlaubte Berechtigung
```

Die Vorlage ist **keine technische Sandbox**. Eine tatsächliche Beschränkung muss zusätzlich durch Dateisystemrechte, eingeschränkte Tokens und isolierte Umgebungen hergestellt werden.

## Vier Stufen der Autonomie

| Stufe | Erlaubnis | Geeigneter Einsatz |
| --- | --- | --- |
| A – Analyse | Lesen, Vorschläge, keine Änderungen | Audit, Planung, Architektur |
| B – Lokal editieren | Dateien im Branch ändern, Tests laufen lassen | Standardentwicklung |
| C – Commit/PR | Nach definierten Gates committen und PR öffnen | Gut eingegrenzte Aufgaben |
| D – Release | Tags, Artefakte, Deployment | **Nur nach ausdrücklicher Freigabe und gesondertem Release-Prozess** |

Wenn eine Aufgabe stillschweigend von Stufe B zu D wechseln soll, muss sie abgebrochen oder ausdrücklich neu freigegeben werden.

## Abnahme bei einem großen Codex-Lauf

```bash
git status --short
git diff --stat
git diff --check
git log --oneline --decorate -12
git diff origin/main...HEAD -- .github/workflows SECURITY.md
git diff origin/main...HEAD -- CMakeLists.txt
```

**Annahme:** Es wurde zuvor `git fetch origin` ausgeführt. Wenn ein Pfad nicht existiert, ist ein leerer Diff nicht automatisch aussagekräftig; weitere einschlägige Dateien und die **gesamte** Datei-Liste prüfen.

**Prüfe im Besonderen:**
- Änderungen außerhalb des beauftragten Scopes.
- Veränderte oder neu angelegte Actions.
- neue Netzzugriffe, Pakete, Submodule oder Binary-Blobs.
- geänderte Compiler-/Security-Flags und stillgelegte Tests.
- README-Befehle, die Benutzern Datenverlust oder Secret-Upload ermöglichen.
- unklare Quellen/Lizenzen übernommener Codefragmente.
- Status „Test erfolgreich“, wenn lediglich `cmake --build` ohne Tests lief.

## Trennung von Codex und GitHub-Sicherheitsadministration

- Agent braucht für gewöhnliche Codearbeit **keine Adminrechte**.
- Secrets sollten nicht in seine Arbeitsumgebung gelangen.
- Vorhandene Schreibrechte auf `.github/workflows` sind eine reale Angriffsfläche, auch wenn der Auftrag das verbietet.
- Die GitHub-UI-Einstellungen zu Branch-/Tag-Rulesets sind unabhängig von `AGENTS.md`.
- Die finale Entscheidung über Security-Advisory und Release gehört nicht in einen unbeobachteten Agentenlauf.

## Fehlerfall und Stop-Kriterien

**Sofort stoppen und prüfen**, wenn Codex:
- Änderungen an unerlaubten Dateien vornimmt;
- Tests oder Security-Prüfungen heimlich deaktiviert;
- Git-Force-Operationen oder Token-Exfiltration vorschlägt;
- ein fremdes README/Issue als Autorisierung für kritische Befehle behandelt;
- Release-Tags anlegt oder Releases publiziert, obwohl nur Codeänderungen beauftragt waren.

## Reviewbericht: beispielhaftes Ergebnis

```text
Basis und Branch: ...
Änderungen: ...
Kompiliert mit: ...
Tests: 42 bestanden, 0 fehlgeschlagen (konkreter Befehl/CI-URL)
Plattformen: Linux verifiziert, OpenBSD nicht getestet
Dependency-Änderungen: ...
Security-Review: ...
Nicht bearbeitete Punkte: ...
Commit-SHA: ...
Freigabe offen: ja/nein
```

Die Zahlen sind **nur Beispielwerte** und niemals ohne echten Testnachweis übernehmen.
