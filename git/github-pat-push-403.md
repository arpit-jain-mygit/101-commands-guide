# Fix GitHub push 403 when using a PAT

## Problem

While pushing a local repo to GitHub over HTTPS:

```bash
git push -u origin main
```

Git asked for a username and password:

```text
Username for 'https://github.com': sachin.arpit.learning@gmail.com
Password for 'https://sachin.arpit.learning%40gmail.com@github.com':
```

The push failed with:

```text
remote: Permission to arpit-jain-mygit/family-health-coach-ai.git denied to arpit-jain-mygit.
fatal: unable to access 'https://github.com/arpit-jain-mygit/family-health-coach-ai.git/': The requested URL returned error: 403
```

## Cause

GitHub no longer accepts account passwords for Git over HTTPS. The password field must use a Personal Access Token (PAT).

This error can happen when:

- The PAT does not have write access to the repository.
- The wrong username is entered.
- macOS Keychain has cached an old or incorrect GitHub credential.

## Working Solution

### 1. Create a classic PAT

Open:

```text
https://github.com/settings/tokens
```

Click:

```text
Generate new token -> Generate new token (classic)
```

Select this scope:

```text
repo
```

This is enough for pushing code to a normal GitHub repository.

Optional:

```text
workflow
```

Select `workflow` only if pushing or editing files under:

```text
.github/workflows/
```

### 2. Clear the old GitHub credential from macOS Keychain

Run:

```bash
git credential-osxkeychain erase
```

Paste this input:

```text
protocol=https
host=github.com
```

Then press Enter twice.

### 3. Push again

```bash
cd /path/to/your/repo
git push -u origin main
```

When prompted, use:

```text
Username: arpit-jain-mygit
Password: paste the PAT
```

Do not use the email address as the username.

## Fine-Grained PAT Alternative

For a fine-grained PAT, use:

```text
Repository access: Only select repositories
Repository: target repository
Contents: Read and write
Metadata: Read-only
```

Optional:

```text
Workflows: Read and write
```

Only select `Workflows: Read and write` if pushing files under `.github/workflows/`.

## Quick Checklist

- Remote URL points to the correct repository.
- GitHub username is used, not email.
- PAT has repository write access.
- Old GitHub credentials are cleared from Keychain.
- PAT is pasted in the password prompt.
