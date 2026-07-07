# Sync Repository 1:1 to Pelican (Strict SFTP)

This GitHub Actions workflow automatically mirrors the contents of your repository to your Pelican server via SFTP (using `lftp`) on every push. The script also deletes files on the server that have been removed from the Git repository to ensure an exact 1:1 copy.

## 📄 The Workflow Code

Create a file at `.github/workflows/sync-pelican.yml` with the following content:

```yaml
name: Sync Repository 1:1 to Pelican (Strict SFTP)

on:
  push:
    branches:
      - '*' # Triggers on every branch

jobs:
  deploy:
    runs-on: ubuntu-latest
    continue-on-error: true # Prevents GitHub from marking the job as failed and sending email alerts
    steps:
      - name: 1. Checkout Git code
        uses: actions/checkout@v4

      - name: 2. 1:1 Mirroring via LFTP (Strict SFTP)
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

## 🚀 Tutorial: How to Use This Workflow

1. **Create the folder**: Inside your repository, create the directory path `.github/workflows/` if it does not exist yet.
2. **Save the workflow**: Create a file named `sync-pelican.yml` inside that folder and copy the YAML code above into it.
3. **Configure Secrets**: Save your server credentials in your GitHub repository settings (see the guide below).
4. **Push to trigger**: As soon as you push code to any branch, the synchronization process will start automatically.

---

## 🔑 Where to Get the Variables (Secrets)

This workflow uses encrypted GitHub Secrets to prevent your sensitive passwords from being exposed in public code.

### Step-by-Step GitHub Setup Guide:
1. Navigate to your GitHub repository and click on the **Settings** tab.
2. In the left sidebar, click on **Secrets and variables** -> **Actions**.
3. Click the green **New repository secret** button.
4. Add the following three variables one by one:

| Secret Name | Description | Where to Find It |
| :--- | :--- | :--- |
| `PELICAN_SFTP_HOST` | Server Address | The IP address or domain name of your Pelican web server (e.g., `://yourserver.com` or `192.168.1.1`). |
| `PELICAN_SFTP_USER` | SFTP Username | The username for your SSH/SFTP access, provided by your web hosting provider or server administrator. |
| `PELICAN_SFTP_PASSWORD` | SFTP Password | The corresponding password for the SFTP user mentioned above. |

---

## ⚠️ Important Notices

* **Danger with `--delete`**: The `mirror --reverse --delete` command completely deletes any file on the server that does not exist in your Git repository. Ensure your target directory on the server is either empty or exclusively dedicated to this Git project!
* **Port 2022**: The workflow is currently hardcoded to use port `2022` (`-p 2022`). If your server relies on the standard SSH port `22`, change this number in the workflow file.
* **Error Handling**: Because `continue-on-error: true` is enabled, you don't get a e-mail if the sync fails (unfortunatly always), preventing unwanted notification emails.