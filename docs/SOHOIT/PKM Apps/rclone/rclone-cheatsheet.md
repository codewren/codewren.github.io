# rclone Quick Sheet (FAQ style)

Remote = `name:` · path in remote = `name:dir/file` · local = plain path.

## "How do I…?"

| Question                            | Command                                      |
| ----------------------------------- | -------------------------------------------- |
| Check my rclone version?            | `rclone version`                             |
| See my remotes?                     | `rclone listremotes --long`                  |
| Add a remote?                       | `rclone config`                              |
| Change one setting?                 | `rclone config update mys3 region=eu-west-1` |
| Delete a remote?                    | `rclone config delete oldremote`             |
| Turn a password into obscured form? | `rclone obscure 'MyPass'`                    |
| Read an obscured password back?     | `rclone reveal 'obscured-string'`            |
| Fix an expired login/token?         | `rclone config reconnect gdrive:`            |
| Test a command safely first?        | add `--dry-run -v`                           |
| Upload without deleting anything?   | `rclone copy /data gdrive:backup -P`         |
| Make dest match source exactly?     | `rclone sync /data gdrive:backup -P`         |
| Two-way sync?                       | `rclone bisync /data gdrive:data --resync`   |
| Verify a transfer?                  | `rclone check /data gdrive:backup`           |
| Delete a whole folder fast?         | `rclone purge gdrive:oldbackup`              |
|                                     |                                              |

---

## Version

### `rclone version`
Show version, OS and build info.
- **Syntax:** `rclone version [--check]`
- **Options:**
  - `--check` — compare against the latest release:
    `rclone version --check`
  - upgrade the binary in place:
    `rclone selfupdate`
- **Example:**
  ```bash
  rclone version
  # rclone v1.68.2
  # - os/version: ubuntu 24.04
  # - go/version: go1.23.3
  ```

---

## Dry-run (test before you break something)

### `--dry-run` / `-n`
Show exactly what a command *would* do, change nothing. Works on `sync`, `copy`, `move`, `delete`, `purge`, `bisync`, `dedupe`.
- **Syntax:** `rclone <cmd> SRC DST --dry-run -v`
- **Options:**
  - `-v` — list each action (use `-vv` for debug detail):
    `rclone sync /data gdrive:backup --dry-run -v`
  - `-i` / `--interactive` — actually run, but ask before each destructive step:
    `rclone sync /data gdrive:backup -i`
  - `--max-delete N` — real safety net on a live run; abort if it would delete more than N:
    `rclone sync /data gdrive:backup --max-delete 50`
- **Example:**
  ```bash
  rclone sync /data gdrive:backup --dry-run -v
  # NOTICE: report.pdf: Skipped copy as --dry-run is set
  # NOTICE: old.tmp: Skipped delete as --dry-run is set
  ```

---

## Config

### `rclone config`
Interactive menu to add, edit, rename or delete remotes.
- **Syntax:** `rclone config`
- **Options (menu keys):** `n` new · `e` edit · `d` delete · `r` rename · `q` quit
- **Example:**
  ```bash
  rclone config
  # n) New remote -> name: gdrive -> storage: drive -> follow prompts
  ```

### `rclone listremotes`
List configured remote names.
- **Syntax:** `rclone listremotes [--long]`
- **Options:**
  - `--long` — show the backend type too:
    `rclone listremotes --long`
- **Example:**
  ```bash
  rclone listremotes
  # gdrive:
  # mys3:
  ```

### `rclone config show`
Print the config for all remotes, or just one.
- **Syntax:** `rclone config show [remote:]`
- **Options:**
  - show the config file path:
    `rclone config file`
  - redact secrets before sharing/pasting:
    `rclone config redacted`
  - dump everything as JSON:
    `rclone config dump`
- **Example:**
  ```bash
  rclone config show gdrive:
  # [gdrive]
  # type = drive
  # token = {"access_token":"ya29..."}
  ```

### `rclone config create`
Create a remote from the command line (no prompts).
- **Syntax:** `rclone config create NAME TYPE [key=value ...]`
- **Options:**
  - `--obscure` — obscure any plaintext password you pass in:
    `rclone config create sftpbox sftp host=srv user=me pass=Secret123 --obscure`
  - `--all` — accept advanced options too:
    `rclone config create box box --all`
  - `--non-interactive` — never prompt (scripts/CI):
    `rclone config create gdrive drive --non-interactive`
- **Example:**
  ```bash
  rclone config create mys3 s3 provider=AWS \
    access_key_id=AKIA... secret_access_key=SECRET... region=us-east-1
  ```

### `rclone config update`
Change keys on an existing remote.
- **Syntax:** `rclone config update NAME key=value [...]`
- **Options:**
  - empty value deletes the key:
    `rclone config update mys3 region=`
  - `--obscure` — for password fields:
    `rclone config update sftpbox pass=NewSecret --obscure`
  - `--all` — reach advanced options:
    `rclone config update gdrive scope=drive.readonly --all`
