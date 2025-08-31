
# Vault Backup Automation

[![GitHub](https://img.shields.io/badge/GitHub-D3One-blue?style=flat&logo=github)](https://github.com/D3One)
![Shell Script](https://img.shields.io/badge/Shell_Script-%23121011.svg?style=flat&logo=gnu-bash&logoColor=white)

A robust shell script for automating encrypted backups of HashiCorp Vault's configuration and data. Designed for reliability and to be integrated into cron jobs or CI/CD pipelines.

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/5447a758-4c63-4e06-a38c-47d85d81be1b" />

## ⚠️ Disclaimer / Warning

**WARNING! USE AT YOUR OWN RISK!**
This script interacts directly with your HashiCorp Vault instance, a critical security component. Incorrect use can lead to:
* Data loss if backups are corrupted or overwritten.
* Service interruption if Vault commands are executed improperly.
* Exposure of sensitive data if encryption or file permissions are misconfigured.

**It is strongly recommended to:**
1. **TEST** this script thoroughly in a development or staging environment that closely mirrors your production setup.
2. **VERIFY** your backups regularly by performing test restores.
3. **SECURE** your backup directory and encryption keys with strict file permissions.
4. **UNDERSTAND** what the script does before deploying it in production.

The author is not responsible for any data loss, security breaches, or downtime caused by the use of this script.

---

## 📖 Overview

This script provides a simple yet powerful way to create consistent snapshots of your HashiCorp Vault deployment. It handles:
1. **Snapshot Creation:** Uses the Vault CLI to generate a snapshot of the Vault's data and configuration.
2. **Encryption:** Optionally encrypts the snapshot file using GPG for security at rest.
3. **Rotation:** Manages backup rotation to prevent unlimited disk space consumption.
4. **Logging:** Provides detailed logging for auditing and troubleshooting.

## 🚀 Features

*   **Full Snapshot:** Captures both the Vault's storage backend and configuration.
*   **GPG Encryption:** Secures the backup file using GnuPG (optional but recommended).
*   **Backup Rotation:** Automatically removes old backups based on a configurable retention policy.
*   **Comprehensive Logging:** Outputs success and error messages with timestamps for easy monitoring.
*   **Pre-flight Checks:** Validates that the Vault CLI is installed, authenticated, and available.

---

## 📁 Script Functionality: `vault_backup.sh`

The main script performs the following steps:

1.  **Initialization:** Sets up variables for backup directory, retention period, log file, and optional GPG recipient.
2.  **Pre-flight Checks:**
    *   Checks if the `vault` binary is available.
    *   Checks if the user is authenticated with Vault (`vault token lookup`).
    *   Creates the backup directory if it doesn't exist.
3.  **Snapshot Creation:** Executes `vault operator raft snapshot save` to generate the snapshot file.
4.  **Encryption (Optional):** If a GPG recipient is configured, encrypts the snapshot file for that key, leaving only the encrypted `.gpg` file and removing the plaintext snapshot.
5.  **Rotation:** Deletes backups older than the specified number of days, keeping only the most recent ones.
6.  **Logging & Cleanup:** Logs the outcome of each step and ensures no sensitive temporary files are left behind.

---

## 🛠️ Prerequisites

*   **HashiCorp Vault:** A running Vault cluster (version 1.4+ for reliable Raft snapshot support).
*   **Vault CLI:** Installed and authenticated on the machine running the script.
    *   The CLI must have the appropriate permissions (a token with a policy granting the `sys/storage/raft/snapshot` capability).
*   **GnuPG (Optional):** Required only if you want to encrypt the backups. The public key of the recipient must be in the keyring of the user running the script.

---

## 📝 Usage

### 1. Clone the Repository
```bash
git clone https://github.com/D3One/Vault-backup-automation.git
cd Vault-backup-automation
```

### 2. Configure the Script
Edit the variables at the top of the `vault_backup.sh` script to match your environment.

```bash
#!/bin/bash

# --- Configuration ---
BACKUP_DIR="/opt/vault/backups"  # Where to store backups
RETENTION_DAYS=7                 # Number of days to keep backups
LOG_FILE="/var/log/vault_backup.log" # Log file path
TIMESTAMP=$(date +"%Y%m%d_%H%M%S") # Timestamp for filename
GPG_RECIPIENT="user@example.com" # Email of GPG key recipient for encryption (comment out to disable)
# ---------------------
```

### 3. Make the Script Executable
```bash
chmod +x vault_backup.sh
```

### 4. Run the Script Manually
```bash
# Test run (ensure your VAULT_TOKEN or other auth method is set)
./vault_backup.sh
```

### 5. Automate with Cron
Add a line to your crontab (e.g., `crontab -e`) to run the backup daily at 2 AM.

```bash
# Example: Run every day at 2 AM
0 2 * * * /bin/bash /path/to/Vault-backup-automation/vault_backup.sh > /dev/null 2>&1

# Example: Run and capture output to a separate log (recommended)
0 2 * * * /bin/bash /path/to/Vault-backup-automation/vault_backup.sh >> /var/log/vault_cron.log 2>&1
```

---

## 🔒 Encryption

For maximum security, it is highly recommended to use GPG encryption.

1.  **Import the Public Key:** Ensure the public key of the backup recipient is in the keyring of the user running the cron job.
    ```bash
    sudo -u vault-user gpg --import recipient-public-key.asc
    ```
2.  **Configure the Script:** Set the `GPG_RECIPIENT` variable in the script to the recipient's email address.
3.  **How it works:** The script will create the snapshot, encrypt it for the specified recipient, and then securely delete the original plaintext file, leaving only the `.gpg` file.

**To decrypt and restore a backup:**
```bash
gpg --decrypt /opt/vault/backups/vault-backup-20231027_020000.snapshot.gpg > vault-restore.snapshot
vault operator raft snapshot restore vault-restore.snapshot
```

---

## 🗂️ Sample Backup File Structure

```
/opt/vault/backups/
├── vault-backup-20231027_020000.snapshot.gpg
├── vault-backup-20231026_020000.snapshot.gpg
├── vault-backup-20231025_020000.snapshot.gpg
└── vault_backup.log
```

---

## 👨‍💻 Authorship

*   **Author:** D3One
*   **GitHub:** [https://github.com/D3One](https://github.com/D3One)
*   **Repository:** [https://github.com/D3One/Vault-backup-automation](https://github.com/D3One/Vault-backup-automation)

Contributions, issues, and feature requests are welcome! Feel free to check the issues page or submit a pull request.

---

## 📜 License

This project is distributed under the MIT License. See the `LICENSE` file for more information.
