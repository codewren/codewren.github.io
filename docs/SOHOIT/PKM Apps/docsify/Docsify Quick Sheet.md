# Docsify: Quick Guide and Commands

Docsify is a lightweight **markdown-to-site** generator that renders your files on the fly without static builds. Unlike Jekyll or Hugo, it doesn’t generate HTML files; it uses a small script to inject content into a single `index.html`, making deployment exceptionally fast and maintenance-free.

It is ideal for **real-time documentation** because updates appear instantly upon refreshing. With a rich plugin ecosystem, you can easily add full-text search, emoji support, and multi-theme switching by simply editing your configuration file, keeping your workflow centered entirely on writing Markdown.

---

### Top 10 Most Frequent Commands

To use these, ensure you have the CLI installed via `npm i docsify-cli -g`.

| Command | Purpose |
| --- | --- |
| `docsify init ./docs` | **Initialize** a new project in the specified folder. |
| `docsify serve ./docs` | **Preview** your site locally (defaults to `localhost:3000`). |
| `docsify serve -p 8080` | Run the local server on a **custom port**. |
| `docsify generate .` | Automatically create a **sidebar** based on file structure. |
| `docsify help` | View the **full list** of available CLI commands. |
| `npm i docsify-cli -g` | **Install** or update the Docsify global command tool. |
| `[text](file.md)` | Standard Markdown **link** to navigate between pages. |
| `window.$docsify = {}` | Core **config object** in `index.html` for plugins/settings. |
| `loadSidebar: true` | Enable the **custom sidebar** in your configuration. |
| `search: 'auto'` | Activate the **built-in search** plugin functionality. |

---
@end
