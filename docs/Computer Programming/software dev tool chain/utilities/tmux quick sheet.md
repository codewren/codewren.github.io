# Essential Tmux Commands Cheat Sheet


In `tmux`, most commands are preceded by a "prefix" key, which by default is **Ctrl+b**. You press this combination first, release it, and then hit the command key.

Here are the 15 most essential `tmux` commands for managing sessions, windows, and panes.

### 1. Sessions (The High-Level Container)

| Command | Action | Example |
| :--- | :--- | :--- |
| `tmux` | Start a new unnamed session. | `tmux` |
| `tmux new -s <name>` | Start a new session with a specific name. | `tmux new -s work` |
| `tmux ls` | List all active sessions. | `tmux ls` |
| `tmux a -t <name>` | Attach to a specific session. | `tmux a -t work` |
| `Ctrl+b` then **d** | **Detach** from the current session (it keeps running in the background). | — |

---

### 2. Windows (Like Tabs in a Browser)



| Command | Action | Description |
| :--- | :--- | :--- |
| `Ctrl+b` then **c** | **Create** a new window. | Adds a new tab to your session. |
| `Ctrl+b` then **,** | **Rename** the current window. | Useful for organizing by task (e.g., "logs", "editor"). |
| `Ctrl+b` then **n** | Go to **Next** window. | Cycle forward through your tabs. |
| `Ctrl+b` then **p** | Go to **Previous** window. | Cycle backward through your tabs. |
| `Ctrl+b` then **0-9** | Switch by **Index**. | Jump directly to window number 0, 1, 2, etc. |

---

### 3. Panes (Splitting One Screen)



| Command | Action | Description |
| :--- | :--- | :--- |
| `Ctrl+b` then **%** | Vertical Split. | Splits the current pane into **Left** and **Right**. |
| `Ctrl+b` then **"** | Horizontal Split. | Splits the current pane into **Top** and **Bottom**. |
| `Ctrl+b` then **Arrow** | Navigate Panes. | Use Up/Down/Left/Right arrows to move focus. |
| `Ctrl+b` then **z** | **Zoom** Pane. | Toggles the current pane to full screen and back. |
| `Ctrl+b` then **x** | **Close** Pane. | Prompts you to kill the current active pane. |

---

### Pro Tip: The "Help" Command
If you ever forget a shortcut while inside a session, press `Ctrl+b` followed by **?**. This will list every single keybinding available to you. Press **q** to exit the help screen.

========
-------