# rclone `crypt` Remote — Client-Side Encryption Guide

> **Terminology note:** rclone doesn't have an "encryption repo." The feature is called a **crypt remote**. 

This guide covers the case where **filename and directory-name encryption are turned off** — only file *contents* are encrypted. Name-encryption options are omitted accordingly.

## 1. Overview & Architecture

**Core value proposition.** `crypt` is a wrapping backend that transparently encrypts data before it is written to an underlying remote (Google Drive, S3, B2, local disk, etc.) and decrypts on read. Encryption is **client-side**: the storage provider only ever sees ciphertext. There is no server component and no key escrow.

**Mental model.** `crypt` is not a storage backend itself; it sits *on top of* another remote and rewrites the byte stream.

```
  rclone command (copy/sync/mount)
            │  plaintext
            ▼
    ┌───────────────┐
    │  crypt remote │  ← password + salt → scrypt → 80 bytes key material
    │  (wrapper)    │     encrypts file content (NaCl SecretBox)
    └───────┬───────┘
            │  ciphertext (names left in plaintext in this setup)
            ▼
    ┌───────────────┐
    │ Google Drive  │  e.g. mydrive:Backups/PhotoArchive
    │ (backend)     │
    └───────────────┘
```

Only what lives *inside* `remote:path` is encrypted; anything outside that path is untouched.

**Key capabilities.** Authenticated encryption of file contents (confidentiality + integrity), streaming chunked encryption (no need to hold whole files in memory), and portability — the same password/salt reproduces the same remote on any machine.

---

## 2. Prerequisites & Setup

**System requirements.** A single `rclone` binary (Go, statically linked). No runtime dependencies. Works on Linux, macOS, Windows, BSD. Install via your platform's method — `brew install rclone`, `choco install rclone`, `apt install rclone`, `pacman -S rclone`, or the official installer (`curl https://rclone.org/install.sh | sudo bash`).

**Precondition.** A Google Drive remote must already be configured and working before you wrap it. Verify with:

```bash
rclone lsd mydrive:
```

---

## 3. Core Concepts & Fundamentals

**The two secrets.**

- **Password (`password`)** — the primary passphrase. Required.
- **Salt (`password2`)** — an *optional* second passphrase that rclone confusingly labels "salt." Strongly recommended.

Both are fed into **scrypt** (a key-derivation function, explained in §4) to produce **80 bytes** of key material: 32 bytes for the data cipher key, 32 bytes for the name cipher key, and 16 bytes for the name tweak. Even when name encryption is off, all 80 bytes are still derived — the data key is the part that matters here.

Rclone uses scrypt with parameters **N=16384, r=8, p=1** with the optional user-supplied salt (`password2`). If the user doesn't supply a salt, rclone uses a built-in internal one.

> **Security warning.** scrypt makes it impractical to mount a dictionary attack on rclone-encrypted data — but for full protection you should always use a salt. If you omit `password2`, rclone substitutes a **hardcoded built-in salt**, which means anyone can run scrypt against your password with a publicly known salt. Always set your own `password2`.

---

## 4. The Cryptography in Detail

### 4.1 scrypt — the key-derivation function (KDF)

You never encrypt with your password directly. Your password is first stretched into a real cryptographic key by **scrypt** (RFC 7914), a **password-based key-derivation function**.

Why scrypt rather than a plain hash:

- **Memory-hard.** scrypt is deliberately designed to consume a large, tunable amount of RAM during derivation. This defeats the economic advantage of GPUs, FPGAs, and ASICs — the hardware that makes brute-forcing cheap password hashes fast. An attacker guessing passwords must pay the full memory cost for *every* guess.
- **Slow by design.** The work factor makes each guess expensive, so even a weak-ish password gains meaningful resistance to dictionary attacks.

scrypt takes these inputs:

- **password** — your passphrase.
- **salt** — random/secret bytes that make identical passwords produce different keys (in rclone this is `password2`, or the built-in default if unset).
- **N** — CPU/memory cost parameter (a power of two). rclone fixes it at **16384**.
- **r** — block size, affecting memory use. rclone fixes it at **8**.
- **p** — parallelization factor. rclone fixes it at **1**.
- **key length** — how many bytes to output. rclone requests **80**.

