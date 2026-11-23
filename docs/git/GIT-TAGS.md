# Git Tags – reproduzierbare Versionsmarken

**Stand:** 2026-11-23 · **Status:** Technisches Nachschlagewerk · **Sprache:** Deutsch

> Diese Sammlung erklärt Git, nicht die verbindliche SASD-Projektpolitik. Vor destruktiven Befehlen immer Arbeitsverzeichnis, Branch, Status und Backup prüfen.

## Lightweight versus annotated
**Lightweight** zeigt direkt auf einen Commit. **Annotated** speichert Tagger, Datum und Nachricht als eigenes Git-Objekt. Für Releases bevorzugen wir annotated Tags.

~~~bash
git status --short
git log -1 --oneline
git tag -a v0.1.0 -m "Release v0.1.0"
git show --no-patch v0.1.0
git push origin v0.1.0
git tag --list "v*"
~~~

## Tags sind kein vollständiger Release-Prozess
Tags sind zunächst nur Git-Referenzen. GitHub Releases ergänzen Notes und Artefakte. Zu einer belastbaren Veröffentlichung gehören: überprüfter Commit, passender Build, Testnachweis, Quellen-/Lizenzprüfung, Release Notes und gegebenenfalls Signaturen/Prüfsummen/Attestations.

**Nicht** nachträglich denselben veröffentlichten Versions-Tag auf einen anderen Commit setzen. Besser neue Patch-Version veröffentlichen und gegebenenfalls eine Korrektur/Advisory dokumentieren.

## Signieren
Je nach vorhandener Konfiguration unterstützt Git signierte Tags (GPG/SSH); Voraussetzung sind passende Schlüssel und Verifikationsmöglichkeiten. Eine Signatur beweist Besitz des Schlüssels, nicht Fehlerfreiheit des Codes.

~~~bash
git tag -v v0.1.0
git ls-remote --tags origin
~~~
**git tag -v** funktioniert nur mit signiertem Tag und einer passenden vertrauenswürdigen Verifikationskonfiguration.

## Empfohlene Konvention
- Versionspräfix **v**, z. B. **v1.2.3**.
- Vorabversionen deutlich kennzeichnen, z. B. **v1.0.0-rc.1**.
- Tags und Releases im offiziellen Repo gesondert schützen; Zugriffsrechte und Veröffentlichungskontext prüfen.

Quelle: [Git Tags](https://git-scm.com/book/en/v2/Git-Basics-Tagging).
