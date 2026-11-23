# GitHub Releases – vom getesteten Commit zum Artefakt

**Stand:** 2026-11-23 · **Dokumenttyp:** GitHub-Nachschlagewerk · **Sprache:** Deutsch

> Menüs, Tarifumfang und API-Details können sich ändern. Die aktuelle Einstellung eines Repositories ist gesondert zu überprüfen; diese Seite nimmt keine Änderungen an anderen Repositories vor.

## Ein Release besteht aus mehr als einem Git-Tag
**Git-Tag** referenziert einen Versionsstand, **GitHub Release** stellt Notes, Downloads und Metadaten bereit, **CI-Artefakte** sind technische Build-Ausgaben. Sie müssen zusammenpassen.

## Release-Gate für SASD-Projekte
1. Versionierung und Release-Commit festlegen, auf sauberem Worktree arbeiten.
2. Lizenz, Attribution, Quellcode und Fremdkomponenten prüfen.
3. CI für **genau diesen Commit** erfolgreich ausführen; relevante Testmatrix dokumentieren.
4. Bei C/C++ Compiler-/Plattformmatrix und statische Analysen bewerten.
5. SECURITY.md, unterstützte Versionen und vertraulichen Meldeweg kontrollieren.
6. Versionstag **einmalig** anlegen; veröffentlichte Tags nicht verschieben.
7. Binaries/Pakete in kontrollierter Umgebung erzeugen und prüfen.
8. Release Notes mit Changes, Breaking Changes, bekannten Risiken und Prüfsummen veröffentlichen.
9. Artefakte gegen Tag/Commit zurückverfolgen können; ggf. Signatur/Attestation erstellen.
10. Rückruf-/Patchweg und Verantwortlichkeit dokumentieren.

## Beispiel lokaler Tag
~~~bash
git status --short
git switch main
git fetch origin
git merge --ff-only origin/main
git log -1 --oneline
git tag -a v0.1.0 -m "Release v0.1.0"
git show --no-patch v0.1.0
git push origin v0.1.0
~~~
Nur im **Test- oder bewusst freigegebenen Release-Repository** ausführen; Version und Commit müssen vorher abgestimmt sein.

## CI und Upload
Ein Publish-Workflow braucht andere Rechte als ein Test-Workflow. **permissions: contents: write** nur gezielt am Release-Job, idealerweise mit geschütztem Environment, passendem Tag-Trigger und unveränderlichen Actions-Versionen.

Kein automatisches Veröffentlichen bei normalen PRs oder beliebigen Branch-Pushes. Externe PRs dürfen keine Release-Tokens oder Produktionsschlüssel erhalten.

## Supply Chain und Verifikation
| Frage | Nachweis |
| --- | --- |
| Aus welchem Code gebaut? | Commit SHA, Tag, Workflow Run |
| Mit welchen Werkzeugen? | Compiler/SDK/Runner/Container-Version |
| Wurden Tests bestanden? | CI-Checks und Artefaktbericht |
| Wurde das Artefakt unverändert geladen? | SHA-256/Signatur/Attestation |
| Welche Versionen sind betroffen? | Release Notes, Security Advisory |

Für Sicherheitsvorfälle im Nachhinein lieber neues Patch-Release statt stillschweigend einen vorhandenen Download auszutauschen.

## Vertiefung: ein belastbares Veröffentlichungsprotokoll

### Drei Artefakte unterscheiden

1. **Source-Referenz:** commit SHA und idealerweise annotated Tag.
2. **Build-Nachweis:** CI-Run, Toolchain, Betriebssystem, Build-Flags, verwendete Abhängigkeiten.
3. **Verteilbares Produkt:** ZIP, Binary, Paket, Docker-Image oder signierter Installer.

Bei einem Release müssen diese Artefakte zueinander passen. Wird ein Binary nach dem Test neu gebaut, muss sichergestellt sein, dass dieser zweite Build denselben freigegebenen Quellenstand und dieselben Anforderungen verwendet.

### Freigabematrix

