# GitHub Account Security – Zugriff auf das Entwicklerkonto absichern

**Stand:** 2026-11-23 · **Status:** Handbuch / empfohlene Prüfverfahren

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
| PATs minimal berechtigt | UNGEPRÜFT | – | |
| OAuth- und GitHub-Apps geprüft | UNGEPRÜFT | – | |
| Backup-/Wiederherstellungsweg erprobt | UNGEPRÜFT | – | |

Quellen: [GitHub Authentication](https://docs.github.com/en/authentication), [2FA Recovery](https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/configuring-two-factor-authentication-recovery-methods), [SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh), [PATs](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens).