- **Example:**
  ```bash
  rclone config update mys3 region=eu-west-1
  ```

### `rclone config delete`
Remove a remote from the config.
- **Syntax:** `rclone config delete NAME`
- **Options:** none — confirm first with `rclone config show oldremote:`
- **Example:**
  ```bash
  rclone config show oldremote:      # check what you're about to lose
  rclone config delete oldremote
  ```

---

## Passwords & logins

### `rclone obscure`
Convert a plaintext password into the obscured form rclone stores in config.
- **Syntax:** `rclone obscure PASSWORD`
- **Options:**
  - read from stdin so the password isn't in your shell history:
    `echo -n 'MyPass' | rclone obscure -`
  - use inline when creating a crypt remote:
    `rclone config create secret crypt remote=gdrive:enc password="$(rclone obscure 'MyPass')"`
- **Example:**
  ```bash
  rclone obscure 'MyPass'
  # 7_UtIz2BqRGkCYhkqBHqMP7Pyw
  ```

### `rclone reveal`
Turn an obscured password back into plaintext (reverse of `obscure`).
- **Syntax:** `rclone reveal OBSCURED_STRING`
- **Options:**
  - read from stdin:
    `echo -n '7_UtIz2BqRGkCYhkqBHqMP7Pyw' | rclone reveal -`
  - recover a stored password straight out of the config:
    `rclone reveal "$(rclone config show sftpbox: | awk '/^pass/{print $3}')"`
- **Example:**
  ```bash
  rclone reveal '7_UtIz2BqRGkCYhkqBHqMP7Pyw'
  # MyPass
  ```
  Obscuring is **not** encryption — anyone with the config file can run `reveal`.

### `rclone config reconnect`
Re-run the OAuth/login flow for a remote — the fix for expired tokens.
- **Syntax:** `rclone config reconnect remote:`
- **Options:**
  - `--auto-confirm` — no "are you sure" prompt:
    `rclone config reconnect gdrive: --auto-confirm`
  - headless machine: get the token elsewhere, paste it in:
    `rclone authorize "drive"` then `rclone config update gdrive token='{"access_token":...}'`
  - revoke access at the provider instead:
    `rclone config disconnect gdrive:`
- **Example:**
  ```bash
  rclone config reconnect gdrive:
  ```

---

## Browse

### `rclone ls`
List files with sizes, recursively.
- **Syntax:** `rclone ls remote:path`
- **Options (sibling commands):**
  - with modtime: `rclone lsl gdrive:backup`
  - directories only: `rclone lsd gdrive:`
  - bare names for scripts: `rclone lsf -R gdrive:backup`
  - JSON with metadata: `rclone lsjson -R gdrive:backup`
- **Example:**
  ```bash
  rclone ls gdrive:backup
  #  1048576 photos/img1.jpg
  #     2048 notes.txt
  ```

### `rclone size`
Total object count and bytes for a path.
- **Syntax:** `rclone size remote:path`
- **Options:**
  - `--json` — machine-readable:
    `rclone size gdrive:backup --json`
- **Example:**
  ```bash
  rclone size gdrive:backup
  # Total objects: 1.204k
  # Total size: 4.201 GiB
  ```

### `rclone about`
Show quota: used / free / total.
- **Syntax:** `rclone about remote:`
- **Options:**
  - `--json` — for scripts:
    `rclone about gdrive: --json`
- **Example:**
  ```bash
  rclone about gdrive:
  # Total:   15 GiB
  # Used:    9.612 GiB
  # Free:    5.388 GiB
  ```

---

## Sync

### `rclone copy`
Copy new/changed files src → dst. **Never deletes.**
- **Syntax:** `rclone copy SRC DST [flags]`
- **Options:**
  - `-P` — live progress:
    `rclone copy /data gdrive:backup -P`
  - `--transfers 8` — more parallel uploads:
    `rclone copy /data gdrive:backup --transfers 8`
  - `--max-age 24h` — only recently changed files:
    `rclone copy /data gdrive:backup --max-age 24h`
  - `--exclude` — skip patterns:
    `rclone copy /data gdrive:backup --exclude "*.tmp"`
- **Example:**
  ```bash
  rclone copy /data gdrive:backup -P
  ```

### `rclone sync`
Make dst identical to src — **deletes extra files on dst**.
- **Syntax:** `rclone sync SRC DST [flags]`
- **Options:**
  - `--dry-run -v` — always run this first:
    `rclone sync /data gdrive:backup --dry-run -v`
  - `--backup-dir` — keep replaced/deleted files instead of losing them:
    `rclone sync /data gdrive:backup --backup-dir gdrive:trash/$(date +%F)`
  - `--max-delete 100` — abort a run that deletes too much:
    `rclone sync /data gdrive:backup --max-delete 100`
  - `--checksum` — compare by hash instead of modtime:
    `rclone sync /data gdrive:backup --checksum`
- **Example:**
  ```bash
  rclone sync /data gdrive:backup -P \
    --backup-dir gdrive:trash/$(date +%F) --max-delete 100
  ```