These parameters are **hardcoded constants in rclone** — they are not configurable and are identical in every rclone version, which is what guarantees old data stays decryptable (see §5).

### 4.2 The content cipher — NaCl SecretBox (XSalsa20 + Poly1305)

File contents are encrypted using **NaCl SecretBox**, a well-vetted authenticated-encryption construction. It combines two primitives:

- **XSalsa20** — a stream cipher (an extended-nonce variant of Salsa20) that provides **confidentiality**. It generates a keystream that is XORed with the plaintext.
- **Poly1305** — a message-authentication code (MAC) that provides **integrity and authenticity**. Each chunk gets a 16-byte authentication tag; if a single ciphertext byte is altered, decryption fails loudly rather than returning corrupt data.

**Is this symmetric or asymmetric?** It is **symmetric** (also called *secret-key*) encryption. The **same key encrypts and decrypts** — there is no public/private key pair. That single key is the 32-byte data key derived by scrypt from your password + salt. There is therefore no separate "public key" to share and no "private key" held elsewhere; whoever knows the password (or holds the config file) can both read and write.

**How files are laid out on disk:**

- Each file begins with an **8-byte magic header** (`RCLONE\x00\x00`) followed by a **24-byte random nonce**.
- The body is split into **64 KiB chunks**. Each chunk is encrypted as an independent SecretBox and carries its own 16-byte Poly1305 tag (so each chunk is 64 KiB + 16 bytes).
- The nonce is generated once from the OS cryptographically-secure RNG, then **incremented per chunk**, guaranteeing every block uses a unique nonce. The reuse probability is negligible even at exabyte scale.

Because a fresh random nonce is chosen for every file, **encrypting the same file twice yields different ciphertext** — identical plaintext does not reveal itself on the remote. The 64 KiB chunk size is a deliberate tradeoff: below it the per-chunk MAC overhead dominates, above it CPU-cache effects reduce throughput. It is not tunable.

---

## 5. Configuration & Parameter Reference (Google Drive backend, no name encryption)

A crypt setup is always **two remotes**: the raw Google Drive backend, and a crypt remote that wraps it. Below is a realistic pair as they appear in `~/.config/rclone/rclone.conf`.

> **Naming convention used here.** The name in `[brackets]` is a label *you* invent when you run `rclone config` — it is **not** a keyword. To make that obvious, every user-chosen remote name in this guide starts with `my` (`mydrive`, `myphoto`). The `type =` line, by contrast, holds a **built-in rclone type name** (`drive`, `crypt`) that you must spell exactly. So `type = drive` is fixed vocabulary; `[mydrive]` is your free choice.

### Step 1 — the underlying Google Drive remote (`mydrive`)

