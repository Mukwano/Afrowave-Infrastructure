GitHub-Sync-Guide
========================
# Afrowave Infrastructure – GitHub Sync Guide
This document explains how to manage synchronization between the local Afrowave Infrastructure documentation and the Afrowave Community GitHub repository using two separate GitHub accounts.  

It allows seamless switching between:
- Personal account: **afrowaveltd**
- Community account: **mukwano**

This guide ensures future administrators can reproduce the setup reliably.

---

# 1. Purpose of This Guide
This guide provides:
- A clear setup for two GitHub accounts on the same machine.
- Secure SSH-based authentication for each account.
- Automatic switching between accounts using SSH config aliases.
- A simple synchronization workflow for updating the documentation repository.

This configuration ensures clean version control, team collaboration, and reproducibility.

---

# 2. SSH Keys Setup

## 2.1 Directory for SSH keys
On Windows, the SSH directory is:
```
C:\Users<username>.ssh\
```

## 2.2 Generate personal account SSH key
```
ssh-keygen -t ed25519 -C "afrowaveltd" -f "$env:USERPROFILE.ssh\id_ed25519_afrowave"
```

## 2.3 Generate community account SSH key
```
ssh-keygen -t ed25519 -C "mukwano" -f "$env:USERPROFILE.ssh\id_ed25519_mukwano"
```

Each command creates:
- `id_ed25519_xxxx` (private key)
- `id_ed25519_xxxx.pub` (public key)

---

# 3. Adding SSH Keys to GitHub

## 3.1 For personal account (afrowaveltd)
1. Login to https://github.com/settings/keys  
2. Click **New SSH key**
3. Title: *Afrowave Docs Key (Local PC)*
4. Paste:
```
cat ~/.ssh/id_ed25519_mukwano.pub
```


# 4. SSH Config – Automatic Account Switching

Open the SSH config file:
```
notepad $env:USERPROFILE.ssh\config
```

Insert this:
---------------------------------
GitHub personal account (afrowaveltd)
---------------------------------

Host github.com-afrowave
HostName github.com
User git
IdentityFile C:/Users/<username>/.ssh/id_ed25519_afrowave
IdentitiesOnly yes

---------------------------------
GitHub community account (mukwano)
---------------------------------

Host github.com-mukwano
HostName github.com
User git
IdentityFile C:/Users/<username>/.ssh/id_ed25519_mukwano
IdentitiesOnly yes


Replace `<username>` with your Windows username (e.g., *mukwano*).

---

# 5. Test Authentication

## Test personal account
```
ssh -T git@github.com-afrowave
```
Expected:
```
Hi afrowaveltd! You've successfully authenticated...
```

## Test community account
```
ssh -T git@github.com-mukwano
```
Expected:
```
Hi mukwano! You've successfully authenticated...
```

If both succeed → configuration is complete.

---

# 6. Cloning Repositories

## Personal repo clone
```
git clone git@github.com-afrowave:afrowaveltd/<repo>.git

```
## Community repo clone
```
git clone git@github.com-mukwano:mukwano/<repo>.git
```

Git will automatically use the correct SSH key.

---

# 7. Adding Git Remote for Documentation

Inside your local documentation folder:
```
cd <path-to-Afrowave-Documentation>
git init
git remote add origin git@github.com-mukwano:Afrowave-Community/Afrowave-Infrastructure.git
```

To verify:
```
git remote -v
```
# 8. Automatic Sync Script

Place this file in:
```
sync-docs.ps1
```

Content:
```
Set-Location "<path to Afrowave-Infrastructure>"

git add .
git commit -m "Documentation update on $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')"
git push origin main
```

Usage:
```
.\sync-docs.ps1
```

---

# 9. Troubleshooting GitHub Authentication

| Symptom | Cause | Fix |
|--------|--------|------|
| `Permission denied (publickey)` | Key not added to GitHub | Add public key |
| `Could not resolve hostname github.com-xxxx` | Wrong SSH config | Check Host name |
| GitHub Desktop uses wrong account | Cloned repo without alias | Re-clone using `github.com-mukwano:` |
| Host key mismatch | Old known_hosts entry | Delete entry from `~/.ssh/known_hosts` |

---

# 10. Cross-References
- [Main Documentation](README.md)  
- [DC1 Inventory](DC1-Inventory.md)  
- [Troubleshooting Index](Troubleshooting/Troubleshooting-Template.md)  
- [WG-LINK Network](Network-WG-Link.md)

---

# 11. Tags
```
#github #sync #ssh #infrastructure #documentation #automation
```