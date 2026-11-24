# GitHub Account Security – Zugriff auf das Entwicklerkonto absichern

**Stand:** 2026-11-24 · **Status:** Handbuch / empfohlene Prüfverfahren

> Eine Dokumentation ist kein Sicherheitsnachweis. Unbekannte GitHub-Einstellungen bleiben **nicht verifiziert**, bis sie im jeweiligen Konto oder Repository überprüft wurden. Öffentliche Dateien dürfen keine Geheimnisse, Recovery-Codes oder internen Systeme enthalten.

## 1. Anmeldeverfahren (P0)
**GitHub → Profilbild → Settings → Password and authentication** (Menü kann sich ändern).
- Starke und unterschiedliche Anmeldeinformationen für GitHub und die verknüpfte E-Mail-Adresse.
- 2FA aktiviert; wenn passend einen **Passkey bzw. FIDO2-Sicherheitsschlüssel** einsetzen.
- Mindestens einen belastbaren Wiederherstellungsweg besitzen; Recovery-Codes **außerhalb von GitHub** geschützt aufbewahren.
- Wiederherstellbarkeit praktisch prüfen, ohne geheime Codes in das öffentliche Dokument zu schreiben.
- Unbekannte Sitzungen/Geräte überprüfen und verdächtige Sessions beenden.

**Wichtig:** Aktivierte 2FA verhindert nicht den Missbrauch eines separat gestohlenen gültigen PAT oder SSH-Schlüssels.

## 2. Git-Authentisierung und Signaturen auseinanderhalten
| Mechanismus | Zweck | Sicherheitsfrage |
| --- | --- | --- |
| SSH Authentication Key | Git-Zugriff erlauben | Wer hat den privaten Schlüssel? |
| Fine-grained PAT | Git/API-Zugriff mit Scopes | Für welche Repositories und wie lange? |
| OAuth/GitHub App | Delegierter Zugriff | Welche Rechte? Welche Repositories? |
| Commit-/Tag-Signatur | Authentizität einer Signatur belegen | Ist der Signaturschlüssel vertrauenswürdig? |

## 3. SSH-Schlüssel pflegen (P0/P1)
~~~bash
ls -la ~/.ssh
ssh -T git@github.com
ssh-add -l
git remote -v
~~~
Diese Befehle zeigen nur lokal relevante Hinweise; **keine privaten Schlüssel, Passphrasen, Recovery-Codes oder Zugangstokens veröffentlichen**. Host-Key-Fingerprint vor erstmaliger Verbindung mit GitHubs offizieller Liste abgleichen. Nicht mehr verwendete Schlüssel vom GitHub-Konto entfernen; Passphrasen und sichere Speichermöglichkeiten prüfen.

## 3.1 SSH-Passphrase bequem nutzen: `ssh-agent` (P1)

Ein **SSH-Schlüssel mit Passphrase** ist sinnvoll, aber niemand möchte sie bei jedem `git pull` oder `git push` erneut eingeben. Ein **SSH-Agent** hält den entschlüsselten Schlüssel nach *einmaliger* Passphrase-Eingabe für weitere SSH-Verbindungen bereit. Die Passphrase wird dabei **nicht in die Git-Konfiguration geschrieben**. Voraussetzung ist, dass der öffentliche Schlüssel bereits im GitHub-Konto hinterlegt ist und das Repository eine SSH-Remote-URL (`git@github.com:OWNER/REPO.git`) verwendet.

### Debian/Linux und WSL – zunächst einfach und zeitlich begrenzt

Im Bash-Terminal, das den Schlüssel `~/.ssh/id_ed25519` verwendet:

```bash
# Einen Agenten für diese Shell starten (falls noch keiner erreichbar ist).
eval "$(ssh-agent -s)"

# Passphrase einmal eingeben; Schlüssel höchstens acht Stunden im Agenten halten.
ssh-add -t 8h ~/.ssh/id_ed25519

# Kontrollieren, welcher Schlüssel angeboten wird.
ssh-add -l
ssh -T git@github.com
```