You configure this first with `rclone config` → `drive`, supplying your own Google Cloud OAuth **client ID/secret** (strongly recommended over rclone's shared default, which is heavily rate-limited). The interactive flow opens a browser for consent and writes back the OAuth `token`.

```ini
[mydrive]
type = drive
client_id = 605919805393-odnfmddo2v24ffodmg80j6ht4oi4kftn.apps.googleusercontent.com
client_secret = GOCSPX-xxxxxxxxxxxxxxxxxxxxxxxx
scope = drive
root_folder_id = 1A2b3C4d5E6f7G8h9I0jKlMnOpQrStUv
token = {"access_token":"ya29.REDACTED","token_type":"Bearer","refresh_token":"1//REDACTED","expiry":"2026-08-01T09:15:42.1234567Z"}
team_drive =
```

Notable Drive fields:

- **`client_id` / `client_secret`** — your own Google Cloud OAuth credentials. Using your own avoids the throttling that hits rclone's built-in shared client.
- **`scope = drive`** — full read/write access to your Drive. Alternatives include `drive.readonly` or `drive.file` (rclone-created files only).
- **`root_folder_id`** — pins the remote to a specific Drive folder by its ID (the string in the folder's URL) instead of the Drive root. Handy for scoping and for Shared Drives / "Computers" folders.
- **`token`** — the OAuth access + refresh token blob rclone manages automatically; the access token is short-lived and auto-refreshed via the refresh token. **This is a secret.**
- **`team_drive`** — set to a Shared Drive ID if the target lives on one; blank for a personal Drive.

### Step 2 — the crypt remote wrapping it (`myphoto`)

Then `rclone config` → `crypt`, pointing `remote` at a path inside the Drive remote and disabling name encryption:

```ini
[myphoto]
type = crypt
remote = mydrive:Backups/PhotoArchive
filename_encryption = off
directory_name_encryption = false
password = aB3dE_fGhIjKlMnOpQrStU
password2 = zY9xW_vUtSrQpOnMlKjIhG
suffix = .bin
```

**Where does `Backups/PhotoArchive` come from?** It is **not** anything rclone or Google predefines — it is simply a folder path *you choose* inside your own Google Drive, written as `remotename:folder/subfolder`. Breaking it down:

- `mydrive:` — the backend remote from Step 1 (i.e. "my Google Drive," scoped to `root_folder_id` if you set one).
- `Backups/PhotoArchive` — a plain folder path relative to that Drive. You could equally write `remote = mydrive:` (the Drive root), `mydrive:Photos`, or `mydrive:anything/you/like`.

You do **not** have to create this folder beforehand — rclone creates missing folders on the first upload. Pick any path; just point the crypt remote at it consistently. Everything rclone writes through `myphoto:` lands **encrypted** inside `mydrive:Backups/PhotoArchive`; anything placed in that folder by other means is not encrypted.

The crypt remote is named `myphoto` — deliberately *not* something like "encrypted" or "vault," so the name itself doesn't advertise that the folder holds encrypted data.

> **Warning.** In the file, `password` and `password2` appear "obscured," not truly encrypted — `rclone reveal <obscured>` prints them back in cleartext. Both the `token` above and these fields make the whole config file a high-value secret. Protect it (see §7).

### Crypt parameters relevant to this setup

**`remote`** — the underlying `backend:path` that `crypt` wraps (`mydrive:Backups/PhotoArchive`). Everything under this path is content-encrypted. It must not point back at the crypt remote itself (rclone rejects that).

**`filename_encryption = off`** — filenames are left in plaintext; only file *content* is encrypted, and a `.bin` suffix is appended to each file. Your provider sees real filenames but cannot read the contents.

**`directory_name_encryption = false`** — directory names left readable. (Required to be `false` here, since name encryption is disabled.)

**`password`** — the primary passphrase (mandatory).

**`password2`** — the salt/second passphrase. Optional but strongly recommended. During setup rclone offers to generate a strong random one; if you accept, **record it immediately** — losing it means losing your data.

**`suffix`** — extension appended to encrypted files (default `.bin`; `none` for empty). Cosmetic.

### Basic usage

Each command below is followed by the output you should expect.

**Upload local photos** (encrypts on the way up):

```bash
rclone copy ~/Pictures myphoto: --progress
```

```
Transferred:        1.204 GiB / 1.204 GiB, 100%, 18.402 MiB/s, ETA 0s
Transferred:          312 / 312, 100%
Elapsed time:      1m7.4s
```

A per-file progress display that ends at 100%. No news is good news — rclone prints errors only on failure. If you re-run it, already-uploaded files are skipped and `Transferred: 0 / 0` appears.

**Download / restore** (decrypts on the way down):

```bash
rclone copy myphoto: ~/Restore --progress
```

```
Transferred:        1.204 GiB / 1.204 GiB, 100%, 20.115 MiB/s, ETA 0s
Transferred:          312 / 312, 100%
Elapsed time:      1m1.2s
```

Files land in `~/Restore` with their **original names and contents**, decrypted on the fly.

**Verify the underlying Drive holds only ciphertext.** Listing through the *crypt* remote shows real names and true sizes:

```bash
rclone ls myphoto:
```

```
  4587213 IMG_2041.jpg
  3912004 IMG_2042.jpg
  5120847 vacation/beach.jpg
```

Listing the *same data through the raw Drive backend* shows the encrypted `.bin` files as Google actually stores them (note the slightly larger sizes from the header + per-chunk tags):

```bash
rclone ls mydrive:Backups/PhotoArchive
```

```
  4587277 IMG_2041.jpg.bin
  3912068 IMG_2042.jpg.bin
  5120943 vacation/beach.jpg.bin
```

With name encryption off, filenames stay readable and only a `.bin` suffix is added; the bytes inside are unreadable.

**Mount as a decrypted filesystem:**

```bash
rclone mount myphoto: /mnt/photos --vfs-cache-mode writes
```

```
(no output — the process runs in the foreground and blocks)
```

The terminal appears to hang; that is normal — the mount stays active until you press `Ctrl-C` or unmount. In another terminal, `ls /mnt/photos` now shows your decrypted files as a normal directory.

---

## 6. Recovery & Version Stability

**Can the key or password be recovered?**

The **derived key cannot be reverse-engineered** from the ciphertext — that is the whole point of scrypt + SecretBox. There is no backdoor, no reset, and no way for rclone's authors or Google to recover it.

The **password itself** exists in only two recoverable places:

1. **Your config file.** As long as you have the config file, you can decrypt your data. The stored value is only lightly *obscured*, not truly encrypted, so the config file *is* effectively the plaintext password — treat it as a secret. Recover the readable password from an obscured string with `rclone reveal <obscured>`.
2. **Your memory / password manager.** Without the config file, as long as you remember the password (and salt, if set), you can re-create the configuration and regain access.

If you lose **both** the config file **and** the password (and `password2`, if set), the data is **permanently unrecoverable**. There is no fallback.

> **Critical.** Because the salt is never written to the remote, `password2` is exactly as unrecoverable and exactly as essential as `password`. Back up both.

**Will keys/passwords change if rclone is updated?**

No. The scheme is **stable across versions by design**. The parameters — scrypt `N=16384, r=8, p=1`, NaCl SecretBox (XSalsa20 + Poly1305), 64 KiB chunks, the built-in default salt — are fixed constants baked into the on-disk format. Changing any of them would make old data undecryptable, so the project treats them as frozen. Upgrading rclone does **not** re-derive keys, re-encrypt data, or invalidate an existing crypt remote. Your password and salt remain valid indefinitely.

**Rotating the password.** There is no in-place "change password" — the key is derived from the password, so a new password means new keys. Procedure: create a second crypt remote with the new password/salt pointing at a new folder, then `rclone sync oldcrypt: newcrypt:` to decrypt-and-re-encrypt everything, then delete the old copy.

---

## 7. Troubleshooting & Common Pitfalls

| Issue / Error | Root Cause | Solution |
| :--- | :--- | :--- |
| Data decrypts to garbage / `failed to authenticate decrypted block` | Wrong `password` **or** wrong `password2`. Both must match the values used at encryption time. | Restore the exact password *and* salt. If salt was omitted originally, leave `password2` blank. |
| Can't decrypt after moving to a new machine | Salt was auto-generated and never recorded; only `password` was remembered. | Copy the original `rclone.conf`, or supply the recorded `password2`. Salt is not stored on the remote and cannot be recovered otherwise. |
| `can't point crypt remote at itself` | `remote =` references the crypt remote instead of the underlying storage. | Point `remote` at the real backend, e.g. `mydrive:Backups/PhotoArchive`. |
| Password visible in config | Obscuring is *not* encryption; it's trivially reversible via `rclone reveal`. | Protect the config with `rclone config` → "Set configuration password" (AES-encrypts the whole file), or use `RCLONE_CONFIG_PASS`. |

---

**Bottom line:** Your `password` and `password2` are the only things standing between you and your data — they are recoverable only from your own config file or your own records, they never change across rclone versions, and if both are lost the data is gone for good. Back them up somewhere independent of the machine running rclone.
