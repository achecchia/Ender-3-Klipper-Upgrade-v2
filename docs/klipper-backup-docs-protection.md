# Klipper-Backup Docs Protection

This page documents the issue where the Raspberry Pi's Klipper-Backup process deleted GitHub documentation files, and the fix that prevents it from happening again.

The goal is simple:

```text
The Raspberry Pi may back up printer config files.
The Raspberry Pi must not manage, delete, stage, or overwrite the docs folder.
```

---

## Current Safe Workflow

Normal local printer config backup:

```text
Press update_git in Mainsail
```

When any page/file is added or edited directly in GitHub first:

```text
1. Press PULL_FROM_GIT in Mainsail
2. Confirm it finishes and lists the docs folder
3. Press update_git in Mainsail only after the pull succeeds
```

The `PULL_FROM_GIT` macro is intentionally pull-only. It does not push anything.

It runs:

```bash
cd $HOME/config_backup && git restore . && git pull --ff-only && git status --short && ls -la docs
```

A good Mainsail console result includes:

```text
Command {pull_from_git_script} finished
```

and a docs listing that includes the current docs files, including:

```text
klipper-backup-docs-protection.md
```

---

## What Happened

The repository documentation files were created directly in GitHub:

```text
docs/upgrades.md
docs/configuration.md
docs/future-ideas.md
docs/project-history.md
docs/punch-list.md
```

Later, the Mainsail `update_git` button was used. That button runs the local Klipper-Backup script on the Raspberry Pi.

The Pi's local backup working folder did not have the GitHub-created `docs/` folder at that time. The backup script treated the Pi's local folder as the source of truth and pushed a backup commit.

Result:

```text
GitHub docs were deleted because they were missing from the Pi's local backup folder.
```

---

## Root Cause

The dangerous part of the original Klipper-Backup script was this section:

```bash
git rm -r --cached . >/dev/null 2>&1
git add .
```

That logic means:

```text
1. Untrack everything
2. Re-add whatever currently exists on the Pi
3. Anything missing locally is treated as deleted
```

Because `docs/` did not exist locally on the Pi, Git staged those files as deleted and pushed that deletion to GitHub.

There were also cleanup commands that removed everything from the local backup folder except a few Git-related files:

```bash
find "$backup_path" -maxdepth 1 -mindepth 1 ! -name '.git' ! -name 'README.md' ! -name '.gitmodules' -exec rm -rf {} \;
```

That cleanup behavior made it even easier for documentation files to be missing locally.

---

## Fix Applied

### 1. Backed up the Klipper-Backup script

Before editing the script, a local backup was made:

```bash
cp ~/klipper-backup/script.sh ~/klipper-backup/script.sh.backup-before-docs-protection
```

---

### 2. Changed Git staging behavior

The dangerous block was replaced.

Old behavior:

```bash
# Untrack all files so that any new excluded files are correctly ignored and deleted from remote
git rm -r --cached . >/dev/null 2>&1
git add .
```

New behavior:

```bash
# Stage only Klipper config files.
# Do NOT let Klipper-Backup stage repo documentation changes/deletions.
# Documentation is managed from GitHub/ChatGPT, not from the Raspberry Pi.
git add -A printer_data/config/

# Extra safety: never include docs or README changes in automatic Pi backups.
git reset -- docs/ README.md >/dev/null 2>&1 || true
git checkout -- docs/ README.md >/dev/null 2>&1 || true
```

Why this works:

```text
Only printer_data/config/ is staged.
docs/ and README.md changes are explicitly unstaged/restored before committing.
```

This prevents the Pi from committing documentation deletions.

---

### 3. Protected the docs folder from local cleanup

The script had cleanup commands that removed files from the local backup folder after syncing.

Old cleanup command:

```bash
find "$backup_path" -maxdepth 1 -mindepth 1 ! -name '.git' ! -name 'README.md' ! -name '.gitmodules' -exec rm -rf {} \;
```

New cleanup command:

```bash
find "$backup_path" -maxdepth 1 -mindepth 1 ! -name '.git' ! -name 'README.md' ! -name '.gitmodules' ! -name 'docs' -exec rm -rf {} \;
```

The important addition is:

```bash
! -name 'docs'
```

This keeps the local `docs/` folder from being deleted by the backup script.

Both cleanup lines in `~/klipper-backup/script.sh` were patched.

Verified with:

```bash
grep -n "find \"\$backup_path\"" ~/klipper-backup/script.sh
```

Expected result:

```text
Both cleanup lines include: ! -name 'docs'
```

---

### 4. Added post-cleanup restore

After the final cleanup command, this was added so the local backup repo is left clean after the script runs:

```bash
# Leave the backup repo clean after the post-backup cleanup.
cd "$backup_path"
git restore . >/dev/null 2>&1 || true
```

Verified section near the bottom of the script:

```bash
# Remove files except .git folder after backup so that any file deletions can be logged on next backup
find "$backup_path" -maxdepth 1 -mindepth 1 ! -name '.git' ! -name 'README.md' ! -name '.gitmodules' ! -name 'docs' -exec rm -rf {} \;

# Leave the backup repo clean after the post-backup cleanup.
cd "$backup_path"
git restore . >/dev/null 2>&1 || true
```