### `rclone move`
Copy then delete from the source.
- **Syntax:** `rclone move SRC DST [flags]`
- **Options:**
  - `--delete-empty-src-dirs` — tidy up after:
    `rclone move /outbox gdrive:inbox --delete-empty-src-dirs`
  - `--min-age 5m` — skip files still being written:
    `rclone move /outbox gdrive:inbox --min-age 5m`
- **Example:**
  ```bash
  rclone move /outbox gdrive:inbox --delete-empty-src-dirs -P
  ```

### `rclone copyto`
Copy a single file, optionally renaming it.
- **Syntax:** `rclone copyto SRCFILE DSTFILE`
- **Options:**
  - move-and-rename instead:
    `rclone moveto /data/f.txt gdrive:backup/f2.txt`
- **Example:**
  ```bash
  rclone copyto /data/f.txt gdrive:backup/f2.txt
  ```

### `rclone check`
Compare src and dst by hash and report differences.
- **Syntax:** `rclone check SRC DST [flags]`
- **Options:**
  - `--one-way` — ignore extra files on dst:
    `rclone check /data gdrive:backup --one-way`
  - `--size-only` — faster, no hashing:
    `rclone check /data gdrive:backup --size-only`
  - `--combined diff.txt` — write a full report:
    `rclone check /data gdrive:backup --combined diff.txt`
  - crypt remotes need the crypt-aware version:
    `rclone cryptcheck /data secret:`
- **Example:**
  ```bash
  rclone check /data gdrive:backup --one-way
  # 0 differences found
  ```

### `rclone bisync`
Two-way sync between two paths.
- **Syntax:** `rclone bisync PATH1 PATH2 [flags]`
- **Options:**
  - `--resync` — required on the very first run to set a baseline:
    `rclone bisync /data gdrive:data --resync`
  - `--check-access` — refuse to run if a marker file is missing (guards against an unmounted path):
    `rclone bisync /data gdrive:data --check-access`
  - `--conflict-resolve newer` — auto-resolve both-sides-changed:
    `rclone bisync /data gdrive:data --conflict-resolve newer`
- **Example:**
  ```bash
  rclone bisync /data gdrive:data --resync      # first run
  rclone bisync /data gdrive:data -P            # every run after
  ```

---

## Delete

### `rclone delete`
Delete files but keep the directory structure.
- **Syntax:** `rclone delete remote:path [flags]`
- **Options:**
  - `--min-age 30d` — retention pruning:
    `rclone delete gdrive:logs --min-age 30d`
  - `--dry-run` — see the list first:
    `rclone delete gdrive:logs --min-age 30d --dry-run -v`
  - `--include` — only certain files:
    `rclone delete gdrive:logs --include "*.log"`
- **Example:**
  ```bash
  rclone delete gdrive:logs --min-age 30d --dry-run -v   # check
  rclone delete gdrive:logs --min-age 30d                # commit
  ```

### `rclone purge`
Delete a directory and everything inside it (fast, no listing).
- **Syntax:** `rclone purge remote:path`
- **Options:**
  - `--dry-run` — confirm the target first:
    `rclone purge gdrive:oldbackup --dry-run -v`
  - only remove it if empty:
    `rclone rmdir gdrive:emptydir`
  - empty the provider's trash / stale uploads:
    `rclone cleanup gdrive:`
- **Example:**
  ```bash
  rclone size gdrive:oldbackup       # see what you're deleting
  rclone purge gdrive:oldbackup
  ```

---

## Flags worth memorising

| Flag | Effect | Example |
|---|---|---|
| `--dry-run` / `-n` | Change nothing, just report | `rclone sync /data gdrive:bk --dry-run -v` |
| `-P` | Live progress | `rclone copy /data gdrive:bk -P` |
| `-i` | Confirm each destructive action | `rclone sync /data gdrive:bk -i` |
| `-v` / `-vv` | Verbose / debug | `rclone copy /data gdrive:bk -vv` |
| `--transfers N` | Parallel transfers (default 4) | `rclone copy /data gdrive:bk --transfers 8` |
| `--bwlimit 10M` | Bandwidth cap | `rclone copy /data gdrive:bk --bwlimit 10M` |
| `--exclude PAT` | Skip matching files | `rclone sync /data gdrive:bk --exclude "*.tmp"` |
| `--backup-dir DIR` | Keep replaced & deleted files | `rclone sync /data gdrive:bk --backup-dir gdrive:trash` |
| `--max-delete N` | Abort if sync deletes too much | `rclone sync /data gdrive:bk --max-delete 50` |
| `--log-file F` | Log to file (cron) | `rclone sync /data gdrive:bk --log-file /var/log/rc.log --log-level INFO` |

**Rules of thumb:** 
- dry-run every `sync` / `move` / `delete` · 
- `rclone sync SRC DST` deletes on **DST** · 
- use `copy` when unsure · 
- obscured passwords are reversible with `rclone reveal`.

*Docs: <https://rclone.org/commands/>*
