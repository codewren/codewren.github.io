
A common and organized project folder layout for a Node.js UI web app (frontend) and a Python backend service, which incorporates the Gemini CLI project configuration, generally follows a monorepo-style structure for clean separation of services.

## 📁 Project Folder Layout (gemini-app)

```
gemini-app/
├── .gemini/
│   └── settings.json             <-- GEMINI CLI PROJECT CONFIGURATION FILE
├── .gitignore
├── GEMINI.md                     <-- Project-specific context for Gemini CLI
├── README.md
├── frontend/
│   ├── node_modules/
│   ├── src/
│   │   ├── components/
│   │   └── pages/
│   ├── public/
│   ├── package.json
│   ├── package-lock.json (or yarn.lock/pnpm-lock.yaml)
│   └── .env.local
├── backend/
│   ├── venv/            <-- Python virtual environment
│   ├── src/
│   │   └── api/
│   │       ├── __init__.py
│   │       └── routes.py
│   ├── tests/
│   ├── pyproject.toml   <-- Python package management and configuration
│   └── .env
├── .gitignore
└── README.md


```

-----

### Key Components

  * **/.gemini/**

      * This is the directory for project-specific Gemini CLI configuration.
      * The primary configuration file is **.gemini/settings.json**. Settings here override your user's global settings (usually at *\~/.gemini/settings.json*). This file is used for configuration like trusted MCP servers, model settings, and other project-level preferences.

  * **GEMINI.md**

      * This file provides custom instructions, coding standards, and context to the Gemini model when running Gemini CLI commands from the project root. It helps tailor the AI's behavior to your specific project.

  * **backend/**

      * This folder encapsulates all Python backend code and configuration.
      * Using a *src/* directory keeps the source code separate from configuration files and assets.

  * **frontend/**

      * This folder contains all Node.js/React/Vue UI code and related build configuration.
      * It has its own *package.json* and *node\_modules/*, separating frontend dependencies from any root-level or backend-specific Node dependencies.

-----

This layout is a best practice for projects with separate front and back ends, as it clearly isolates the technology stacks while keeping them in a single, manageable repository.
