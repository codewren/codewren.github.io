
##  What are the connectivity requirements for VS Code Server?
ref : https://code.visualstudio.com/docs/remote/faq#_what-are-the-connectivity-requirements-for-vs-code-server


Installation of VS Code Server requires that your local machine have outbound HTTPS (port 443) connectivity to:

```
update.code.visualstudio.com
vscode.download.prss.microsoft.com
```

By default, the Remote - SSH will attempt to download on the remote host, and fail back to downloading VS Code Server locally and transferring it remotely once a connection is established. You can change this behavior with the **remote.SSH.localServerDownload** setting to always download locally and then transfer it, or to never download locally.

```
remote.SSH.localServerDownload
```

The Dev Containers extension always downloads locally and transfers into the container.

You can install extensions manually without an internet connection using the Extensions: Install from VSIX... command, but if you use the extension panel or devcontainer.json to install extensions, your local machine and VS Code Server will need outbound HTTPS (port 443) access to:
```
marketplace.visualstudio.com
*.gallerycdn.vsassets.io (Azure CDN)
```
Finally, some extensions (like C#) download secondary dependencies from download.microsoft.com or download.visualstudio.microsoft.com. Others (like Visual Studio Live Share) may have additional connectivity requirements. Consult the extension's documentation for details if you run into trouble.

All other communication between the server and the VS Code client is accomplished through the following transport channels depending on the extension:

SSH: An authenticated, secure SSH tunnel.
Containers: Docker's configured communication channel (via docker exec).
WSL: A random local port.
You can find a list of locations VS Code itself needs access to in the network connections article.