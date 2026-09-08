## 1. What is Rclone?

Rclone ("rsync for cloud storage") is a command-line program to manage files on cloud storage. It syncs, copies, moves, mounts, and manages data across local disks, network storage, and 70+ cloud providers — all through one unified interface.

- Written in Go, single static binary, cross-platform (Linux, macOS, Windows, BSD, Android via Termux).
- Open source, actively maintained.
- Often called "the Swiss Army knife of cloud storage."

---

## 2. How It Works

- Rclone treats every storage backend (local disk, S3 bucket, Google Drive, FTP server, etc.) as a **"remote"**.
- Each remote is configured once (auth, endpoint, options) and saved in a config file (`rclone.conf`).
- Commands operate on paths in the form `remote:path/to/folder`.
- Under the hood, rclone talks to each service's native API (not a generic protocol), so it can use provider-specific features (server-side copy, chunked uploads, etc.) for speed and reliability.
- For syncing, it compares file **size + modification time** (or checksum, depending on backend) to decide what needs uploading/downloading/deleting — similar to `rsync`, but stateless per run (no persistent delta database by default).
- Supports **encryption**, **compression**, **caching**, **chunking**, and **union/combine** as special "wrapper" remotes that sit on top of real remotes.
- Can **mount** a remote as a local filesystem (via FUSE/WinFsp) so any app can read/write cloud storage like a normal drive.
- Has a built-in **HTTP/WebDAV/FTP/SFTP server mode** to re-serve a remote to other clients.

---

## 3. Supported Services (70+)

Categories and examples:

|Category|Examples|
|---|---|
|Major cloud drives|Google Drive, OneDrive, Dropbox, Box, pCloud, Mega|
|Object storage (S3-compatible)|AWS S3, Backblaze B2, Wasabi, MinIO, Cloudflare R2, DigitalOcean Spaces, Storj|
|Enterprise/other cloud|Azure Blob/Files, Google Cloud Storage, Oracle Object Storage, Alibaba OSS, IBM COS|
|Protocols|SFTP, FTP, WebDAV, SMB/CIFS, HDFS|
|Self-hosted / NAS|Nextcloud, ownCloud, Synology (via WebDAV/SFTP)|
|Local & special|Local disk, in-memory, "crypt" (encryption wrapper), "cache", "union", "combine", "chunker", "compress"|
|Others|Yandex Disk, Mail.ru, Jottacloud, Koofr, Seafile, Zoho WorkDrive, Citrix ShareFile, HiDrive, Uptobox, Premiumize, put.io|