| Prüfung | Beispielnachweis | Bei Fehler |
| --- | --- | --- |
| Tag zeigt auf erwartete SHA | `git rev-list -n 1 v1.0.0` | Release stoppen |
| Quellstand sauber | `git status`, CI-Run | Commit/Build klären |
| Testmatrix | Compiler-/OS-/Target-Liste | Umfang dokumentieren |
| Security Alerts | Bewertete offene Befunde | Risikoentscheidung oder Blockade |
| Abhängigkeiten und Lizenzen | SBOM/Inventory | Nachprüfung |
| Binary Integrität | SHA-256, Signatur, Attestation | Neu erzeugen |
| Release Notes | Breaking Changes, bekannte Probleme | Nachtragen |
| Verantwortliche Freigabe | Datum, Decision Log | Nicht automatisch publizieren |

### Release-Entscheidung für C++-Toolkits

Für ein wiederverwendbares C++-Toolkit zusätzlich prüfen:
- Header- und ABI-Kompatibilität, Compiler-Matrix, C++-Standard.
- Installierbarkeit mit **CMake Package Config**, Export-Targets und Release-Archiv.
- Integrierbare Tests durch einen **Consumer** in separatem Buildverzeichnis.
- Verhalten auf Linux/Unix-Plattformen einschließlich nicht getesteter Zielsysteme.
- Compilerwarnungen, Sanitizer, statische Analyse und bekannte Parser-/Speichergrenzen.

Bei einem Repository ohne fertiges CMake-Installpaket keine solche Fähigkeit in Release Notes behaupten, bis das konkrete Szenario erfolgreich getestet wurde.

## Integrität der Artefakte

```bash
sha256sum dist/PACKAGE.zip
```

Das funktioniert in Bash/WSL/Linux mit entsprechendem Tool; Pfad ist ein Platzhalter. Eine Prüfsumme erkennt Veränderung, **wenn** die Vergleichssumme aus einer vertrauenswürdigen Quelle stammt.

### Attestation- und Signaturunterschied

- **Prüfsumme:** technisch gleiche Bytes erkennen.
- **Digitale Signatur:** Verwendung eines bestimmten privaten Schlüssels prüfen.
- **Build Attestation:** Herkunftsdaten zur Build-Ausführung verifizieren; sie behauptet nicht die Sicherheit des Quellcodes.
- **Reproducible Build:** unabhängig nachbaubare gleiche Artefakte nach streng beschriebenen Eingaben – deutlich aufwendiger als nur eine feste Commit-SHA.

Für GitHub Artifact Attestations gelten **tarif- und sichtbarkeitsabhängige Voraussetzungen**. Im öffentlichen Free-Kontext ist die Funktion grundsätzlich nutzbar; Schreibberechtigungen/Workflow-Actions nach aktueller Dokumentation einrichten. Siehe [GitHub Artifact Attestations](https://docs.github.com/en/actions/how-tos/secure-your-work/use-artifact-attestations/use-artifact-attestations).

## Veröffentlichungen sicher zurückziehen

Ein sicherheitskritisches Release nicht einfach stillschweigend ersetzen. Stattdessen:
1. Betroffene Versionen dokumentieren.
2. Zugangsschlüssel gegebenenfalls rotieren.
3. Fix aus überprüftem Commit erzeugen.
4. Neue Version/Tag freigeben.
5. Advisory/Release Notes mit Migrations- bzw. Update-Empfehlung publizieren.
6. Weiter bestehende Mirror-Downloads und Paketregistry-Einträge berücksichtigen.

## Release-Vorlage für den Entscheidungsdatensatz

```text
Repository (persönlich / SASDGMBH):
Produkt/Version:
Veröffentlichungszeitpunkt:
Tag und vollständige Commit-SHA:
Quellcode- und Drittkomponenten-Lizenzen:
CI-Run-IDs und erwartete Tests:
Getestete Plattformen / nicht getestete Plattformen:
Build/Toolchain/Dependency-Lock:
SHA-256 / Signatur / Attestation:
SECURITY.md und unterstützte Versionen:
Freigabeverantwortlicher:
Offene Risiken und Entscheidung:
Vorgehen für Hotfix/Rollback:
```

**Keine Secrets oder privaten Sicherheitsmeldungen im öffentlichen Freigabeprotokoll erfassen.**

Weiterführend: [Release Security](../security/GITHUB-RELEASE-SECURITY.md), [Git Tags](../git/GIT-TAGS.md).

Quellen: [Managing Releases](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository), [Artifact Attestations](https://docs.github.com/en/actions/concepts/security/artifact-attestations), [git tag](https://git-scm.com/docs/git-tag).
