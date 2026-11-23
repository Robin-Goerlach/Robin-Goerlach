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

Quellen: [git-config](https://git-scm.com/docs/git-config), [GitHub SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh).
