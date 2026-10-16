# Git Backup — K2

Git Backup watches the Klipper configuration folder (`/mnt/UDISK/printer_data/config/`) and pushes every change to a GitHub repository.

!!! note "Entware required"
    Install [Entware](entware-k2.md) (option 13) first. Git Backup installs the packages it needs (`git`, `git-http`, `inotifywait`) automatically.

!!! tip "Want macros?"
    Install [Klipper Gcode Shell Command](gcode-shell-command.md) (option 18) **before** Git Backup to also get the `GIT_BACKUP_*` macros.

---

## Prerequisites

1. Create a [GitHub account](https://github.com/signup) if you don't have one.
2. Create a new **private** and **empty** repository on GitHub for your printer config — **no README, no license, no .gitignore**. The first push fails if the repository already has commits.
3. Generate a GitHub Personal Access Token:
    - Go to [GitHub → Settings → Tokens (classic)](https://github.com/settings/tokens)
    - Click **Generate new token (classic)**
    - Check the **repo** scope
    - Copy the token — you will not be able to see it again

---

## Installation

From the helper script menu, select **option 17 — Git Backup** and enter when prompted:

- GitHub username
- GitHub email address (if GitHub rejects the push because it would publish a private email, use your `...@users.noreply.github.com` address)
- Repository name
- Branch (default `main`)
- Personal access token (hidden input)

The script then:

1. Stores the token in `/mnt/UDISK/helper-script/.git-credentials` — readable by root only, **outside** the config folder (which the web interfaces serve) and **not** part of the remote URL.
2. Creates a `.gitignore` in the config folder that excludes `backups/`, `*.tar.gz`, `*.bkp` (plus `.moonraker.conf.bkp`, `*.pyc`, `__pycache__/`).
3. Initialises the repository, commits everything as *Initial Backup* and pushes it. If the push fails (repository not found, wrong token, repository not empty), the error is explained and the local changes are undone.
4. Installs the procd service `/etc/init.d/git-backup`. It watches the config folder with `inotifywait` and commits and pushes **10 seconds after a change** (bursts such as `SAVE_CONFIG` end up in one commit). A failed push (e.g. offline) is retried with the next change.
5. If Gcode Shell Command is installed: writes `git_backup.cfg` with the macros below and includes it from `printer.cfg`.

!!! note
    The config folder must not already be a git repository. If `/mnt/UDISK/printer_data/config/.git` exists, remove it first.

---

## Usage

### From the helper script

Selecting **option 17** again (when Git Backup is installed) opens:

```
  1) Back up now
  2) Pause automatic backups (until reboot)
  3) Resume automatic backups
  0) Back
```

### Klipper macros

Only available when [Klipper Gcode Shell Command](gcode-shell-command.md) was installed before Git Backup (otherwise install it and reinstall Git Backup):

| Macro | Description |
|---|---|
| `GIT_BACKUP_NOW` | Commit and push the config folder now |
| `GIT_BACKUP_PAUSE` | Stop automatic backups until `GIT_BACKUP_RESUME` or the next reboot |
| `GIT_BACKUP_RESUME` | Resume automatic backups |

### Via SSH

```bash
/etc/init.d/git-backup stop     # pause until reboot
/etc/init.d/git-backup start    # resume
```

---

## Removing

Remove menu (option 20) → **17) Git Backup**. This stops and removes the service, removes `git_backup.cfg` and its include, deletes the stored token, and asks whether to also delete the local git history (`config/.git` and `.gitignore`). **Your GitHub repository is not touched.**
