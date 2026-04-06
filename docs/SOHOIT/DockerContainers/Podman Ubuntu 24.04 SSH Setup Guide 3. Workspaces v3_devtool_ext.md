# Podman Ubuntu 24.04 SSH Setup Guide 3. Workspaces v3_devtool_ext

## Externalize workspace to reduce container size

The develpers container is easily ballooning to very large size, e.g. 50G is because that toolchains like Rust, Scala, and Java are "greedy"—they download compiler toolchains, libraries, and metadata into your home directory (e.g., `~/.cargo`, `~/.m2`, `~/.rustup`). When you `commit` container to image, all that "ephemeral" cache is frozen into a permanent image layer.

For a long-term professional setup, you should treat the **container as a disposable engine** and the **toolchain artifacts as persistent fuel**.

---

### 1. Externalize the "Greedy" Directories (Volumes)
Instead of letting these tools write into the container's writable layer, mount specific host directories or named volumes to the locations where these tools store their artifacts.

| Toolchain | Default Cache Path | Why it's big |
| :--- | :--- | :--- |
| **Rust** | `~/.cargo` & `~/.rustup` | Toolchains, downloaded crates, and build artifacts. |
| **Java/Maven**| `~/.m2/repository` | Every `.jar` dependency for every project. |
| **Scala/SBT** | `~/.sbt` & `~/.cache/coursier` | Compiler bridges and library dependencies. |
| **VS Code** | `~/.vscode-server` | Extensions and server-side binaries for remote dev. |
| **Python/uv** | `~/.cache/uv` | Cached wheels and environment metadata. |



### 2. Setup Script: Create Local Cache Dirs

This script ensures the directories exist on your host machine before you fire up the container. It also handles the creation of the `$CNTNER_HOME` variable if it isn't already set.

```bash
#!/bin/bash

# 1. Define the base directory if not already set
export CNTNER_HOME="${CNTNER_HOME:-$HOME/dev_container_cache}"

echo "Creating cache directories in: $CNTNER_HOME"

# 2. List of subdirectories to create
directories=(
    "cargo_home"        
    "rustup_home"     
    "maven_cache"
    "sbt_cache"
    "coursier_cache"
    "uv_cache"
    "vscode_server"
    "devspace"
)

# 3. Create directories and set permissions
for dir in "${directories[@]}"; do
    if [ ! -d "$CNTNER_HOME/$dir" ]; then
        echo "Creating $dir..."
        mkdir -p "$CNTNER_HOME/$dir"
    else
        echo "$dir already exists, skipping."
    fi
done

echo "Done! All directories are ready."
ls -l "$CNTNER_HOME"
```

### 3. The  Podman Command to create containers


```bash
export CNTNER_HOME=real_container_home
podman run -d -it \
  --name labWSL  --hostname labWSL --add-host labWSL:127.0.0.1  --network host \
  --user devuser \
  --uidmap 1001:0:1 \
  --uidmap 0:1:1001 \
  --uidmap 1002:1002:64534 \
  --gidmap 1001:0:1 \
  --gidmap 0:1:1001 \
 --gidmap 1002:1002:64534 \
  --cap-add=SYS_PTRACE \
  --security-opt seccomp=unconfined \
  --security-opt label=disable \
  -v "$CNTNER_HOME/cargo_home:/home/devuser/.cargo:U" \
  -v "$CNTNER_HOME/rustup_home:/home/devuser/.rustup:U" \
  -v "$CNTNER_HOME/maven_cache:/home/devuser/.m2:U" \
  -v "$CNTNER_HOME/sbt_cache:/home/devuser/.sbt:U" \
  -v "$CNTNER_HOME/coursier_cache:/home/devuser/.cache/coursier:U" \
  -v "$CNTNER_HOME/uv_cache:/home/devuser/.cache/uv:U" \
  -v "$CNTNER_HOME/vscode_server:/home/devuser/.vscode-server:U" \
  -v "$CNTNER_HOME/devspace:/home/devuser/devspace:U" \
  labdev:v2_devuser
```

**NOTE**  
1. --user devuser  : "devuser" is the user name inside the container

#### Explanation on uid map and gid map
Since you are on **Podman 3.4.4**, you don't have the "luxury" of the modern `keep-id:uid=` syntax. You have to manually define the **User Namespace Mapping**.

When you run a rootless container, Podman creates a "map" that translates IDs inside the container to IDs on your host. The syntax for `--uidmap` is:
`--uidmap [ContainerID]:[HostID]:[Range]`

Here is the line-by-line breakdown of that specific configuration, assuming your **Host User** wants to act as **Container UID 1001**.

---
To explain these parameters accurately, we have to look at the **budget** of User IDs (UIDs) your Linux system allocates to your host user for running containers.

#### 0. The Fact: Your "Subordinate" UID Budget
Every rootless user is given a range of "fake" UIDs in `/etc/subuid`. This is a security feature that prevents a container breakout from having real permissions on your host.

When you run `grep $USER /etc/subuid`, you see something like this:
`devuser:100000:65536`

| Field | Value | Meaning |
| :--- | :--- | :--- |
| **User** | `devuser` | Your host username. |
| **Start ID** | `100000` | The first UID on your host that the container can "borrow." |
| **Range** | `65536` | The total number of UIDs you are allowed to map. |