---

## Pull From Git Mainsail Button

A dedicated Mainsail macro/button was added so GitHub-side documentation changes can be pulled into the Pi's local backup repo before running a backup.

The macro is named:

```text
PULL_FROM_GIT
```

It lives in:

```text
printer_data/config/shell_command.cfg
```

Macro config:

```ini
[gcode_macro PULL_FROM_GIT]
description: Pull latest GitHub repo changes into the local Klipper-Backup repo without pushing
gcode:
    RUN_SHELL_COMMAND CMD=pull_from_git_script

[gcode_shell_command pull_from_git_script]
command: bash -c "cd $HOME/config_backup && git restore . && git pull --ff-only && git status --short && ls -la docs"
timeout: 90.0
verbose: True
```

This button should be used whenever documentation or repo files are changed directly in GitHub before pressing the normal `update_git` backup button.

Purpose:

```text
PULL_FROM_GIT = pull GitHub changes down to the Pi
update_git     = push current printer config backup up to GitHub
```

Keep these two actions separate. Pull first, then push/back up.

---

## Disabled Automatic Backup Services

The Pi had automatic Klipper-Backup services enabled. These were disabled so backups only happen intentionally.

Disabled services:

```text
klipper-backup-on-boot.service
klipper-backup-filewatch.service
```

Reason:

```text
Automatic backups can push before the Pi has pulled GitHub-side documentation changes.
Manual backups are safer for this repo because documentation is also maintained directly in GitHub.
```

Commands used:

```bash
sudo systemctl disable klipper-backup-on-boot.service
sudo systemctl stop klipper-backup-on-boot.service

sudo systemctl disable klipper-backup-filewatch.service
sudo systemctl stop klipper-backup-filewatch.service
```

Verification:

```bash
systemctl list-units --all | grep -i klipper-backup
systemctl is-enabled klipper-backup-filewatch.service
systemctl is-enabled klipper-backup-on-boot.service
```

Expected result:

```text
disabled
disabled
```

Manual Mainsail buttons still work because they call the script directly. Disabling these services only stops automatic background backups.

---

## Pulling Restored Docs to the Pi

After the GitHub docs were recreated, the Pi's local backup repo was updated:

```bash
cd ~/config_backup
git pull
```

Then the docs folder was verified:

```bash
ls -la docs
```

Expected files currently include:

```text
configuration.md
future-ideas.md
klipper-backup-docs-protection.md
project-history.md
punch-list.md
upgrades.md
```

---

## Final Verification Test

After patching, the backup script was tested manually:

```bash
cd ~/config_backup
git restore .
~/klipper-backup/script.sh
git status --short
ls -la docs
```

Successful result:

- `git status --short` printed nothing.
- `docs/` still existed.
- All documentation files were still present.
- The backup pushed normally.

This means the Mainsail `update_git` button should be safe to use after GitHub-side changes have been pulled locally.

---

## If This Ever Happens Again

### Step 1 - Do not press the Mainsail backup button again yet

Stop and inspect from SSH first.

---

### Step 2 - Check the backup script staging behavior

```bash
grep -R "git add" ~/klipper-backup -n
```

Safe behavior should include:

```bash
git add -A printer_data/config/
```

Dangerous behavior:

```bash
git add .
```

If `git add .` is present, patch the script before running backups.

---

### Step 3 - Check that docs are protected from cleanup

```bash
grep -n "find \"\$backup_path\"" ~/klipper-backup/script.sh
```

Safe cleanup lines must include:

```bash
! -name 'docs'
```

---

### Step 4 - Check local Git status before pushing

```bash
cd ~/config_backup
git status --short
```

Danger signs:

```text
D docs/upgrades.md
D docs/configuration.md
D docs/future-ideas.md
D docs/project-history.md
D docs/punch-list.md
D docs/klipper-backup-docs-protection.md
```

If docs are staged or shown as deleted, run:

```bash
git restore .
git status --short
```

Do not run the backup script until docs deletions are gone.

---

### Step 5 - Pull docs if needed

If `docs/` is missing locally but exists on GitHub, use the Mainsail button:

```text
PULL_FROM_GIT
```

or SSH:

```bash
cd ~/config_backup
git pull --ff-only
ls -la docs
git status --short
```

---

### Step 6 - Run a manual backup test

```bash
cd ~/config_backup
git restore .
~/klipper-backup/script.sh
git status --short
ls -la docs
```

Only use the Mainsail `update_git` button after this test succeeds.

---

## Current Project Rule

```text
The Raspberry Pi owns printer_data/config/ backups.
GitHub/ChatGPT owns docs/ and README.md documentation edits.
```

The Pi must never stage documentation changes or documentation deletions.

Any future Klipper-Backup update or reinstall should be checked against this page before using `update_git` again.

When a page or file is added directly in GitHub, pull it down to the Pi first:

```text
Press PULL_FROM_GIT before pressing update_git.
```
