# Generate SSH Key for github

Generating an SSH key without a passphrase is straightforward, but remember that anyone with access to your computer will be able to access your GitHub account via SSH without an extra layer of security.

Since you want the shortest path using Ubuntu defaults:

### 1. Generate the Key

Run the following command. When prompted to "Enter passphrase," simply press **Enter** twice to leave it empty.

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"

```

* **Note:** When it asks where to save the key, press **Enter** to accept the default location (`/home/user/.ssh/id_ed25519`).

---

### 2. Copy the Public Key

You need to copy the contents of the public key file to your clipboard.

```bash
cat ~/.ssh/id_ed25519.pub

```

---

### 3. Add to GitHub

1. Go to **GitHub** → **Settings** (top right profile icon).
2. Click **SSH and GPG keys** in the left sidebar.
3. Click **New SSH key**.
4. Give it a **Title** (e.g., "Ubuntu Laptop") and paste your key into the **Key** field.
5. Click **Add SSH key**.

---

### 4. Verify Connection

Since you aren't using an SSH agent, `ssh` will look for the default file names automatically. Test it with:

```bash
ssh -T git@github.com

```

> If it asks if you want to continue connecting, type `yes`. You should see a message saying: *"Hi [username]! You've successfully authenticated..."*

---

# Extra SSH and SCP with customized ssh port

```
ssh -p [port_number] [user]@[remote_host]
```

```
scp -P [port_number] [source_file] [user]@[remote_host]:[destination_path]
```

--- @end