##### 1. `--uidmap 1001:0:1`
**The "Identity Bridge"**
* **1001 (Container ID):** This is the UID of `devuser` inside the container.
* **0 (Host ID):** In rootless Podman, **0** is a special shorthand for **YOU** (your actual host UID, e.g., 1001 or 5000). 
* **1 (Range):** Only map this single ID.
* **Meaning:** "Map the container's UID 1001 directly to my current host user." This ensures that when `devuser` (1001) writes a file, the host sees it as owned by you.

---

#####  2. `--uidmap 0:1:1001`
**The "Root & System" Fill**
* **0 (Container ID):** The container's root user.
* **1 (Host ID):** The first ID in your host's subordinate UID range (defined in `/etc/subuid`, usually starting at 100000).
* **1001 (Range):** Map the next 1001 IDs.
* **Meaning:** This fills the "gap" before our target ID. It maps container UIDs **0 through 1000** to your host's subuid range. This allows the container to have a `root` user and system users (like `bin`, `mail`, `sshd`) without them overlapping with your host identity.

---

##### 3. `--uidmap 1002:1002:64534`
This command is the final piece of the puzzle that maps the "rest" of the container's users. The syntax is `[ContainerID]:[IntermediateID]:[Range]`.

* **`1002` (Container ID):** The starting UID **inside** the container that we are mapping. We start at 1002 because we already used 0-1000 for system/root and 1001 for your specific `devuser`.
* **`1002` (Intermediate ID):** This is the offset into your subordinate range (the IDs starting at 100,000 on your host).
* **`64534` (Range):** This is the **count** of IDs to map.

**Why 64534?**
It is simple subtraction to stay within your budget of 65,536:
$65,536 \text{ (Total Budget)} - 1002 \text{ (IDs already mapped)} = 64,534$


##### Summary Fact
In Podman 3.4.4, you are manually "stitching" together a 1:1 map. 
1. The **`grep`** command confirms how much "fabric" (UIDs) you have to work with.
2. The **`64534`** value ensures you don't try to use more fabric than you actually have, preventing the kernel from killing the process.

**Note:** If your `grep` command showed a different range (like 100,000), you would change `64534` to `98998` ($100,000 - 1002$). Since 65,536 is the standard default for most Linux distros, 64,534 is the "magic number" that usually works.

---

##### Visualizing the Map


| Container UID | Maps to Host UID | Description |
| :--- | :--- | :--- |
| **0 - 1000** | 100000 - 101000 | System users/Root (safe isolation) |
| **1001** | **Your Host UID** | **Your `devuser` (the bridge)** |
| **1002 - 65535** | 101002 - 165535 | The rest of the possible users |

---


---
@end


## Python Dev  based on uv

#### 1. python uv home page

https://github.com/astral-sh/uv

#### 2. Install `uv` via the Official Script

Since you don’t have Python, we’ll use `curl` to download the standalone `uv` binary. Open your terminal and run:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**What this does:**

  * Downloads the `uv` and `uvx` binaries.
  * Places them in `~/.local/bin`.
  * Automatically attempts to add that directory to your **PATH** in your `.bashrc` or `.profile`.



#### 3. Refresh Your Shell

For the changes to take effect so you can simply type `uv`, you need to reload your shell configuration:

```bash
source $HOME/.bashrc
```

*(If you are using Zsh, use `source ~/.zshrc` instead).*

-----

#### 4. Install Python Using `uv`

Now for the magic. Since your system is empty of Python, you can tell `uv` to grab the version you want.

To install the latest stable Python:

```bash
uv python install
```

To see all available versions you can install:

```bash
uv python list
```

-----

#### 5. Verify the Installation

You can check that everything is wired up correctly by asking `uv` which Python it’s looking at:

```bash
uv run python --version
```

> **Note:** `uv` prefers to run Python inside managed environments rather than installing it "globally" to your system. This keeps your Ubuntu OS clean and prevents version conflicts.

## Rust Devenv Installation

Installing Rust on Ubuntu is best done using **rustup**, the official toolchain manager. This method ensures you get the latest version of the compiler (**rustc**), the package manager (**cargo**), and the language server (**rust-analyzer**).

#### Rust installation homepage
https://rust-lang.org/tools/install/

#### 1. Install Dependencies
Before installing Rust, ensure your system has the necessary build tools. Open your terminal and run:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl build-essential gcc make -y
```

#### 2. Install Rust and Cargo
The following command downloads and runs the official installation script.

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

* **When prompted:** Type `1` and press **Enter** to proceed with the default installation.
* **Update your shell:** Once the script finishes, run this command to add the Rust binaries to your current session's PATH:
    ```bash
    source "$HOME/.cargo/env"
    ```

**Verification:**
Check that everything is installed correctly:
```bash
rustc --version
cargo --version
```

---

#### 3. Install rust-analyzer

```bash
rustup component add rust-analyzer
```

---

#### Useful Commands for Later
* **Update Rust:** `rustup update`
* **Check components:** `rustup component list --installed` (Useful to see if `rustfmt` or `clippy` are there too).
* **Uninstall:** `rustup self uninstall`


###  Export the Image after devtool chain

Bash

```
podman commit labWSL  labdev:v3_devtool_ext
```

---