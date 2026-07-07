# Sync Repository 1:1 to Pelican (Strict SFTP)

Dieser GitHub Actions Workflow spiegelt den Inhalt deines Repositories bei jedem Push automatisch per SFTP (mittels `lftp`) auf deinen Pelican-Server. Das Skript löscht dabei auch Dateien auf dem Server, die im Git-Repository entfernt wurden, um eine exakte 1:1-Kopie zu gewährleisten.

## 📄 Der Workflow-Code

Erstelle eine Datei unter `.github/workflows/sync-pelican.yml` mit folgendem Inhalt:

```yaml
name: Sync Repository 1:1 to Pelican (Strict SFTP)

on:
  push:
    branches:
      - '*' # Reagiert auf jeden Branch

jobs:
  deploy:
    runs-on: ubuntu-latest
    continue-on-error: true # Verhindert, dass GitHub den Job als Fehlschlag wertet und E-Mails sendet
    steps:
      - name: 1. Code aus Git auschecken
        uses: actions/checkout@v4

      - name: 2. 1:1 Spiegelung via LFTP (Strict SFTP)
        run: |
          sudo apt-get update && sudo apt-get install -y lftp
          lftp -e "
            set sftp:connect-program 'ssh -a -x -p 2022 -o StrictHostKeyChecking=no';
            set sftp:auto-confirm yes;
            open -u '\${{ secrets.PELICAN_SFTP_USER }}','\({{ secrets.PELICAN_SFTP_PASSWORD }}' sftp://\){{ secrets.PELICAN_SFTP_HOST }};
            mirror --reverse --delete --verbose ./ ./;
            quit;
          "
```

---

## 🚀 Tutorial: So nutzt du den Workflow

1. **Datei erstellen**: Erstelle in deinem Repository den Ordnerpfad `.github/workflows/`, falls dieser noch nicht existiert.
2. **Workflow speichern**: Erstelle in diesem Ordner eine Datei namens `sync-pelican.yml` und kopiere den obigen YAML-Code hinein.
3. **Secrets einrichten**: Hinterlege die Zugangsdaten in den GitHub-Einstellungen (siehe Anleitung unten).
4. **Automatisch starten**: Sobald du Code in einen beliebigen Branch pushst, startet der Sync-Prozess von selbst.

---

## 🔑 Woher man die Variablen (Secrets) bekommt

Der Workflow nutzt verschlüsselte GitHub Secrets, damit deine Passwörter nicht öffentlich im Code stehen. 

### Schritt-für-Schritt-Anleitung in GitHub:
1. Gehe in deinem GitHub-Repository auf den Reiter **Settings** (Einstellungen).
2. Klicke in der linken Navigationsleiste auf **Secrets and variables** -> **Actions**.
3. Klicke auf den grünen Button **New repository secret**.
4. Trage die folgenden drei Variablen einzeln ein:

| Secret-Name | Beschreibung | Woher man es bekommt |
| :--- | :--- | :--- |
| `PELICAN_SFTP_HOST` | Die Server-Adresse | Die IP-Adresse oder Domain deines Pelican-Webservers (z. B. `://deinserver.com` oder `192.168.1.1`). |
| `PELICAN_SFTP_USER` | Der FTP/SFTP-Benutzername | Der Benutzername für deinen SSH/SFTP-Zugang, den du von deinem Webhoster oder Server-Administrator erhalten hast. |
| `PELICAN_SFTP_PASSWORD` | Das SFTP-Passwort | Das zugehörige Passwort für den oben genannten SFTP-Benutzer. |

---

## ⚠️ Wichtige Hinweise zum Workflow

* **Achtung bei `--delete`**: Der Befehl `mirror --reverse --delete` löscht gnadenlos alle Dateien auf dem Server, die nicht im Git-Repository existieren. Stelle sicher, dass der Zielordner auf dem Server leer ist oder nur für dieses Git-Projekt genutzt wird!
* **Port 2022**: Der Workflow ist aktuell fest auf den Port `2022` eingestellt (`-p 2022`). Wenn dein Server den Standard-Port `22` nutzt, passe die Zahl im Workflow-Code an.
* **Fehler-Ignorierung**: Durch `continue-on-error: true` wird der GitHub-Job auch bei Fehlern, der leider immer Auftritt, damit du keine nervigen E-Mails erhältst. Prüfe bei Problemen daher aktiv die Logs unter dem Reiter **Actions**.