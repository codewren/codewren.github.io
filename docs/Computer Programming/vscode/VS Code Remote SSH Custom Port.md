# VS Code Remote SSH Custom Port

Connecting to a remote server via SSH on a non-standard port is a classic move for extra security. Instead of typing out the port number every single time, you can bake it right into your SSH config file.

Here is how to set that up so VS Code handles it seamlessly.

### 1. Locate or Create Your Config File

On most systems, your configuration file is located at:

* **Windows:** `C:\Users\YourUsername\.ssh\config`
* **macOS / Linux:** `~/.ssh/config`

### 2. Add the Configuration Entry

Open that file in a text editor (or VS Code itself) and add a block following this structure. The key directive here is `Port`.

```bash
Host my-remote-server
    HostName 123.456.78.90
    User your-username
    Port 2222

```

---

### Configuration Breakdown

| Parameter | Description |
| --- | --- |
| **`Host`** | A nickname of your choice. This is what you'll select in VS Code. |
| **`HostName`** | The actual IP address or domain of the remote server. |
| **`User`** | Your login username on the remote machine. |
| **`Port`** | **The magic ingredient.** Replace `2222` with your custom port. |
| **`IdentityFile`** | (Optional) The path to your private SSH key if you aren't using passwords. |

---

### 3. Connect via VS Code

Once you've saved the file, follow these steps in VS Code:

1. Open the **Command Palette** (`Ctrl+Shift+P` or `Cmd+Shift+P`).
2. Type **"Remote-SSH: Connect to Host..."** and select it.
3. You should see `my-remote-server` (or whatever nickname you chose) in the list.
4. Click it, and VS Code will automatically use the custom port defined in your config.

> **Pro Tip:** Ensure your remote server's firewall (like `ufw` or `iptables`) is actually open on that custom port, otherwise, you'll get a "Connection Timed Out" error no matter how perfect your config file looks!

---

# Enable Password-less login


Here is the quickest way to get that set up.

### 1. Generate your SSH Key (if you haven't)

Open your local terminal (PowerShell on Windows, or Terminal on macOS/Linux) and run:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"

```

* **Press Enter** to save it in the default location.
* **Passphrase:** You can leave this blank for maximum convenience, or enter one for extra security (VS Code can remember it for you).

### 2. Copy the Key to your Server

Now you need to tell your server that your local machine is "trusted." Since you are using a **custom port**, you must specify it in the command.

**For macOS and Linux:**
Replace `2222` with your port and use the nickname from your config file:

```bash
ssh-copy-id -p 2222 your-username@123.456.78.90

```

**For Windows (PowerShell):**
Windows doesn't have `ssh-copy-id` by default, so you can use this "one-liner" to get the job done:

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh your-username@123.456.78.90 -p 2222 "cat >> .ssh/authorized_keys"

```

---

### 3. Verify the Config

Ensure your `.ssh/config` file points to the private key you just created. It should look like this:

```bash
Host my-remote-server
    HostName 123.456.78.90
    User your-username
    Port 2222
    IdentityFile ~/.ssh/id_ed25519  # Point this to your private key

```

### How the Handshake Works

Once this is set up, the "lock" (public key) lives on your server, and the "key" (private key) lives on your laptop.

---

### 4. Final Step: Connect

1. Go back to **VS Code**.
2. Click the **Remote Window** icon (the small green/blue button in the bottom-left corner).
3. Select **Connect to Host** and pick your server.
4. It should now log you in automatically without asking for a password!

---@end