Während diese Shell mit dem Agenten verbunden ist, fragen normale Git-SSH-Zugriffe üblicherweise nicht erneut nach der Passphrase. **Wichtig:** Der Befehl `eval "$(ssh-agent -s)"` startet einen **neuen** Agenten und sollte **nicht bei jedem neuen Terminal blind wiederholt werden**. Andere Shells oder ein Neustart von WSL erben die Verbindung nicht unbedingt.

**Bequemer für mehrere Bash-Terminals:** Auf Debian/WSL kann das optionale Paket `keychain` einen Agenten über mehrere Shell-Sitzungen wiederverwenden:

```bash
sudo apt install keychain
```

Nach Prüfung der vorhandenen SSH-Agent-Konfiguration diese Zeile **einmalig** in `~/.bashrc` ergänzen (nicht doppelt eintragen):

```bash
eval "$(keychain --eval --quiet ~/.ssh/id_ed25519)"
```

Damit wird ein vorhandener Agent nach Möglichkeit wiederverwendet und die Passphrase normalerweise nicht in jeder neu geöffneten Shell verlangt. **Die obige `keychain`-Variante setzt nicht automatisch das Acht-Stunden-Limit aus dem ersten Beispiel**; wer eine begrenzte Freigabe verlangt, muss das Lebenszeitlimit mit `ssh-add -t` bzw. einer geeigneten Agent-Policy separat sicherstellen.

### Windows 11 – eingebauten OpenSSH-Agent verwenden

**Einmalig in PowerShell mit Administratorrechten** den Windows-Dienst aktivieren:

