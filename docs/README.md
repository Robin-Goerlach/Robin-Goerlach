# Technische Dokumentation – Git, GitHub und Security

**Stand:** 2026-11-23 · **Sprache:** Deutsch · **Lizenz:** Es gilt nur die ausdrücklich für das Repository veröffentlichte Lizenz; keine Annahmen.

Willkommen in der technischen Wissenssammlung des Profil-Repositories **Robin-Goerlach/Robin-Goerlach**. Hier stehen Erklärungen, reproduzierbare Übungen, praktische Entscheidungshilfen und Sicherheitschecklisten für den persönlichen Git-/GitHub-Einsatz und die SASD-Entwicklung.

## Inhalt

| Bereich | Einstieg | Für wen? |
| --- | --- | --- |
| **Git** | [git/README.md](git/README.md) | Versionsverwaltung, Branches, Recovery, CLI |
| **GitHub** | [github/README.md](github/README.md) | Repositories, Actions, PRs, Rulesets, Releases |
| **Security** | [security/README.md](security/README.md) | Kontosicherheit, Repository-Prüfungen und Releases |

### Schnellzugriff
- [Git-Alltagsworkflow](git/GIT-WORKFLOWS.md) – kleine direkte main-Commits und größere Feature-Änderungen.
- [Git Recovery](git/GIT-RECOVERY.md) – sichere Diagnose und Wiederherstellung.
- [GitHub Rulesets](github/GITHUB-RULESETS.md) – Minimalabsicherung für den Default-Branch.
- [GitHub Actions](github/GITHUB-ACTIONS.md) – CI und Sicherheitsgrenzen.
- [Codex-Workflow](github/GITHUB-CODEX-WORKFLOW.md) – autonome Änderungen kontrolliert abnehmen.
- [GitHub Security Checklist](security/GITHUB-SECURITY-CHECKLIST.md) – wiederholbare Sicherheitsprüfung.

## Zweck und Grenzen
**Hier dokumentieren wir Techniken und Entscheidungen.** Verbindliche SASD-Projektrichtlinien liegen getrennt im jeweiligen Entwicklungsstandard bzw. ausdrücklich eingebundenen Projektvorgaben. Diese Sammlung allein **ändert keine Branch-Rulesets, Repo-Einstellungen oder Kontorechte**. GitHub Profile-READMEs sind kein zentraler Policy-Verteiler.

## Konventionen
- Dokumente auf **Deutsch**, technische API-/CLI-Bezeichner im Original.
- Beispiele mit **OWNER/REPO**, **PFAD** und **example.invalid** sind Platzhalter.
- Riskante Befehle werden explizit markiert; Beispiele bevorzugen Wegwerf-Repositories.
- Beobachtungen und Empfehlungen klar auseinanderhalten.
- Bei Änderungen an GitHub-Oberfläche und Features Datum/Quelle aktualisieren.
- Querverweise möglichst relativ; externe Quellen bevorzugt aus offizieller Dokumentation.

## Pflege
Die Inhalte bei Änderungen an Git/GitHub oder dem eigenen Workflow prüfen. Sicherheitschecks in [security/GITHUB-SECURITY-CHECKLIST.md](security/GITHUB-SECURITY-CHECKLIST.md) mit Datum, Status und überprüfbarem Nachweis ausfüllen; **nicht** voreilig als bestanden markieren.

## Qualitätskontrolle und redaktionelle Regeln

Diese Sammlung soll **Lernmaterial, Runbook und Nachschlagewerk** sein. Bei Überarbeitungen werden Inhalt und technische Sicherheit mit denselben Maßstäben beurteilt wie Quellcode:

| Kriterium | Prüffrage |
| --- | --- |
| Korrektheit | Funktioniert der beschriebene Befehl im angegebenen Kontext? |
| Sicherheit | Könnte unbedachtes Kopieren Daten, Secrets oder Git-Historie gefährden? |
| Aktualität | Sind Git-/GitHub-Feature und Planumfang mit offizieller Quelle verifiziert? |
| Nachvollziehbarkeit | Werden Ziel, Vorbedingungen, Ablauf und erwarteter Erfolg genannt? |
| Scope | Handbuch oder **verbindlicher** SASD-Standard? |
| Transparenz | Was ist **beobachtet**, **empfohlen**, **ungeprüft** oder nur ein **Beispiel**? |
| Navigation | Funktionieren relative Links auf vorhandene Dokumente? |
| Release | Sind Risiken, Rollback- und Wartungskonsequenzen benannt? |

### Qualitätsstufen der Beispiele

- **Nur lesend:** erzeugt keine Versionierungsänderungen; einzelne Befehle können lokale Remote-Tracking-Daten aktualisieren (z. B. `git fetch`).
- **Wegwerf-Labor:** darf Commits/Branches lokal anlegen; Repository-Pfad ist ausdrücklich temporär.
- **Produktionsroutine:** benötigt sauberen Ausgangsstand, freigegebenen Branch, Tests und eine überprüfte Rückfallstrategie.
- **Designskizze:** erläutert Struktur, ist aber nicht ohne Ergänzung ausführbar.

Wenn ein Beispiel `OWNER/REPO`, `PFAD`, `DATEI.zip` oder andere Platzhalter enthält, muss es angepasst werden.

### Publikationsregel für sicherheitsbezogene Nachweise

**Öffentlich**: Methode, Anforderungen, Link auf allgemein zugängliche GitHub-Einstellungen und freigegebene Ergebnisse.  
**Privat**: echte Token-/Schlüsselinventare, Recovery-Prozedur mit sensiblen Details, interne Infrastruktur, unveröffentlichte Schwachstellenmeldungen und Zugangspfade.

Die Security-Checkliste ist absichtlich **eine öffentliche Vorlage**. Ein ausgefüllter Auditbericht mit sensiblen Informationen darf nicht in dieses öffentliche Profil-Repository committet werden.

## Weiterentwicklung

Neue Kapitel sollten immer mindestens **Ziel, Voraussetzungen, Hintergrund, sichere Beispiele, typische Fehler, Erfolgskriterien und Primärquellen** enthalten. Die Gliederung darf wachsen; das Repository soll jedoch kein zweiter, widersprüchlicher SASD-Development-Standard werden.

**Stand der fachlichen Überarbeitung:** 2026-11-23. Wesentliche Korrekturen betreffen sichere Actions-SHA-Pins, CMake/Dependabot/vcpkg-Abdeckung, CodeQL-C/C++-Buildmodus, persönliche versus organisatorische Repo-Richtlinien und sichere Recovery-Beispiele.
