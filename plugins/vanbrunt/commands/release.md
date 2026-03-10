---
description: Run the master sync release workflow — creates a dated branch from staging reset to master, commits, pushes, and opens a GitLab MR automatically. Auto-increments branch suffix (n1, n2...) if branch already exists.
allowed-tools: Bash, AskUserQuestion
---

# vanbrunt: release

Run the master sync git workflow for the vanbrunt project. This syncs staging changes on top of master via a dated branch, then automatically creates a GitLab Merge Request.

## Step 0: Detect Operating System

Run OS detection to determine which workflow to follow:

```bash
uname -s 2>/dev/null; echo "OS_ENV=$OS"
```

- If `uname -s` returns `Darwin` → follow the **macOS Workflow** below
- If `uname -s` returns `MINGW*`, `MSYS*`, or `CYGWIN*`, or `OS_ENV` contains `Windows_NT` → follow the **Windows Workflow** below

---

## macOS Workflow

### Step 1: Ensure glab is authenticated

Check if `glab` is installed:

```bash
command -v glab
```

If not installed, install it:

```bash
brew install glab
```

Check if already authenticated to the self-hosted instance:

```bash
glab auth status --hostname gitlab.podium.com 2>&1
```

If the output shows authentication is valid (contains "Logged in to gitlab.podium.com"), skip to Step 2.

If the output contains `could not authenticate`, `No token found`, `has not been authenticated`, or any error, authenticate using a Personal Access Token:

1. Open the browser to the GitLab token creation page:

```bash
open "https://gitlab.podium.com/-/user_settings/personal_access_tokens?name=glab-cli&scopes=api,read_user,write_repository"
```

2. Output a message to the user (plain text, not AskUserQuestion) asking them to create the token with `api`, `read_user`, and `write_repository` scopes checked, then paste the token in their reply. Wait for their next message containing the token.

3. Once the user provides the token in their reply, authenticate glab using stdin — replace `<TOKEN>` with the exact value the user provided:

```bash
echo "<TOKEN>" | glab auth login --hostname gitlab.podium.com --stdin 2>&1
```

Note: this command may print a telemetry warning like `Could not send telemetry data: POST https://gitlab.com/api/v4/usage_data/track_event: 401` — this is harmless and expected; ignore it.

4. Verify authentication succeeded (look for "Logged in to gitlab.podium.com"):

```bash
glab auth status --hostname gitlab.podium.com 2>&1
```

If verification fails, stop and report the error. Do not proceed to Step 2.

### Step 2: Sync master and staging

Run these commands sequentially, stopping and reporting any error:

```bash
git checkout master
git pull
git checkout staging
git pull
```

### Step 3: Determine branch name

Get today's date in YYYYMMDD format:

```bash
date +%Y%m%d
```

Try branch name `master_sync_<DATE>_n1`. Check if it already exists locally or on remote:

```bash
git branch --list "master_sync_<DATE>_n1"
git ls-remote --heads origin "master_sync_<DATE>_n1"
```

If the branch already exists (locally or remotely), increment the suffix: try `_n2`, then `_n3`, etc., until you find one that does not exist. Use the first available name.

### Step 4: Create branch and reset to master

Create the branch from the current staging HEAD, then reset soft to master so staging-only changes are staged:

```bash
git checkout -b <BRANCH_NAME>
git reset --soft master
```

After `git reset --soft master`, the working tree reflects staging but the index (staged changes) contains everything staging has that master does not. Do not alter or cherry-pick files — commit everything as-is.

### Step 5: Commit

Commit all staged changes. Use `-n` to skip pre-commit hooks:

```bash
git commit -nm "Master Sync <DATE> <SUFFIX>"
```

Where `<SUFFIX>` matches the branch suffix (e.g., `n1`, `n2`).

Example: `Master Sync 20260303 n1`

### Step 6: Push

Push the new branch and set upstream tracking:

```bash
git push --set-upstream origin <BRANCH_NAME>
```

### Step 7: Create GitLab Merge Request

The git remote uses `gitlab-ssh.podium.com` (an SSH alias), but `glab` needs an HTTPS remote pointing to `gitlab.podium.com` to make API calls. Add a temporary HTTPS remote, create the MR, then remove it:

```bash
git remote add gitlab-https https://gitlab.podium.com/engineering/marketing/vanbrunt.git
```

```bash
glab mr create \
  --target-branch master \
  --title "Master Sync <DATE> <SUFFIX>" \
  --description "Automated master sync from staging to master. Resolve any conflicts keeping changes from staging." \
  --yes
```

```bash
git remote remove gitlab-https
```

Capture the MR URL from the `glab mr create` output.

### Step 8: Report

After successful MR creation, print:

```
✅ Branch pushed: <BRANCH_NAME>
✅ MR created: <MR_URL>
Commit: Master Sync <DATE> <SUFFIX>

Next steps:
  1. Resolve any remaining conflicts keeping changes from staging
  2. Merge the MR once approved
```

