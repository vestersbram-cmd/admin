# Browser-Based Authentication Key Procedures

## Overview

This document describes how to obtain authentication credentials (SSH keys, OAuth tokens, API keys) via browser automation for GitHub and Gmail.

---

## GitHub Authentication

### Option A: SSH Keys (Recommended for git operations)

#### When to Use
- Terminal git operations (clone, push, pull)
- CI/CD pipelines that need git access
- Automated scripts that interact with GitHub

#### Prerequisites
- Active GitHub session in Chrome (logged in as target user)
- Chrome DevTools MCP server running on port 9111
- `ssh-keygen` available in terminal

#### Procedure

**Step 1: Check Existing Keys**
```bash
ls -la ~/.ssh/
cat ~/.ssh/id_ed25519.pub 2>/dev/null || cat ~/.ssh/id_rsa.pub 2>/dev/null || echo "No SSH key found"
```

**Step 2: Generate New SSH Key**
```bash
mkdir -p ~/.ssh
ssh-keygen -t ed25519 -C "your-email@github" -f ~/.ssh/id_ed25519 -N ""
```
⚠️ Use empty passphrase (`-N ""`) for non-interactive automation.

**Step 3: Add Key to GitHub via Browser**
Using browser-use agent:
1. Navigate to: `https://github.com/settings/keys`
2. Click "New SSH key"
3. Fill form:
   - **Title**: "Terminal Key"
   - **Key type**: "Authentication key"
   - **Key**: Paste public key from Step 2
4. Click "Add SSH key"

**Step 4: Email Verification** (Manual - Required)
GitHub sends verification email to your registered address.
1. Check your email inbox
2. Click the verification link (time-sensitive)
3. Key is now active

**Step 5: Configure Git for SSH**
```bash
git config --global url."git@github.com:".insteadOf "https://github.com/"
```

**Step 6: Test Connection**
```bash
git clone git@github.com:your-username/any-public-repo.git /tmp/test-clone
```

---

### Option B: GitHub CLI (Fully Automatable)

If you prefer fully automated setup without browser verification:

```bash
# Install gh
brew install gh  # macOS
# or: sudo apt install gh  # Linux

# Authenticate (non-interactive)
echo "your-token" | gh auth login --with-token
gh auth setup-git
```

---

### Automation Matrix

| Step | SSH+Browser | gh CLI | Notes |
|------|-------------|--------|-------|
| Generate key | ✅ | ✅ | Both CLI |
| Add to GitHub | ⚠️ Manual verify | ✅ | Browser needs email verify |
| Configure git | ✅ | ✅ | Both CLI |
| Test | ✅ | ✅ | Both CLI |

---

## Gmail Authentication

### Status: NOT YET CONFIGURED ⚠️

The Gmail OAuth setup requires additional steps that were not completed in this session.

#### To Configure (Future Work)

**Option A: OAuth2 via Browser**
1. Go to: `https://console.cloud.google.com/apis/credentials`
2. Create OAuth client ID (Desktop app type)
3. Download JSON, extract client_id/client_secret
4. Browser-use navigates to consent page
5. User authorizes manually
6. Exchange code for tokens

**Option B: Browser Session Cookies**
For gmailz/gmailz skills, browser-use can provide session cookies from existing Chrome session.

---

## Non-Interactive Settings

### SSH
```bash
export GIT_SSH_COMMAND="ssh -o StrictHostKeyChecking=no -o BatchMode=yes"
```

Or add to `~/.ssh/config`:
```
Host github.com
    StrictHostKeyChecking no
    BatchMode yes
```

### Git
```bash
git config --global init.defaultBranch main
git config --global pull.rebase false
git config --global fetch.prune true
```

---

## Quick Reference: browser-use Commands

```
# Check GitHub login status
Navigate to: https://github.com

# Add SSH key
Navigate to: https://github.com/settings/ssh/new
Fill form and submit

# Verify key added
Navigate to: https://github.com/settings/keys
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Permission denied" | Check key at github.com/settings/keys; run `ssh -T git@github.com` |
| "Host key verification failed" | `ssh-keyscan github.com >> ~/.ssh/known_hosts` |
| Interactive prompt | Use `GIT_SSH_COMMAND="ssh -o BatchMode=yes"` |
| Email not received | Check spam folder; GitHub sends from noreply@github.com |
| Key expired/revoked | Regenerate key and add to GitHub again |

---

## Complete Automation Script

```bash
#!/bin/bash
set -e

# GitHub SSH Setup Function
setup_github_ssh() {
    local email="${1:-tom260513@github}"
    local key_path="~/.ssh/id_ed25519"
    
    # Generate key if not exists
    if [ ! -f "$key_path" ]; then
        mkdir -p ~/.ssh
        ssh-keygen -t ed25519 -C "$email" -f "$key_path" -N ""
    fi
    
    echo "Public key:"
    cat "${key_path}.pub"
}

# GitHub API key addition (requires token)
add_github_key_api() {
    local token="$1"
    local title="$2"
    local pub_key="$3"
    
    curl -s -X POST \
        -H "Authorization: token $token" \
        -d "{\"title\":\"$title\",\"key\":\"$pub_key\"}" \
        https://api.github.com/user/keys
}

# Configure git for non-interactive SSH
git config --global url."git@github.com:".insteadOf "https://github.com/"
export GIT_SSH_COMMAND="ssh -o StrictHostKeyChecking=no -o BatchMode=yes"

# Usage example:
# setup_github_ssh "my-email@example.com"
# add_github_key_api "ghp_xxx" "My Key" "$(cat ~/.ssh/id_ed25519.pub)"
```

---

## Execution Log

| Date | Service | Action | Result |
|------|---------|--------|--------|
| 2025-05-22 | GitHub | Generate SSH key | ✅ Success |
| 2025-05-22 | GitHub | Add key via browser | ✅ Success (email verify) |
| 2025-05-22 | GitHub | Configure git SSH | ✅ Success |
| 2025-05-22 | GitHub | Test clone | ✅ Success |
| 2025-05-22 | Gmail | Configure OAuth | ⚠️ Not done |

---

## Files Generated

| File | Purpose | Location |
|------|---------|----------|
| `id_ed25519` | Private key (secret) | `~/.ssh/` |
| `id_ed25519.pub` | Public key (share) | `~/.ssh/` |
| `known_hosts` | GitHub host key | `~/.ssh/` |