Full current list: `rclone config providers` or the [rclone docs](https://rclone.org/overview/).

---

## 4. Installation

**Linux/macOS (official script):**

```bash
sudo -v ; curl https://rclone.org/install.sh | sudo bash
```

**Windows:** download the zip from rclone.org and add to PATH, or `winget install Rclone.Rclone`.

**Package managers:**

```bash
# Debian/Ubuntu
sudo apt install rclone

# macOS
brew install rclone

# Termux (Android)
pkg install rclone
```

Check version:

```bash
rclone version
```

---

## 5. Setup / Configuration

### 5.1 Interactive config (easiest)

```bash
rclone config
```

Walks you through: name the remote → pick storage type (number from list) → enter client ID/secret (optional, uses rclone's own by default) → authorize (opens browser for OAuth, or gives a link to paste on a headless server) → save.

Config saved at:

- Linux/macOS: `~/.config/rclone/rclone.conf`
- Windows: `%APPDATA%\rclone\rclone.conf`

### 5.2 Headless / remote server setup

Since OAuth needs a browser:

```bash
# On the remote (headless) machine
rclone authorize "drive"
```

Or configure on your local machine, then copy the `rclone.conf` file to the server, or use `rclone config` and paste the auth URL output into a browser on any device, then paste the resulting code back.

### 5.3 Manually via command line (no prompts)

```bash
rclone config create mydrive drive
rclone config create mys3 s3 provider=AWS access_key_id=XXX secret_access_key=YYY region=us-east-1
```

### 5.4 Encrypting the config file (optional, recommended for shared machines)

```bash
rclone config
# choose "Set configuration password"
```

### 5.5 Listing / checking remotes

```bash
rclone listremotes
rclone config show
rclone config dump   # JSON view
```

---

## 6. Useful Commands

### Basic file operations

```bash
rclone ls remote:path            # list files with size
rclone lsd remote:path           # list directories only
rclone lsl remote:path           # list with size, date, time
rclone tree remote:path          # tree view

rclone copy source:path dest:path     # copy (won't delete extra files at dest)
rclone sync source:path dest:path     # make dest identical to source (DELETES extra files at dest!)
rclone move source:path dest:path     # move (deletes from source after copy)
rclone copyto source:file dest:file   # copy single file
rclone moveto source:file dest:file   # move single file

rclone mkdir remote:path/newfolder
rclone rmdir remote:path/folder       # remove empty dir
rclone purge remote:path/folder       # remove dir AND contents (careful!)
rclone delete remote:path             # delete files matching path/filters
rclone deletefile remote:file         # delete a single file
```

### Checking & comparing

```bash
rclone check source:path dest:path    # verify files match (checksum/size)
rclone size remote:path               # total size + file count
rclone dedupe remote:path             # find/handle duplicate files
```

### Sync essentials (safe usage)

```bash
rclone sync source:path dest:path --dry-run     # ALWAYS test first!
rclone sync source:path dest:path -v            # verbose
rclone sync source:path dest:path --progress    # live progress bar
rclone sync source:path dest:path --backup-dir=remote:backup   # keep deleted/changed files instead of losing them
```

### Mounting as a local drive

```bash
rclone mount remote:path /path/to/mountpoint --vfs-cache-mode writes
# Windows example:
rclone mount gdrive: X: --vfs-cache-mode writes
```

### Serving a remote to others

```bash
rclone serve http remote:path --addr :8080
rclone serve webdav remote:path --addr :8081
rclone serve sftp remote:path --addr :2022
```

### Bandwidth & performance tuning

```bash
rclone sync src: dst: --bwlimit 5M              # limit to 5 MB/s
rclone sync src: dst: --transfers 8             # parallel file transfers (default 4)
rclone sync src: dst: --checkers 16             # parallel checks
rclone sync src: dst: --fast-list               # fewer API calls, more memory (good for S3/GCS/B2)
```

### Filtering

```bash
rclone sync src: dst: --exclude "*.tmp"
rclone sync src: dst: --include "*.jpg"
rclone sync src: dst: --max-size 100M
rclone sync src: dst: --min-age 1d
rclone sync src: dst: --filter-from filters.txt
```

### Encryption (crypt remote)

```bash
rclone config   # create a "crypt" remote pointing at an existing remote
rclone copy localfolder cryptedremote:path   # transparently encrypts on upload
```

### Automation

```bash
rclone sync src: dst: --log-file=/var/log/rclone.log --log-level INFO
crontab -e
# 0 * * * * rclone sync /home/user/docs remote:backup/docs --log-file=/home/user/rclone.log
```

### Config management

```bash
rclone config show remotename
rclone config delete remotename
rclone obscure password        # encode a password for use in config files/scripts
```

### Web GUI (built-in, optional)

```bash
rclone rcd --rc-web-gui
```

---

## 7. Common Use Cases

- **Cloud-to-cloud backup**: `rclone sync gdrive:Photos b2:photo-backup`
- **Local backup to cloud**: `rclone sync ~/Documents onedrive:Backups/Documents --backup-dir onedrive:Backups/Old`
- **Mounting Google Drive as a drive letter/folder** for everyday use.
- **Migrating between providers** without downloading to a local machine first (server-side/direct transfer where supported).
- **Encrypted cloud storage** using a crypt remote on top of any provider.
- **Self-hosted NAS ↔ cloud sync** for offsite backup (great for homelab setups).
- **Deduplicating** a messy Drive/Dropbox folder.

---

## 8. Key Flags Cheat Sheet

|Flag|Purpose|
|---|---|
|`--dry-run`|Preview changes without applying|
|`-v` / `-vv`|Verbose / extra verbose logging|
|`--progress`|Live transfer progress|
|`--transfers N`|Parallel transfers|
|`--checkers N`|Parallel file checks|
|`--bwlimit`|Bandwidth cap|
|`--exclude` / `--include`|Filter files|
|`--min-size` / `--max-size`|Size filters|
|`--max-age` / `--min-age`|Date filters|
|`--vfs-cache-mode`|Caching behavior for `mount`|
|`--fast-list`|Reduce API calls (uses more RAM)|
|`--backup-dir`|Preserve overwritten/deleted files|
|`--config`|Use a specific config file path|

---

## 9. Tips & Gotchas

- `sync` is destructive on the destination — always `--dry-run` first.
- `copy` is non-destructive — safer default for backups.
- Rclone doesn't do continuous/real-time sync natively — pair with `cron`/systemd timers, or use `rclone bisync` for two-way sync, or `rclone mount` for live-feel access.
- `rclone bisync` (bidirectional sync) is separate from `sync`/`copy` — needed when both sides can change.
- Rate limits/API quotas vary by provider (Google Drive has daily quota limits) — `--tpslimit` can help avoid bans.
- Config file contains secrets in plaintext unless password-protected — protect it like an SSH key.
- For very large numbers of files, `--fast-list` and `--checkers` tuning significantly affects speed.

---

## 10. Resources

- Official docs: https://rclone.org/docs/
- Provider list: https://rclone.org/overview/
- Forum: https://forum.rclone.org/
- GitHub: https://github.com/rclone/rclone