# Manually Install RemoteSSH VS Code Server

Starting around version 1.82, Microsoft shifted the default installation path for the VS Code Server. While the old path (`~/.vscode-server/bin/...`) sometimes still works as a fallback, the modern "CLI-based" architecture uses a more nested structure.


### The New Architecture Path (e.g. v1.108)

Instead of putting everything in `bin`, the server is now tucked under a `cli` subfolder:

**`~/.vscode-server/cli/servers/Stable-${COMMIT_ID}/server/`**

-----

### Updated Manual Installation Steps

The `code-<commit-id>` file is the "hidden" entry point that determines whether the Remote-SSH extension starts a download or uses what you’ve provided. If that file is missing from the root, the extension assumes the environment is empty, regardless of what is in your subfolders.

For **VS Code 1.108**, here is the **absolute, no-steps-missing, final guide** to a manual installation.

---

### Phase 1: The "Download List"
You need **two** distinct packages from a machine with internet. Replace `${COMMIT_ID}` with your 40-character hex string (e.g., `c3a2684...`).

1.  **The Server:** `https://update.code.visualstudio.com/commit:${COMMIT_ID}/server-linux-x64/stable`
2.  **The CLI:** `https://update.code.visualstudio.com/commit:${COMMIT_ID}/cli-linux-x64/stable`

---

### Phase 2: The "Silent" Setup (Step-by-Step)

#### 1. Create the Directory Tree
Log in to your Linux box and run:
```bash
COMMIT_ID="your_commit_id_here"
# Create the deep path for the server engine
mkdir -p ~/.vscode-server/cli/servers/Stable-${COMMIT_ID}/server
```

#### 2. Install the CLI (The Manager)
Extract the CLI tarball. It contains one file named `code`.
```bash
# Extract 'code' to the specific version folder
tar -xzf vscode-cli-linux-x64.tar.gz -C ~/.vscode-server/cli/servers/Stable-${COMMIT_ID}
```

#### 3.  The Entry Point
You must copy that `code` binary into the root of `.vscode-server` and rename it.
```bash
cp ~/.vscode-server/cli/servers/Stable-${COMMIT_ID}/code ~/.vscode-server/code-${COMMIT_ID}
```

#### 4. Install the Server (The Engine)
Extract the Server tarball into the `server` subfolder.
```bash
tar -xzf vscode-server-linux-x64.tar.gz -C ~/.vscode-server/cli/servers/Stable-${COMMIT_ID}/server --strip-components 1
```

#### 5. Set the "Done" Flags (The Proof)
You need **two** specific markers so VS Code doesn't touch the network:
```bash
# Flag 1: Tells the SSH extension the CLI is ready
touch ~/.vscode-server/vscode-cli-${COMMIT_ID}.tar.gz.done

# Flag 2: Tells the CLI that the Server is ready
touch ~/.vscode-server/cli/servers/Stable-${COMMIT_ID}/server/0
```

---

### Phase 3: The Final Verification
Your directory **must** look like this. If a single file is missing, the download starts.



```text
~/.vscode-server/
├── code-${COMMIT_ID}                      <-- The Entry Point (The one you found)
├── vscode-cli-${COMMIT_ID}.tar.gz.done    <-- Global Flag
└── cli
    └── servers
        └── Stable-${COMMIT_ID}/
            ├── code                       <-- The CLI binary
            └── server/                    
                ├── bin/
                ├── node
                └── 0                      <-- Internal Flag
```

### Phase 4: Disable "Exec Server" (Required for v1.108)
VS Code 1.108 often tries to use a new "Exec Server" connection method that can ignore manual files. 
1. Open local VS Code settings (`Ctrl + ,`).
2. Search for `Remote.SSH: Use Exec Server`.
3. **Uncheck** it.

---

### One-Shot Script
If you want to be 100% sure, run this (replace the ID first):
```bash
ID="c3a26841a84f20dfe0850d0a5a9bd01da4f003ea" # Use YOUR id
cd ~/.vscode-server
mkdir -p cli/servers/Stable-$ID/server
# Assuming your tarballs are in ~/packages
tar -xzf ~/packages/vscode-cli-linux-x64.tar.gz -C cli/servers/Stable-$ID
cp cli/servers/Stable-$ID/code code-$ID
tar -xzf ~/packages/vscode-server-linux-x64.tar.gz -C cli/servers/Stable-$ID/server --strip-components 1
touch vscode-cli-$ID.tar.gz.done
touch cli/servers/Stable-$ID/server/0
```

-----

### Why the change?

The "new" architecture was designed to unify how the standalone **VS Code CLI** (the `code` command for tunnels) and the **Remote-SSH** extension share server binaries. By moving them to `/cli/servers/`, VS Code can manage multiple versions more cleanly and share the same binary if you switch between an SSH connection and a Tunnel connection.