```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

**Danach in einer normalen PowerShell** den Schlüssel hinzufügen und testen:

```powershell
ssh-add "$env:USERPROFILE\.ssh\id_ed25519"
ssh-add -l
ssh -T git@github.com
```

Die Passphrase wird beim Hinzufügen eingegeben; der Windows-Agent erleichtert danach weitere Git-Zugriffe. **Falls Git for Windows trotzdem ständig nachfragt**, verwendet es möglicherweise seine eigene `ssh.exe` statt des Windows-OpenSSH-Clients. Dann kann man in **Git for Windows** gezielt umstellen:

```powershell
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```

Diese Einstellung gilt für die jeweilige **Git-Installation** und sollte nicht pauschal für WSL-Git übernommen werden. Windows und WSL besitzen normalerweise getrennte SSH-Agenten und Home-Verzeichnisse; Schlüssel und Agenten nicht unüberlegt zwischen den Systemen kopieren oder weiterreichen.

### Komfort gegen Sicherheit abwägen

- **Empfehlung:** Passphrase **beibehalten**, Agent nutzen; nicht den Schlüssel unverschlüsselt speichern oder die Passphrase in Skripten hinterlegen.
- **Sitzungsdauer:** Ein länger laufender Agent ist bequemer. Während ein Schlüssel geladen ist, können jedoch Prozesse mit entsprechendem Zugriff auf den Agent-Socket möglicherweise Authentisierungen anfordern. `ssh-add -t 8h` begrenzt diese Zeit unter Linux/WSL; bei Windows den Lebenszyklus und die Dienstberechtigungen bewusst beurteilen.
- **Agent leeren (Linux/WSL und üblicher OpenSSH-Client):** `ssh-add -D` entfernt **alle** geladenen Identitäten aus dem erreichbaren Agenten, **nicht** die Schlüsseldateien. Für einen einzelnen Schlüssel: `ssh-add -d ~/.ssh/id_ed25519`.
- **Rechner verloren oder kompromittiert?** Betroffenen öffentlichen SSH-Schlüssel bei GitHub entfernen; bei Verdacht privaten Schlüssel als kompromittiert behandeln und neuen Schlüssel verwenden. Ein bloßes Leeren des Agenten reicht nicht aus.
- **Fehlersuche:** `ssh-add -l` zeigt geladene Schlüssel, `git remote -v` das Transportprotokoll. Bei `Permission denied (publickey)` Identität, Agent-Verbindung und auf GitHub hinterlegten **öffentlichen** Schlüssel prüfen.

**Quellen:** [GitHub – SSH-Schlüssel zum Agenten hinzufügen](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent), [GitHub – SSH-Passphrasen](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases), [Microsoft – OpenSSH-Schlüsselverwaltung](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement).

## 4. Tokens und App-Berechtigungen (P0)
Auf GitHub unter **Settings → Developer settings → Personal access tokens** und **Settings → Applications** prüfen:
1. PAT-Liste mit Zweck, Ablauf, Besitzer und Repository-Scope inventarisieren.
2. Wo möglich **fine-grained PATs**, minimale Rechte und kurze Laufzeit nutzen.
3. Verwaiste Tokens widerrufen; keine Tokens in Git, Shell-History, .env-Beispielen oder Chat-Protokollen zeigen.
4. GitHub-App-/OAuth-Berechtigungen regelmäßig überprüfen; nötigenfalls Repository-Zugriff einschränken.
5. Bot-/Automationszugriff vom persönlichen Hauptzugang trennen, sofern sinnvoll.

## 5. Aufbewahrung und Verlustfall
- Backup der relevanten lokalen Git-Repos und *separate* Verfahren für GitHub-Metadaten.
- Offline-Nachweis, wie nach Verlust des Rechners oder 2FA-Geräts der Zugriff wiederhergestellt werden kann.
- Beim Verlust eines Authentisierungsgeräts, bekannten Secret-Leak oder Verdacht auf Kompromittierung: betroffene Tokens/Keys **sofort widerrufen oder rotieren**, unbekannte Sessions beenden, Repository-/Actions-Zugriffe und Releases prüfen.

## 6. Checkliste mit Nachweisfeldern
| Kontrolle | Status | Prüfdatum | Nachweis **ohne Geheimnisse** |
| --- | --- | --- | --- |
| 2FA und Wiederherstellung | UNGEPRÜFT | – | |
| Verknüpfte E-Mail geschützt | UNGEPRÜFT | – | |
| SSH- und Signing-Schlüssel getrennt geprüft | UNGEPRÜFT | – | |
| SSH-Agent/Passphrase-Handhabung und Freigabedauer bewertet | UNGEPRÜFT | – | |
| PATs minimal berechtigt | UNGEPRÜFT | – | |
| OAuth- und GitHub-Apps geprüft | UNGEPRÜFT | – | |
| Backup-/Wiederherstellungsweg erprobt | UNGEPRÜFT | – | |

## Vertiefung: Prioritäten nach einem realistischen Angreifermodell

**Angriffspfade:** Phishing des GitHub-Logins, Diebstahl eines aktiven Sitzungstokens, Malware auf einem Entwicklerrechner, private SSH-Schlüssel ohne Passphrase, überberechtigte GitHub Apps, versehentlich veröffentlichte PATs und Verlust des einzigen 2FA-Geräts. Ein öffentlicher Account kann außerdem durch bösartige Issues oder Fork-PRs zur Preisgabe von Zugängen verleitet werden.

### Kontoschutz in der richtigen Reihenfolge

1. **E-Mail-Konto absichern.** Wer die Account-Recovery-Mail kontrolliert, kann unter Umständen eine wesentliche Sicherungsbarriere angreifen. E-Mail unabhängig mit starker MFA schützen.
2. **GitHub-MFA prüfen.** Bevorzugt phishingresistente Verfahren (Passkey/FIDO2). Eine SMS-Methode bietet anderen Schutz als ein Hardware-Schlüssel; nicht als gleichwertig behandeln.
3. **Wiederherstellung sichern.** Recovery-Codes an mindestens einem getrennten sicheren Ort aufbewahren. Die Anwesenheit der Codes kontrollieren, ohne sie im Chat, Repo oder Screenshot abzubilden.
4. **Sitzungen und Geräte prüfen.** Veraltete Zugriffe beenden; Verlustfälle in den Recovery-Plan aufnehmen.
5. **Maschinelle Zugriffe inventarisieren.** SSH-Keys, PATs, OAuth Apps, GitHub Apps und Deploy Keys getrennt betrachten.
6. **Schlüsselrotation/Notfallübung** durchführen, bevor eine ernste Störung eintritt.

## Schlüssel- und Berechtigungsinventar

Ein **privates** Inventar könnte folgende Metadaten enthalten; Geheimnisse dürfen dort nicht als Klartext stehen:

| Kennung | Art | Zweck | Scope | Letzte Nutzung | Ablauf/Rotation |
| --- | --- | --- | --- | --- | --- |
| DEV-LAPTOP-SSH | Auth SSH | Git Push | persönliches Konto | YYYY-MM-DD | bei Geräteverlust |
| CI-APP | GitHub App | Release-Automation | einzelnes Repo | YYYY-MM-DD | Installation prüfen |
| SIGNING-KEY | Commit-Signing | Herkunftsnachweis | gewünschte Identität | YYYY-MM-DD | eigener Prozess |

**Wichtig:** Ein SSH-**Authentication**-Key kann Git lesen/schreiben; ein SSH-**Signing**-Key kann Commits signieren. Beide sollten im Inventar unterschiedliche Zwecke erhalten, selbst wenn das technische Schlüsselformat gleich ist.

## PATs – Angriff und Gegenmaßnahme

**Classic PAT:** kann je nach Scope sehr breit wirken. **Fine-grained PAT:** kann mit beschränkter Repository-Auswahl und passenden Einzelberechtigungen zweckmäßiger sein, aber einzelne API-Aufgaben verlangen gegebenenfalls anderes. **GitHub App Installation Token:** für wiederkehrende Automatisierung häufig besser geeignet, weil Rechte/Scope präzise festgelegt und Tokens kurzlebig sind.

Für jedes Token fragen: **Wer verwendet es? Wie kann es entzogen werden? Was passiert bei einem Leak? Ist die Laufzeit so kurz wie möglich?** Tokens niemals durch Commit, PR-Kommentar, CI-Log, Debug-Ausgabe oder Shell-History „testen“.

## Lokale Git- und SSH-Diagnose ohne Secrets

```bash
git config --list --show-origin --show-scope
git remote -v
ssh-add -l
ssh -T git@github.com
```

Bei `ssh -T` kann GitHub die erfolgreiche Authentisierung melden und trotzdem mit einem für interaktives Shell-Login nicht erfolgreichen Exitcode enden; das ist nicht automatisch ein Zugriffsfehler. Vor einer ersten Host-Key-Annahme den Fingerprint gegen die [offiziell publizierten GitHub-Fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints) prüfen.

## Notfall: Kompromittierter Account oder Schlüssel

### Sofortmaßnahmen
- Verdächtige Zugriffe identifizieren und kompromittierte PATs/SSH-Keys/App-Installationen sperren oder widerrufen.
- Login-Sitzungen und verknüpfte E-Mail-Sicherheit prüfen.
- Repos, Actions-Logs, Tags, Releases, Webhooks und Integrationen auf unbefugte Änderungen kontrollieren.
- Aus bekannt sauberer Umgebung neue Authentisierungsdaten erstellen; nicht auf kompromittiertem Gerät blind neu anmelden.
- Betroffene Dritte und Team-/Organisationsverantwortliche nach Schweregrad informieren.

### Nachbereitung
- Ursache (Phishing, Geräteverlust, gestohlene App-Berechtigung) dokumentieren.
- Erneut **Wiederherstellbarkeit** üben; aus einem entfernten Gerät wieder Zugang erlangen können.
- Geprüfte Schutzmaßnahmen in einem **nicht öffentlichen** Sicherheitsprotokoll festhalten.

## Nachweisführung ohne Sicherheitsleck

**Öffentlich im Repository:** Richtlinie und Prüfschritte.  
**Privat dokumentiert:** konkrete Token-/Device-Namen, Sitzungsdaten, Recovery-Details, Login-Ereignisse, Screenshots mit Account-Identifikatoren.

Ein öffentlicher Checklisten-Haken „2FA geprüft“ ist nicht erforderlich, um 2FA zu betreiben. Öffentlich gepostete detaillierte Kontoinventare können dem Angreifer mehr nutzen als dem Leser.

Quellen: [GitHub Authentication](https://docs.github.com/en/authentication), [2FA Recovery](https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/configuring-two-factor-authentication-recovery-methods), [SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh), [PATs](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens).
