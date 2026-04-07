# uv Tool Quick Sheet

As of 2026, **uv** has become the industry standard for Python management because it consolidates several tools (pip, pyenv, poetry, pipx) into one extremely fast Rust-based binary.

This quick sheet follows the standard developer workflow for a project.

---

## 1. Project Initialization
Instead of manually creating folders, use `uv init` to scaffold a project with a `pyproject.toml`.

| Command | Description |
| :--- | :--- |
| `uv init my-project` | Create a new project directory with basic structure. |
| `uv init --lib` | Initialize as a library (includes `src` layout). |
| `uv python pin 3.12` | Pin the project to a specific Python version (creates `.python-version`). |

---

## 2. Environment Management
`uv` manages virtual environments automatically, but you can control them manually if needed.

| Command | Description |
| :--- | :--- |
| `uv venv` | Create a `.venv` in the current directory. |
| `uv venv --python 3.13` | Create an environment with a specific Python version. |
| `source .venv/bin/activate` | **Manual activation** (Optional; `uv run` handles this automatically). |

---

## 3. Installing & Adding Packages
`uv` uses a "declarative" approach. Adding a package updates your `pyproject.toml` and `uv.lock` simultaneously.

| Command | Description |
| :--- | :--- |
| `uv add requests` | Add a package to project dependencies and install it. |
| `uv add ruff --dev` | Add a package as a **development** dependency. |
| `uv sync` | Install all dependencies listed in the lockfile (ideal for fresh clones). |
| `uv pip install -r reqs.txt` | **Legacy mode**: Fast install from a standard requirements file. |

---

## 4. Updating & Removing Packages
Updating in `uv` is handled primarily through the lockfile to ensure reproducibility.

| Command | Description |
| :--- | :--- |
| `uv lock --upgrade` | Update **all** packages to the latest versions allowed by `pyproject.toml`. |
| `uv lock --upgrade-package flask` | Update only a specific package. |
| `uv remove requests` | Remove a package from the project and uninstall it. |

---

## 5. Checking & Validating
Use these commands to inspect your environment and ensure security/consistency.

| Command | Description |
| :--- | :--- |
| `uv tree` | Display a visual tree of all installed dependencies and their sub-dependencies. |
| `uv pip list` | List installed packages in the current environment (pip-style). |
| `uv audit` | **Security**: Check your dependencies for known vulnerabilities (CVEs). |

---

## 6. Execution & Tools
One of `uv`'s best features is running code without manual environment activation.

| Command | Description |
| :--- | :--- |
| `uv run main.py` | Run a script inside the project's virtual environment. |
| `uv run python` | Open a REPL within the project environment. |
| `uvx ruff check` | **Ephemeral**: Run a tool (like Ruff or Black) without installing it permanently. |
| `uv tool install ruff` | **Global**: Install a CLI tool globally (replaces `pipx`). |

---
@end