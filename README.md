# Web Development Fundamentals Class

# Secure GitHub Authentication for Clients (SSH Guide)

Using an SSH key is the industry standard for authentication. It’s significantly more secure than saving a token in your `.git/config` for several reasons:

1. **It's never transmitted:** When you use a token, your computer has to send that token over the internet to GitHub every time you push. With SSH, your computer uses complex math to "prove" it holds the private key without actually sending the key over the internet.
2. **Cannot be accidentally shared:** Your `.git/config` lives directly inside your project folder. If you ever zip up the project and send it to someone, you might accidentally send your token too. SSH keys live safely isolated in your system's `~/.ssh` folder.
3. **Passphrase Protection:** When you create an SSH key, you can lock it with a password. Even if a hacker stole the physical private key file from your hard drive, they still couldn't use it without knowing your passphrase!

---

## Cheat Sheet: Creating an SSH Key for a New Client

Follow these instructions whenever you land a new client and need to isolate their GitHub access.

### Step 1: Generate the Key
Open PowerShell and run the key generator. We use `ed25519` because it's the fastest and most secure modern encryption standard. 

Replace `client2_name` with the actual client's name:

```powershell
ssh-keygen -t ed25519 -C "client2_name" -f "$HOME\.ssh\client2_name_key"
```
*(It will ask you if you want to enter a passphrase. You can press Enter to leave it empty for convenience, or type a password for maximum security).*

### Step 2: Give the Key to GitHub
You need to copy the "Public" version of the key to your clipboard. You can do this by running:

```powershell
cat "$HOME\.ssh\client2_name_key.pub" | clip
```

Then:
1. Log into the new client's GitHub account.
2. Go to **Settings** -> **SSH and GPG keys**.
3. Click **New SSH key**.
4. Give it a descriptive name (like "Pedro Laptop"), paste the key from your clipboard into the key field, and save.

### Step 3: Tell Git to Use the Key
Whenever you clone a repository for this new client, ensure you grab the **SSH URL** (it starts with `git@github.com:` instead of `https://`).

After you clone the repository, open a terminal inside the project folder and run this command so Git knows to use the new key for this specific folder:

```powershell
git config core.sshCommand "ssh -i ~/.ssh/client2_name_key -F /dev/null"
```

That's it! You now have a perfectly isolated, highly secure connection for the new client.
