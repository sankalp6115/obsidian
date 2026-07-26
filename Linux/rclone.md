`rclone` is much more than a Google Drive client—it's essentially a universal cloud filesystem. Below are the commands you'll actually use in practice, grouped by category.

---

# 1. General

Check version

```
rclone version
```

List configured remotes

```
rclone listremotes
```

Show config file location

```
rclone config file
```

Configure remotes

```
rclone config
```

---

# 2. Listing Files

List files with size

```
rclone ls gdrive:
```

List only names

```
rclone lsf gdrive:
```

List directories

```
rclone lsd gdrive:
```

Recursive tree

```
rclone tree gdrive:
```

Long listing

```
rclone lsl gdrive:
```

---

# 3. Information

Storage usage

```
rclone about gdrive:
```

Disk usage

```
rclone size gdrive:
```

Hashes

```
rclone md5sum gdrive:
```

```
rclone sha1sum gdrive:
```

---

# 4. Copying

Local → Drive

```
rclone copy D:\Photos gdrive:Photos
```

Drive → Local

```
rclone copy gdrive:Photos D:\Photos
```

Drive → Drive

```
rclone copy gdrive1:Docs gdrive2:Docs
```

Show progress

```
rclone copy D:\Data gdrive:Backup -P
```

Verbose

```
rclone copy . gdrive:test -vv
```

Dry run

```
rclone copy . gdrive:test --dry-run
```

---

# 5. Sync

Mirror local to cloud

```
rclone sync D:\Projects gdrive:Projects
```

Mirror cloud to local

```
rclone sync gdrive:Projects D:\Projects
```

Dry run

```
rclone sync . gdrive:test --dry-run
```

**Difference**

`copy`

- Never deletes destination files

`sync`

- Makes destination identical to source

---

# 6. Move

Move files

```
rclone move D:\Videos gdrive:Videos
```

Move only empty directories

```
rclone move . gdrive:test --delete-empty-src-dirs
```

---

# 7. Delete

Delete files

```
rclone delete gdrive:Old
```

Delete directory

```
rclone purge gdrive:Old
```

Delete empty dirs

```
rclone rmdirs gdrive:
```

Delete one file

```
rclone deletefile gdrive:test.txt
```

---

# 8. Create Directories

```
rclone mkdir gdrive:Projects
```

---

# 9. Copy Single File

```
rclone copyto report.pdf gdrive:Reports/report.pdf
```

Cloud to local

```
rclone copyto gdrive:Reports/report.pdf report.pdf
```

---

# 10. Move Single File

```
rclone moveto notes.txt gdrive:Notes/notes.txt
```

---

# 11. Rename

```
rclone moveto gdrive:old.txt gdrive:new.txt
```

---

# 12. Mount

Mount as drive

```
rclone mount gdrive: X:
```

Read only

```
rclone mount gdrive: X: --read-only
```

---

# 13. Serve

HTTP server

```
rclone serve http gdrive:
```

WebDAV

```
rclone serve webdav gdrive:
```

FTP

```
rclone serve ftp gdrive:
```

SFTP

```
rclone serve sftp gdrive:
```

---

# 14. Check

Verify local and cloud

```
rclone check D:\Photos gdrive:Photos
```

Download differences

```
rclone check D:\Photos gdrive:Photos --download
```

---

# 15. Crypt

Encrypt filenames

```
rclone cryptcheck encrypted: local:
```

---

# 16. Filters

Include

```
rclone copy . gdrive:Backup --include "*.jpg"
```

Exclude

```
rclone copy . gdrive:Backup --exclude "*.tmp"
```

Only PDFs

```
rclone copy . gdrive:Backup --include "*.pdf"
```

Multiple

```
rclone copy . gdrive:Backup --include "*.jpg" --include "*.png"
```

---

# 17. Transfer Limits

Limit bandwidth

```
rclone copy . gdrive:Backup --bwlimit 5M
```

Transfers

```
rclone copy . gdrive:Backup --transfers 8
```

Checkers

```
rclone copy . gdrive:Backup --checkers 16
```

---

# 18. Logging

Log file

```
rclone sync . gdrive:test --log-file rclone.log
```

Debug

```
rclone sync . gdrive:test -vv
```

Stats

```
rclone sync . gdrive:test --stats 2s
```

---

# 19. Remote Control (RC)

Start API server

```
rclone rcd
```

Useful for automation.

---

# 20. Backend-Specific Commands

Google Drive cleanup

```
rclone cleanup gdrive:
```

Empty trash

```
rclone backend cleanup gdrive:
```

---

# 21. Bisync (Two-Way Sync)

Keep both locations synchronized

```
rclone bisync D:\Notes gdrive:Notes
```

This is much safer than trying to alternate `sync` in both directions. It's ideal if you edit files on both your PC and Drive.

---

# 22. Backend Utilities

Show backend features

```
rclone backend features gdrive:
```

Backend-specific commands

```
rclone backend
```

---

# 23. Useful Flags

|Flag|Purpose|
|---|---|
|`-P`|Progress bar|
|`-v`|Verbose|
|`-vv`|Debug|
|`--dry-run`|Simulate only|
|`--stats=2s`|Update stats every 2 seconds|
|`--checksum`|Compare hashes instead of size/time|
|`--ignore-existing`|Skip existing files|
|`--update`|Copy only newer files|
|`--fast-list`|Faster listing (supported backends)|
|`--transfers=N`|Parallel transfers|
|`--checkers=N`|Parallel checks|
|`--bwlimit=10M`|Bandwidth limit|
|`--max-age=7d`|Files newer than 7 days|
|`--min-age=1h`|Files older than 1 hour|
|`--max-size=100M`|Limit by file size|
|`--min-size=10K`|Ignore tiny files|

---

# 24. My Most-Used Commands

```
rclone ls remote:

rclone tree remote:

rclone copy local remote -P

rclone sync local remote -P

rclone check local remote

rclone about remote:

rclone bisync local remote

rclone mount remote X:

rclone delete remote:path

rclone purge remote:path

rclone mkdir remote:newfolder

rclone copyto file remote:file

rclone moveto file remote:file

rclone lsf remote -R

rclone size remote

rclone config
```

## Tips

- Prefer `copy` unless you specifically want the destination to exactly mirror the source.
- Use `--dry-run` before running `sync`, `move`, `delete`, or `purge` on important data.
- `bisync` is the right choice for keeping a local folder and a cloud folder in sync when both can change.
- `copyto` and `moveto` are handy when you're working with individual files rather than entire directories.
- Add `-P` to long-running transfers to get a live progress display.