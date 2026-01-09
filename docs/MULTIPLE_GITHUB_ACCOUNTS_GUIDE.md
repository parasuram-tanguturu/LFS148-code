# Complete Guide: Managing Multiple GitHub Accounts on macOS

> **A beginner-friendly, step-by-step guide to managing multiple GitHub accounts on a single Mac using SSH keys and Git configuration.**

## Table of Contents

1. [Why This Guide?](#why-this-guide)
2. [Understanding the Problem](#understanding-the-problem)
3. [Our Solution Overview](#our-solution-overview)
4. [What We Set Up](#what-we-set-up)
5. [Step-by-Step Setup (Already Done)](#step-by-step-setup-already-done)
6. [How It Works](#how-it-works)
7. [Daily Usage Guide](#daily-usage-guide)
8. [Troubleshooting](#troubleshooting)
9. [Reference: All Files Created](#reference-all-files-created)

---

## Why This Guide?

If you have multiple GitHub accounts (e.g., personal and work), you've probably encountered this error:

```
remote: Permission denied to account1
fatal: unable to access 'https://github.com/account2/repo.git/': The requested URL returned error: 403
```

This happens because Git tries to use the wrong credentials. This guide shows you how to set up your Mac so each repository automatically uses the correct GitHub account.

---

## Understanding the Problem

### The Challenge

When you have multiple GitHub accounts:
- **Account 1**: `parasuram-tanguturu` (personal)
- **Account 2**: `sai-annamacharya` (work/other)

Git doesn't know which account to use for which repository. By default, it uses the same credentials everywhere, causing authentication failures.

### Why HTTPS is Problematic

With HTTPS:
- You need Personal Access Tokens (passwords don't work with 2FA)
- Credentials are stored globally in macOS Keychain
- Git can't easily distinguish between accounts
- You get constant authentication prompts

### Why SSH is Better

With SSH:
- **No passwords needed** - uses cryptographic keys
- **Each account has its own key** - perfect separation
- **No prompts** - works automatically once set up
- **More secure** - keys are harder to compromise than passwords

---

## Our Solution Overview

We set up a system that:

1. **Creates separate SSH keys** for each GitHub account
2. **Configures SSH** to use the right key for each account
3. **Sets up Git configs** for each account
4. **Uses conditional includes** to auto-select the right config based on directory

### Visual Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Your Mac                              │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Repository: ~/Coding/Observability/LFS148-code │  │
│  │  → Uses: parasuram-tanguturu account              │  │
│  │  → SSH Key: id_ed25519_parasuram                  │  │
│  │  → Git Config: .gitconfig-parasuram               │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Repository: ~/OtherProjects/WorkRepo             │  │
│  │  → Uses: sai-annamacharya account                │  │
│  │  → SSH Key: id_ed25519_sai                        │  │
│  │  → Git Config: .gitconfig-ram                     │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## What We Set Up

### Accounts Configured

| Account | Email | SSH Host Alias | SSH Key File |
|---------|-------|----------------|--------------|
| **parasuram-tanguturu** | lnparasuram@gmail.com | `github.com-parasuram` | `~/.ssh/id_ed25519_parasuram` |
| **sai-annamacharya** | ysai0603@outlook.com | `github.com-sai` | `~/.ssh/id_ed25519_sai` |

### Files Created

1. **SSH Keys** (in `~/.ssh/`):
   - `id_ed25519_parasuram` (private key)
   - `id_ed25519_parasuram.pub` (public key)
   - `id_ed25519_sai` (private key)
   - `id_ed25519_sai.pub` (public key)
   - `config` (SSH configuration)

2. **Git Config Files** (in `~`):
   - `.gitconfig` (main config with conditional includes)
   - `.gitconfig-parasuram` (config for parasuram-tanguturu)
   - `.gitconfig-ram` (config for sai-annamacharya)

---

## Step-by-Step Setup (Already Done)

Here's what we did (for reference):

### Step 1: Generate SSH Keys

```bash
# Generate key for parasuram-tanguturu account
ssh-keygen -t ed25519 -C "lnparasuram@gmail.com" -f ~/.ssh/id_ed25519_parasuram

# Generate key for sai-annamacharya account
ssh-keygen -t ed25519 -C "ysai0603@outlook.com" -f ~/.ssh/id_ed25519_sai
```

**What this does:**
- Creates cryptographic key pairs (public + private)
- `ed25519` is a modern, secure algorithm
- The email is just a label (doesn't need to match GitHub email)

### Step 2: Create SSH Config

Created `~/.ssh/config`:

```
# GitHub account: parasuram-tanguturu
Host github.com-parasuram
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_parasuram
    IdentitiesOnly yes

# GitHub account: sai-annamacharya
Host github.com-sai
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_sai
    IdentitiesOnly yes
```

**What this does:**
- Creates aliases (`github.com-parasuram`, `github.com-sai`)
- Maps each alias to the correct SSH key
- `IdentitiesOnly yes` prevents Git from trying other keys

### Step 3: Add Public Keys to GitHub

**For parasuram-tanguturu:**
1. Copied public key: `cat ~/.ssh/id_ed25519_parasuram.pub | pbcopy`
2. Went to: https://github.com/settings/keys
3. Clicked "New SSH key"
4. Pasted key and saved

**For sai-annamacharya:**
1. Copied public key: `cat ~/.ssh/id_ed25519_sai.pub | pbcopy`
2. Logged into sai-annamacharya account
3. Went to: https://github.com/settings/keys
4. Clicked "New SSH key"
5. Pasted key and saved

**Important:** Public keys are safe to share. Private keys (`id_ed25519_*` without `.pub`) should **NEVER** be shared.

### Step 4: Create Separate Git Configs

**Created `~/.gitconfig-parasuram`:**
```ini
[user]
    name = parasuram-tanguturu
    email = lnparasuram@gmail.com
[credential]
    helper = osxkeychain
    useHttpPath = true
```

**Created `~/.gitconfig-ram`:**
```ini
[user]
    name = sai
    email = ysai0603@outlook.com
[credential]
    helper = osxkeychain
    useHttpPath = true
```

### Step 5: Set Up Conditional Includes

Updated `~/.gitconfig`:
```ini
[user]
	name = ram
	email = ysai0603@outlook.com
# ... other global settings ...

# Conditional includes for multiple GitHub accounts
# Use parasuram-tanguturu config for Observability projects
[includeIf "gitdir:~/Coding/Observability/"]
	path = ~/.gitconfig-parasuram
```

**What this does:**
- Any repo in `~/Coding/Observability/` automatically uses parasuram-tanguturu config
- Other repos use the default (sai/ram) config

### Step 6: Update Repository Remote URLs

Changed from HTTPS to SSH:
```bash
# Old (HTTPS)
https://github.com/parasuram-tanguturu/LFS148-code.git

# New (SSH with alias)
git@github.com-parasuram:parasuram-tanguturu/LFS148-code.git
```

---

## How It Works

### The Magic: SSH Host Aliases

When you use `git@github.com-parasuram:...`, SSH looks up `github.com-parasuram` in `~/.ssh/config` and finds:
- Use key: `~/.ssh/id_ed25519_parasuram`
- Connect to: `github.com` (the real server)

GitHub sees the public key and matches it to your account.

### The Flow

```
1. You run: git push origin main
   ↓
2. Git sees remote: git@github.com-parasuram:...
   ↓
3. SSH looks up "github.com-parasuram" in ~/.ssh/config
   ↓
4. SSH uses key: ~/.ssh/id_ed25519_parasuram
   ↓
5. GitHub receives public key
   ↓
6. GitHub matches key to parasuram-tanguturu account
   ↓
7. Push succeeds! ✅
```

### Conditional Git Configs

When you're in `~/Coding/Observability/LFS148-code/`:
- Git reads `~/.gitconfig`
- Sees `[includeIf "gitdir:~/Coding/Observability/"]`
- Checks: "Am I in that directory?" → Yes!
- Loads `~/.gitconfig-parasuram`
- Uses `name = parasuram-tanguturu` and `email = lnparasuram@gmail.com`

---

## Daily Usage Guide

### Cloning a New Repository

**For parasuram-tanguturu account:**
```bash
# Clone using SSH alias
git clone git@github.com-parasuram:parasuram-tanguturu/REPO_NAME.git

# Or if you have the HTTPS URL, convert it:
# Original: https://github.com/parasuram-tanguturu/REPO_NAME.git
# Convert to: git@github.com-parasuram:parasuram-tanguturu/REPO_NAME.git
```

**For sai-annamacharya account:**
```bash
git clone git@github.com-sai:sai-annamacharya/REPO_NAME.git
```

### Changing Remote URL of Existing Repository

**Check current remote:**
```bash
git remote -v
```

**Change to parasuram-tanguturu:**
```bash
git remote set-url origin git@github.com-parasuram:parasuram-tanguturu/REPO_NAME.git
```

**Change to sai-annamacharya:**
```bash
git remote set-url origin git@github.com-sai:sai-annamacharya/REPO_NAME.git
```

### Verifying Which Account You're Using

**Check Git user config:**
```bash
git config user.name
git config user.email
```

**Check which config file is being used:**
```bash
git config --show-origin user.name
git config --show-origin user.email
```

**Test SSH connection:**
```bash
# Test parasuram-tanguturu
ssh -T git@github.com-parasuram

# Test sai-annamacharya
ssh -T git@github.com-sai
```

Both should show: `Hi [username]! You've successfully authenticated...`

### Adding a New Repository

**Scenario:** You want to create a new repo under parasuram-tanguturu

1. Create repo on GitHub (via web interface)
2. Clone it:
   ```bash
   git clone git@github.com-parasuram:parasuram-tanguturu/NEW_REPO.git
   ```
3. Verify config:
   ```bash
   cd NEW_REPO
   git config user.name  # Should show: parasuram-tanguturu
   ```

### Working with Different Directories

**Automatic config selection:**
- Repos in `~/Coding/Observability/` → Uses parasuram-tanguturu config
- Repos elsewhere → Uses default (sai/ram) config

**To add more directories for parasuram-tanguturu:**
Edit `~/.gitconfig`:
```ini
[includeIf "gitdir:~/Coding/Observability/"]
	path = ~/.gitconfig-parasuram

[includeIf "gitdir:~/PersonalProjects/"]
	path = ~/.gitconfig-parasuram
```

---

## Troubleshooting

### Problem: "Permission denied (publickey)"

**Solution:**
1. Check if key is added to GitHub:
   ```bash
   ssh -T git@github.com-parasuram
   ```
2. If it fails, verify the public key is on GitHub:
   - Go to: https://github.com/settings/keys
   - Check if your key is listed
3. Verify SSH config:
   ```bash
   cat ~/.ssh/config
   ```
4. Check key permissions:
   ```bash
   ls -la ~/.ssh/id_ed25519_parasuram
   # Should show: -rw------- (600)
   ```

### Problem: "Wrong account being used"

**Solution:**
1. Check current remote:
   ```bash
   git remote -v
   ```
2. Verify it uses the correct SSH alias:
   - Should be: `git@github.com-parasuram:...` or `git@github.com-sai:...`
   - NOT: `git@github.com:...` (this uses default key)
3. Update remote if needed:
   ```bash
   git remote set-url origin git@github.com-parasuram:USERNAME/REPO.git
   ```

### Problem: "Wrong Git user name/email in commits"

**Solution:**
1. Check which config is active:
   ```bash
   git config --show-origin user.name
   ```
2. If wrong, set repo-specific config:
   ```bash
   git config user.name "parasuram-tanguturu"
   git config user.email "lnparasuram@gmail.com"
   ```
3. Or add directory to conditional includes in `~/.gitconfig`

### Problem: "SSH connection works but push fails"

**Possible causes:**
1. **Repository doesn't exist** - Create it on GitHub first
2. **No write access** - Check repository permissions
3. **Branch protection** - Some branches may be protected
4. **Wrong remote URL** - Verify with `git remote -v`

### Problem: "Key already in use" on GitHub

**Solution:**
- Each SSH key can only be added to one GitHub account
- If you need the same key on multiple accounts, you need multiple keys (which we have!)

### Problem: "Can't find SSH config"

**Solution:**
```bash
# Create if missing
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Create config file
touch ~/.ssh/config
chmod 600 ~/.ssh/config
```

---

## Reference: All Files Created

### SSH Files (`~/.ssh/`)

| File | Purpose | Permissions |
|------|---------|-------------|
| `id_ed25519_parasuram` | Private key for parasuram-tanguturu | `600` (rw-------) |
| `id_ed25519_parasuram.pub` | Public key for parasuram-tanguturu | `644` (rw-r--r--) |
| `id_ed25519_sai` | Private key for sai-annamacharya | `600` (rw-------) |
| `id_ed25519_sai.pub` | Public key for sai-annamacharya | `644` (rw-r--r--) |
| `config` | SSH host aliases configuration | `600` (rw-------) |

### Git Config Files (`~`)

| File | Purpose |
|------|---------|
| `.gitconfig` | Main Git config with conditional includes |
| `.gitconfig-parasuram` | Config for parasuram-tanguturu account |
| `.gitconfig-ram` | Config for sai-annamacharya account |

### File Contents Reference

**`~/.ssh/config`:**
```
# GitHub account: parasuram-tanguturu
Host github.com-parasuram
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_parasuram
    IdentitiesOnly yes

# GitHub account: sai-annamacharya
Host github.com-sai
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_sai
    IdentitiesOnly yes
```

**`~/.gitconfig-parasuram`:**
```ini
[user]
    name = parasuram-tanguturu
    email = lnparasuram@gmail.com
[credential]
    helper = osxkeychain
    useHttpPath = true
```

**`~/.gitconfig-ram`:**
```ini
[user]
    name = sai
    email = ysai0603@outlook.com
[credential]
    helper = osxkeychain
    useHttpPath = true
```

---

## Quick Reference Commands

### Check Current Setup
```bash
# Check SSH keys
ls -la ~/.ssh/id_ed25519*

# Check SSH config
cat ~/.ssh/config

# Check Git configs
cat ~/.gitconfig
cat ~/.gitconfig-parasuram
cat ~/.gitconfig-ram

# Test SSH connections
ssh -T git@github.com-parasuram
ssh -T git@github.com-sai
```

### Copy Public Keys
```bash
# Copy parasuram key
cat ~/.ssh/id_ed25519_parasuram.pub | pbcopy

# Copy sai key
cat ~/.ssh/id_ed25519_sai.pub | pbcopy
```

### Update Remote URLs
```bash
# For parasuram-tanguturu repo
git remote set-url origin git@github.com-parasuram:parasuram-tanguturu/REPO.git

# For sai-annamacharya repo
git remote set-url origin git@github.com-sai:sai-annamacharya/REPO.git
```

### Check Which Account is Active
```bash
# In a repository
git config user.name
git config user.email
git remote -v
```

---

## Security Best Practices

1. **Never share private keys** (`id_ed25519_*` without `.pub`)
2. **Keep private keys secure** - permissions should be `600` (owner read/write only)
3. **Use strong passphrases** (optional but recommended when generating keys)
4. **Rotate keys periodically** - Generate new keys every 1-2 years
5. **Remove old keys** from GitHub when no longer needed

---

## Adding More Accounts

To add a third GitHub account:

1. **Generate SSH key:**
   ```bash
   ssh-keygen -t ed25519 -C "email@example.com" -f ~/.ssh/id_ed25519_account3
   ```

2. **Add to SSH config:**
   ```bash
   # Add to ~/.ssh/config
   Host github.com-account3
       HostName github.com
       User git
       IdentityFile ~/.ssh/id_ed25519_account3
       IdentitiesOnly yes
   ```

3. **Add public key to GitHub:**
   ```bash
   cat ~/.ssh/id_ed25519_account3.pub | pbcopy
   # Then add at: https://github.com/settings/keys
   ```

4. **Create Git config:**
   ```bash
   cat > ~/.gitconfig-account3 << EOF
   [user]
       name = account3-name
       email = email@example.com
   EOF
   ```

5. **Add conditional include** (if needed):
   ```ini
   # In ~/.gitconfig
   [includeIf "gitdir:~/Account3Projects/"]
       path = ~/.gitconfig-account3
   ```

---

## Summary

You now have a complete system for managing multiple GitHub accounts:

✅ **Separate SSH keys** for each account  
✅ **SSH host aliases** for easy switching  
✅ **Separate Git configs** for each account  
✅ **Automatic config selection** based on directory  
✅ **No password prompts** - everything works automatically  

**Remember:**
- Use `git@github.com-parasuram:...` for parasuram-tanguturu repos
- Use `git@github.com-sai:...` for sai-annamacharya repos
- Repos in `~/Coding/Observability/` automatically use parasuram-tanguturu config

Happy coding! 🚀