================================
-----

## Identify the commnit id in tar.gz

If you already have the `.tar.gz` file but aren't sure which version of VS Code it belongs to, you can find the Commit ID by peering inside the archive without needing to fully extract it first.

Here are the two most reliable ways to find that ID.

### Method 1: Check the `product.json` (Recommended)

Every VS Code distribution includes a `product.json` file that explicitly lists its build details. You can use a "one-liner" command to extract just that file to your terminal:

```bash
tar -xzf vscode-server-linux-x64.tar.gz --wildcards '*/product.json' -O | grep '"commit"'
```

  * **`tar -xzf`**: Extracts the file.
  * **`--wildcards '*/product.json'`**: Looks for the product metadata file.
  * **`-O`**: Redirects the file content to standard output (your screen) instead of saving it to disk.
  * **`grep '"commit"'`**: Filters the results to show you the 40-character hex string.

-----

### Method 2: Inspect the Package Version File

If you have already partially extracted the files, or if the method above feels too cluttered, look for a file simply named `version`.

If you are still looking at the compressed archive, run:

```bash
tar -ztvf vscode-server-linux-x64.tar.gz | head -n 20
```

Often, the top-level directory inside the tarball is named after the commit ID, or a file named `version` or `commit` will be visible in the file list.

-----

### How to Verify if it Matches your Local VS Code

Once you have that string, you must verify it matches the VS Code application you are running on your desktop.

1.  Open VS Code on your **local machine**.
2.  Go to **Help** \> **About** (on Windows/Linux) or **Code** \> **About Visual Studio Code** (on macOS).
3.  Compare the **Commit** line in the popup with the string you found in the tarball.

> **Why this matters:** If the IDs do not match exactly, VS Code will ignore your manual installation and attempt to download its own version as soon as you try to connect.

========================
-----

## remote.SSH.useExecServer

The "Exec Server" is the biggest architectural shift in VS Code Remote development in recent years. If you are manually installing things, it is your #1 enemy because it is designed to be **autonomous and dynamic**, which often means it ignores your carefully placed manual files in favor of its own automated logic.

### 1. What is the "Exec Server"?
In the **Legacy Architecture**, VS Code used a series of shell scripts (like `server.sh`) to probe your Linux box and start the server. 

In the **New "Exec Server" Architecture (v1.108)**, VS Code connects to your remote machine and immediately tries to launch a small, high-performance binary called the **VS Code CLI** (the `code-c3a268...` file you found). This CLI then acts as a "mini-OS" on your server. It handles:
* **Process management:** Starting and watching the main Node.js server.
* **Port forwarding:** Managing the tunnels between your laptop and the server.
* **Dynamic fetching:** If it thinks a file is missing or corrupted, it will ignore your local directory and try to download a fresh copy from Microsoft’s CDN.

### 2. Why it breaks Manual Installations
The Exec Server is designed to be "self-healing." If the setting `Remote.SSH: Use Exec Server` is **On** (which is the default in newer versions), the extension follows this logic:
1.  Connect to SSH.
2.  Check for the CLI binary in the root.
3.  If anything looks "unofficial" or if a flag is missing, **it bypasses your `~/.vscode-server` folder entirely** and creates a temporary one in `/tmp` or tries to re-download.

By turning it **Off**, you force VS Code to fall back to the **Legacy Script-based bootstrap**. The legacy script is much more "obedient"—it checks for the `${COMMIT_ID}/0` flag and, if it finds it, stops immediately and runs whatever is in that folder without asking questions.

### 3. Key Differences at a Glance

| Feature | Legacy (Exec Server OFF) | Modern (Exec Server ON) |
| :--- | :--- | :--- |
| **Primary Tool** | Shell Scripts (`bash`/`sh`) | Compiled Binary (`vscode-cli`) |
| **Install Path** | `~/.vscode-server/bin/[ID]` | `~/.vscode-server/cli/servers/Stable-[ID]` |
| **Trust Level** | Trusts your manual flags (`0`) | Verifies binaries; re-downloads if unsure |
| **Main Advantage** | Extremely predictable for offline use | Faster connection; better for Dev Containers |

---

### How to toggle it for your setup
Since you are manually managing your server files, you should almost certainly keep this **OFF** to prevent VS Code from "outsmarting" your manual configuration.

1.  Press `Ctrl + ,` (Comma) in VS Code.
2.  Search for **`remote.SSH.useExecServer`**.
3.  Set it to **`false`** (Uncheck the box).

---

### Summary for your 1.108 setup:
If you want to use the files you manually installed:
* **With Exec Server ON:** You must have the `code-<ID>` entry point, the `cli/servers/...` path, and the `.done` flag in the root.
* **With Exec Server OFF:** You only need the `bin/<ID>` path and the `0` flag. 

=======
-------



