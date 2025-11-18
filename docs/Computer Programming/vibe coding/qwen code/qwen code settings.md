
### qwen code settings
https://qwenlm.github.io/qwen-code-docs/zh/cli/configuration/

Qwen Code is a coding agent that lives in the digital world.
https://github.com/QwenLM/qwen-code


The common directory layout for a **Qwen coding agent** often depends on whether you are using the Qwen-Agent framework for developing a custom agent or the official Qwen Code command-line interface (CLI) tool.

In a **general coding project environment** where you are running the **Qwen Code CLI** or a similar agent, the structure is usually based on a typical software project, with a few key additions for agent configuration and memory.

### 📁 Common Agent Project Layout

The following structure represents a common layout you might encounter when developing with or using Qwen's agentic tools in a project:

```
/your_project_root/
├── .qwen/
│   └── settings.json
├── QWEN.md
├── <Below are project related>
├── src/
├── tests/
├── docs/
├── package.json (for Qwen Code CLI which is based on Node.js)
├── requirements.txt (for Qwen-Agent which is Python-based)
├── README.md
└── (Your project-specific files: .git/, .gitignore, etc.)
```



-----

### 📂 Key Directories and Files

| Directory/File | Description | Agent Relevance |
| :--- | :--- | :--- |
| **`.qwen/`** | A hidden directory typically created in the user's home directory (`~/.qwen/`) or the current project root. | Stores **local agent configuration and session state**. |
| **`settings.json`** | Configuration file inside the `.qwen/` directory. | Contains agent settings like **API key**, **model endpoint**, **session token limits**, and **tool/context configurations** (e.g., for vector database integration like Milvus). |
| **`src/`** | Contains the core source code of the project you are building or modifying. | The **primary target** for the coding agent's actions (code generation, refactoring, debugging, etc.). |
| **`tests/`** | Contains unit tests and integration tests for the `src/` code. | The agent might be tasked with **generating or running tests** here. |
| **`docs/`** | Contains documentation, tutorials, and guides for the project. | The agent can be used to **generate or update documentation** here (e.g., JSDoc comments, READMEs). |
| **`requirements.txt` / `package.json`** | Lists the project's dependencies (Python or Node.js/npm). | The agent may read this to **understand the tech stack** or autonomously **install dependencies** to run code. |
| **`README.md`** | The main description and entry point for the project. | The agent can be used to **summarize the project** or **generate/update the README**. |

----
