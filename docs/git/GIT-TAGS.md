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

## Vertiefung: Tags, Versionierung und Vertrauenskette

### Was ist ein Tag technisch?

```bash
git rev-parse HEAD
git rev-parse v0.1.0
git cat-file -t v0.1.0
git rev-list -n 1 v0.1.0
```

Ein **annotated Tag** ist ein eigenes Git-Objekt und kann etwa eine Signatur enthalten. Ein **lightweight Tag** ist bloß eine Referenz direkt auf einen Commit. `git rev-parse v0.1.0` und `git rev-list -n 1 v0.1.0` liefern bei annotated Tags deshalb unterschiedliche Objekttypen/SHAs.

### Versionsschema und Semantik

Beispielhafte [Semantic Versioning](https://semver.org/lang/de/): `MAJOR.MINOR.PATCH`. Für Bibliotheken mit öffentlicher API/ABI kann ein **MAJOR**-Schritt für inkompatible Änderungen stehen, **MINOR** für rückwärtskompatible Erweiterungen und **PATCH** für kompatible Fehlerkorrekturen. Das Schema ist nur sinnvoll, wenn „öffentlich kompatibel“ im Projekt definiert ist.

| Tag | Sinn | Beispiel |
| --- | --- | --- |
| `v0.4.0` | Entwicklungsversion vor stabiler API | Neue experimentelle UI-Komponente |
| `v1.0.0-rc.1` | Release-Kandidat | Abnahmephase |
| `v1.0.0` | Stabile Version | Bewusstes Release |
| `v1.0.1` | Patch-Release | Korrektur ohne Breaking Change |

### Unveränderliche Veröffentlichungen

**Nie** einen bereits an Nutzer verteilten Versions-Tag stillschweigend verschieben. Sonst können zwei Nutzer mit dem vermeintlich gleichen Tag unterschiedliche Quellen erhalten. Wenn ein Fehler im Release vorliegt, neuen Commit und **neue Version** erstellen.

**Release-Tags schützen:** Im Zielrepository prüfen, ob Tag-Rulesets vorhanden und **Active** sind; sonst reicht das normative Dokument nicht aus.

### Signaturprüfung praktisch verstehen

```bash
git show --format=fuller --no-patch v0.1.0
git tag -v v0.1.0
git log -1 --show-signature
```

`git tag -v` funktioniert nur bei entsprechend signiertem Tag und passender Verifikationsumgebung. Eine „Verified“-Markierung ist kein Beleg dafür, dass Tests liefen, Quellcode frei von Problemen ist oder ein Release unabhängig freigegeben wurde.

### Welche Referenzen werden gepusht?

```bash
git push origin main          # Branch, nicht automatisch jeden Tag
git push origin v0.1.0        # genau diesen Tag
git push --follow-tags origin main
git ls-remote --tags origin
```

`--follow-tags` überträgt unter passenden Bedingungen **annotated Tags**, die von den zu pushenden Commits erreichbar sind. **Nicht** blind `git push --tags` nutzen, wenn alte lokale Experimente/Probe-Tags existieren.

### Kontrollliste vor Tagging

- Stimmt `main` mit dem zu veröffentlichenden Commit überein?
- Ist der Worktree sauber? Gibt es unpushte Änderungen?
- Sind CI und Security-Tests auf exakt dieser Revision erfolgreich?
- Ist die Lizenzprüfung abgeschlossen?
- Existiert der Tag schon remote?
- Können Artefakte, SHA-256 und Release Notes eindeutig zugeordnet werden?

**Achtung:** Tag- und Release-Schutz ist nicht automatisch vorhanden. Er muss in den Einstellungen des jeweiligen Repositories eingerichtet und geprüft werden.

## Mini-Labor
Zwei Tags auf denselben Test-Commit setzen – einen lightweight, einen annotated – und `git cat-file -t` sowie `git show` vergleichen. Danach die **lokalen** Test-Tags wieder entfernen, nicht etwa veröffentlichte Release-Tags.

Quelle: [Git Tags](https://git-scm.com/book/en/v2/Git-Basics-Tagging).