If any step fails, stop immediately, print the error output, and do not proceed to the next step.

---

## Windows Workflow

Runs inside Git Bash (MINGW/MSYS). Uses `powershell` for date formatting.

### Step W1: Ensure glab is authenticated

Check if `glab` is installed:

```bash
command -v glab
```

If not installed, install it via winget:

```bash
winget install glab
```

After installing, restart Git Bash and re-run `command -v glab` to confirm it is available.

Check if already authenticated to the self-hosted instance:

```bash
glab auth status --hostname gitlab.podium.com 2>&1
```

If the output shows authentication is valid (contains "Logged in to gitlab.podium.com"), skip to Step W2.

If the output contains `could not authenticate`, `No token found`, `has not been authenticated`, or any error, authenticate using a Personal Access Token:

1. Open the browser to the GitLab token creation page:

```bash
start "https://gitlab.podium.com/-/user_settings/personal_access_tokens?name=glab-cli&scopes=api,read_user,write_repository"
```

2. Output a message to the user (plain text, not AskUserQuestion) asking them to create the token with `api`, `read_user`, and `write_repository` scopes checked, then paste the token in their reply. Wait for their next message containing the token.

3. Once the user provides the token in their reply, authenticate glab using stdin — replace `<TOKEN>` with the exact value the user provided:

```bash
echo "<TOKEN>" | glab auth login --hostname gitlab.podium.com --stdin 2>&1
```

Note: this command may print a telemetry warning like `Could not send telemetry data: POST https://gitlab.com/api/v4/usage_data/track_event: 401` — this is harmless and expected; ignore it.

4. Verify authentication succeeded (look for "Logged in to gitlab.podium.com"):

```bash
glab auth status --hostname gitlab.podium.com 2>&1
```

If verification fails, stop and report the error. Do not proceed to Step W2.

### Step W2: Sync master and staging

Run these commands sequentially, stopping and reporting any error:

```bash
git checkout master
git pull
git checkout staging
git pull
```

### Step W3: Determine date and branch name

Get today's date via PowerShell. The `tr -d '\r'` is required to strip the Windows carriage return from PowerShell output, otherwise branch names will be corrupted:

```bash
DATE=$(powershell -Command "Get-Date -Format yyyyMMdd" | tr -d '\r')
echo "DATE=$DATE"
```

Verify the output looks like `DATE=20260310` (8 digits, no extra characters).

Try branch name `master_sync_${DATE}_n1`. Check if it already exists locally or on remote:

```bash
git branch --list "master_sync_${DATE}_n1"
git ls-remote --heads origin "master_sync_${DATE}_n1"
```

If the branch already exists (locally or remotely), increment the suffix: try `_n2`, then `_n3`, etc., until you find one that does not exist. Use the first available name as `<BRANCH_NAME>`.

### Step W4: Create branch and reset to master

Create the branch from the current staging HEAD, then reset soft to master so staging-only changes are staged:

```bash
git checkout -b master_sync_${DATE}_<SUFFIX>
git reset --soft master
```

Use the exact `<BRANCH_NAME>` with the correct suffix determined in Step W3.

After `git reset --soft master`, do not alter or cherry-pick files — commit everything as-is.

### Step W5: Commit

Commit all staged changes. Use `-n` to skip pre-commit hooks:

```bash
git commit -nm "Master Sync ${DATE} <SUFFIX>"
```

Where `<SUFFIX>` matches the branch suffix (e.g., `n1`, `n2`).

Example: `Master Sync 20260310 n1`

### Step W6: Push

Push the new branch and set upstream tracking:

```bash
git push --set-upstream origin <BRANCH_NAME>
```

### Step W7: Create GitLab Merge Request

The git remote uses `gitlab-ssh.podium.com` (an SSH alias), but `glab` needs an HTTPS remote pointing to `gitlab.podium.com` to make API calls. Add a temporary HTTPS remote, create the MR, then remove it:

```bash
git remote add gitlab-https https://gitlab.podium.com/engineering/marketing/vanbrunt.git
```

```bash
glab mr create \
  --target-branch master \
  --title "Master Sync ${DATE} <SUFFIX>" \
  --description "Automated master sync from staging to master. Resolve any conflicts keeping changes from staging." \
  --yes
```

```bash
git remote remove gitlab-https
```

Capture the MR URL from the `glab mr create` output.

### Step W8: Report

After successful MR creation, print:

```
✅ Branch pushed: <BRANCH_NAME>
✅ MR created: <MR_URL>
Commit: Master Sync <DATE> <SUFFIX>

Next steps:
  1. Resolve any remaining conflicts keeping changes from staging
  2. Merge the MR once approved
```

If any step fails, stop immediately, print the error output, and do not proceed to the next step.
