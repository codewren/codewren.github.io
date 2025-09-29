# Git Credential Storage

Ref  : [https://git-scm.com/book/en/v2/Git-Tools-Credential-Storage](https://git-scm.com/book/en/v2/Git-Tools-Credential-Storage) &#x20;

Take-aways:

Fortunately, Git has a credentials system that can help with this. Git has a few options provided in the box:

* The default is not to cache at all. Every connection will prompt you for your username and password.
* The “cache” mode keeps credentials in memory for a certain period of time. None of the passwords are ever stored on disk, and they are purged from the cache after 15 minutes.
* The “store” mode saves the credentials to a plain-text file on disk, and they never expire. This means that until you change your password for the Git host, you won’t ever have to type in your credentials again. The downside of this approach is that your passwords are stored in cleartext in a plain file in your home directory.
* If you’re using macOS, Git comes with an “osxkeychain” mode, which caches credentials in the secure keychain that’s attached to your system account. This method stores the credentials on disk, and they never expire, but they’re encrypted with the same system that stores HTTPS certificates and Safari auto-fills.
* If you’re using Windows, you can enable the **Git Credential Manager** feature when installing [Git for Windows](https://gitforwindows.org/) or separately install [the latest GCM](https://github.com/git-ecosystem/git-credential-manager/releases/latest) as a standalone service. This is similar to the “osxkeychain” helper described above, but uses the Windows Credential Store to control sensitive information. It can also serve credentials to WSL1 or WSL2. See [GCM Install Instructions](https://github.com/git-ecosystem/git-credential-manager#readme) for more information.

Some of these helpers have options. The “store” helper can take a `--file <path>` argument, which customizes where the plain-text file is saved (the default is `~/.git-credentials`)

```
// Some code
$ git config --global credential.helper store
```
