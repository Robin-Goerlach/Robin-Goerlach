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

Quellen: [Managing Releases](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository), [Artifact Attestations](https://docs.github.com/en/actions/concepts/security/artifact-attestations), [git tag](https://git-scm.com/docs/git-tag).
