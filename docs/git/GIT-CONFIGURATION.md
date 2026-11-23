# Git Configuration – Identität, Editor, SSH und Sicherheit

**Stand:** 2026-11-23 · **Status:** Technisches Nachschlagewerk · **Sprache:** Deutsch

> Diese Sammlung erklärt Git, nicht die verbindliche SASD-Projektpolitik. Vor destruktiven Befehlen immer Arbeitsverzeichnis, Branch, Status und Backup prüfen.

## Geltungsbereiche und Herkunft
Git nutzt u. a. systemweite, globale und repositorylokale Konfigurationen; lokale Werte haben für denselben Schlüssel normalerweise Vorrang. Auch **includeIf** und Umgebungsvariablen können Werte beeinflussen.

~~~bash
git config --list --show-origin --show-scope
git config --global user.name "Vorname Nachname"
git config --global user.email "adresse@example.invalid"
git config --local user.email "projekt@example.invalid"
git config --get user.email
git config --global init.defaultBranch main
~~~
Die Beispieladressen ersetzen; keine Zugangsdaten in Config-Dateien hinterlegen.

## Repositorybezogene Identität
Bei privaten und beruflichen Projekten jeweils explizit prüfen, welche Commit-E-Mail verwendet wird. Eine öffentliche GitHub-Noreply-Adresse kann die private E-Mail verbergen; dazu die GitHub-E-Mail-Einstellungen berücksichtigen.

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git var GIT_AUTHOR_IDENT
~~~

## SSH-Authentifizierung (Linux / WSL)
~~~bash
ls -la ~/.ssh
ssh -T git@github.com
git remote -v
git remote set-url origin git@github.com:OWNER/REPO.git
~~~
Vor dem ersten Vertrauen den veröffentlichten **GitHub-SSH-Host-Key-Fingerprint** mit GitHubs offizieller Dokumentation vergleichen. Niemals eine unerwartete Host-Key-Änderung blind bestätigen. Private Schlüssel und deren Passphrasen nicht in öffentliche Ausgaben kopieren.

**SSH-Schlüssel** autorisieren Git-Zugriffe; **Commit-Signaturen** weisen unabhängig davon Commit-Identitäten nach. Sie sind zwei verschiedene Funktionen.

## Zeilenenden, Plattformen und Editor
~~~bash
git config --global core.editor "vim"
git config --global pull.ff only
git config --get core.autocrlf
git check-attr -a -- README.md
~~~
**core.autocrlf** nicht blind ändern: Bei Windows/WSL-Konflikten besser eine projektbezogene **.gitattributes** (z. B. für *.sh text eol=lf) verwenden. **pull.ff only** verhindert implizite Merge-Commits beim Pullen, erfordert bei Divergenzen aber eine bewusste Entscheidung.

## Prüfung nach Neukonfiguration
~~~bash
git status --short
git remote -v
git config --list --show-origin
~~~
Vor Änderungen an Remotes URL und Berechtigungen kontrollieren. Benutzernamen, Passwörter oder Tokens nicht direkt in Remote-URLs speichern.

## Vertiefung: reproduzierbare Konfiguration unter Windows, WSL und Linux

Git liest Konfigurationen u. a. aus System-, Benutzer- und Repository-Ebene. Achte auf **welche Git-Installation** du verwendest: Windows-Git und WSL-Git lesen üblicherweise unterschiedliche Benutzerkonfigurationsdateien und Schlüsselumgebungen.

```bash
command -v git
git --version
git config --show-origin --show-scope --list
git config --get-regexp '^user\.'
git config --get-regexp '^core\.(autocrlf|eol|editor)$' || true
```

Der letzte Aufruf liefert ohne konfigurierte Werte keinen Treffer; das ist nicht automatisch ein Fehler.

### Zwei Identitäten ohne globale Nebenwirkungen

`git config --local` setzt Optionen im **aktuellen** Repository. Für getrennte Projektbereiche kann man `includeIf` nutzen:

```gitconfig
# Ausschnitt aus ~/.gitconfig
[includeIf "gitdir:~/Documents/Source/repositories/SASD/"]
    path = ~/.config/git/sasd.inc
```

In `~/.config/git/sasd.inc` könnten dann `user.name`, `user.email` und optional `commit.gpgsign` für den Bereich liegen. Vorab prüfen, welche Pfade WSL wirklich verwendet. Nicht blind setzen, wenn Home-Verzeichnisse oder gemountete Windows-Pfade unterschiedlich sind.

```bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git var GIT_COMMITTER_IDENT
```

### Zeilenenden als Projektentscheidung

Für Bash-Skripte sind LF-Zeilenenden wichtig. Eine kleine `.gitattributes` kann das plattformübergreifend absichern:

```gitattributes
* text=auto
*.sh text eol=lf
*.bash text eol=lf
*.ps1 text eol=crlf
*.png binary
*.jpg binary
*.zip binary
```

**Achtung:** `*.ps1` ist hier eine **bewusste Beispielentscheidung**, kein universelles Muss; PowerShell kann LF verarbeiten. Ändert man `.gitattributes` in einem etablierten Repository, Dateien kontrolliert neu normalisieren und Diff prüfen; nicht gleichzeitig fachliche Änderungen verstecken.

```bash
git check-attr -a -- scripts/example.sh
git ls-files --eol | head -30
```

### SSH: Host, Client, Agent und GitHub

```bash
ssh -G github.com | grep -E '^(hostname|user|identityfile|identitiesonly) '
ssh-add -l
ssh -T git@github.com
```

Vor einer ersten Host-Key-Annahme den Fingerprint mit [GitHubs veröffentlichten SSH-Fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints) vergleichen. **Nicht** mittels `StrictHostKeyChecking=no` die Prüfung umgehen. SSH-Schlüssel für die Authentifizierung sind nicht dasselbe wie Signaturschlüssel.

### Signierte Commits und Tags

Ein typischer Signatur-Workflow muss **zuerst** lokal eingerichtet und verifiziert werden, bevor ein Ruleset „Require signed commits“ verlangt.

```bash
git log -1 --show-signature
git verify-commit HEAD
git config --show-origin --get commit.gpgsign
```

Ein fehlgeschlagenes `verify-commit` kann schlicht einen **unsignierten** Commit anzeigen. Nicht jede Signatur wird auf jedem Rechner ohne passende Trust-Konfiguration positiv verifiziert; zusätzlich die Anzeige „Verified“ auf GitHub und deren Bedingungen prüfen.

## Sicherheitsfalle: Credential Helper

```bash
git config --show-origin --get-all credential.helper
git remote -v
```

Credentials können in Betriebssystem-Keychains, Credential-Managern oder (unsicher) Textdateien liegen. **Keine** Tokens in `https://TOKEN@github.com/...` oder Git-URLs speichern, weil solche Werte leicht in Logs, History und Remote-Konfiguration gelangen.

## Abnahme nach jeder Konfigurationsänderung

- Richtige Git-Installation und richtigen Account gewählt?
- `user.email` passend zum Projekt?
- Shell-Skripte mit LF?
- Remote-URL gehört wirklich zum beabsichtigten Repository?
- SSH-Host-Key geprüft und SSH-Agent nicht mit unnötigen Schlüsseln geladen?
- Neue Einstellung nur im beabsichtigten Scope wirksam?

**Fehlersuche:** Bei abweichender Identität nicht mehrfach blind `git config --global` ausführen, sondern zuerst `--show-origin`, `--show-scope` und `includeIf` prüfen.

Quellen: [git-config](https://git-scm.com/docs/git-config), [GitHub SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh).
