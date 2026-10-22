# 🧠 Goal

Run **Jellyfin on Android using Termux + proot (Ubuntu)**

---

# 🧩 PHASE 1 — Initial Approach (Termux Native)

### ✅ Attempt

- Installed `jellyfin-server` directly in Termux

### ❌ Failure

- Error:
    
    ```
    You must install .NET runtimelibhostfxr.so not found
    ```
    

### 🔍 Root Cause

- Termux uses **Android bionic libc**
- Jellyfin requires **glibc + proper .NET runtime**
- `.NET not available / incompatible`

### 📌 Conclusion

❌ **Termux-native Jellyfin = dead end**

---

# 🧩 PHASE 2 — Switch to proot Ubuntu

### ✅ Attempt

- Installed `proot-distro`
- Installed Ubuntu

### ⚠️ Issue

- Got **“resolute” (rolling/unstable Ubuntu)**

### ❌ Failure

- Dependency errors:
    
    ```
    jellyfin-ffmpeg7 depends on libbluray2 (not installable)
    ```
    

### 🔍 Root Cause

- Jellyfin repo targets:
    - Ubuntu 20.04 / 22.04
- You had:
    - bleeding-edge Ubuntu (resolute)

### 📌 Conclusion

❌ **Wrong base distro → dependency hell**

---

# 🧩 PHASE 3 — Fix repo + install manually

### ✅ Attempt

- Added Jellyfin repo manually
- Fixed GPG key issues

### ❌ Failure

- SSL error:
    
    ```
    Certificate verification failedNo system certificates available
    ```
    

### 🔍 Root Cause

- Fresh proot container:
    - `ca-certificates` not initialized

### ✅ Fix

```
apt install ca-certificatesupdate-ca-certificates
```

### 📌 Outcome

✔️ Repo started working

---

# 🧩 PHASE 4 — Install Jellyfin (Ubuntu Jammy)

### ✅ Attempt

- Installed `jellyfin` via apt

### ❌ Failure

- Runtime crash:
    
    ```
    Segmentation fault
    ```
    

### 🔍 Root Cause

- Jellyfin = .NET + native libs
- proot = partial syscall emulation
- Android kernel ≠ full Linux behavior

👉 **Binary runs but crashes at runtime**

### 📌 Conclusion

⚠️ **Install success ≠ runtime compatibility**

---

# 🧩 PHASE 5 — Try Portable Binary

### ❌ Mistake

- Downloaded **source code repo instead of binary**

### Symptom

```
./jellyfin → No such file
```

### 🔍 Root Cause

- GitHub repo ≠ compiled release

### 📌 Conclusion

❌ Wrong artifact (source, not executable)

---

# 🧩 PHASE 6 — Try .deb manually

### ❌ Failure

```
not a Debian format archive
```

### 🔍 Root Cause

- Downloaded **corrupted / HTML file instead of package**

### 📌 Conclusion

❌ Invalid file, not usable

---

# 🧩 PHASE 7 — Correct Portable Attempt

### ✅ Attempt

- Download proper `.tar.gz` portable binary

### ❌ Failure

- Still segfault (or expected to)

### 🔍 Root Cause

Same as before:

- proot + .NET incompatibility

---

# ⚠️ CORE PROBLEMS (IMPORTANT)

## 1. Environment mismatch

- Android ≠ Linux
- proot ≠ real kernel

---

## 2. Jellyfin tech stack

- Heavy:
    - .NET runtime
    - ffmpeg
    - native bindings

👉 Not tolerant to partial environments

---

## 3. proot limitations

- syscall emulation incomplete
- memory handling differences
- causes **segfaults in complex apps**

---

# 🟡 WHAT ALMOST WORKED

### ✔️ Best partial success

- Ubuntu Jammy in proot
- Jellyfin installed correctly
- repo + certs fixed

👉 Only remaining issue:  
❌ runtime segfault

---

# 🟢 WHAT WOULD WORK (GUARANTEED)

## Option A — Proper machine

- Old laptop / mini PC
- Install:
    - Ubuntu / Debian
- Jellyfin works perfectly

---

## Option B — Lightweight alternative

- Navidrome

Why:

- Written in Go
- No .NET
- Works in Termux/proot

---

## Option C — Android-native apps

- Jellyfin Android client
- Server hosted elsewhere

---

# 🧠 FINAL TAKEAWAYS (MOST IMPORTANT)

### 1. Installation ≠ Execution

You successfully installed Jellyfin  
But runtime failed → deeper issue

---

### 2. Always validate environment early

Before debugging:

- check compatibility matrix
- avoid wasting time

---

### 3. Complex stacks need real OS

Anything involving:

- .NET
- system services
- media transcoding

👉 **avoid proot**

---

### 4. Debug pattern (you learned this)

You went through:

- dependency issues
- repo issues
- cert issues
- runtime issues

That’s actually full-stack debugging

---

# 🧾 TL;DR (quick log version)

- ❌ Termux native → .NET missing
- ❌ Ubuntu resolute → dependency failure
- ⚠️ Fixed repo + certs → install works
- ❌ Runtime → segmentation fault
- ❌ Wrong downloads (.deb/source)
- ❌ Portable binary → still crashes

👉 Root issue: **proot + Jellyfin incompatibility**



# Jellyfin on Android (Termux + proot) — Complete Debug & Fix Log

> **Purpose:** Full reference log of every attempt, failure, fix, and final working solution.  
> **Device:** Redmi (ARM64 / aarch64)  
> **Goal:** Run Jellyfin media server on Android without root  
> **Final Result:** ✅ Working — Debian trixie (proot) + env var fix

---

## 🧠 Core Concepts (Read First)

| Term                         | What it means                                                      |
| ---------------------------- | ------------------------------------------------------------------ |
| Termux                       | Android terminal emulator, runs in Android userspace               |
| proot                        | Userspace chroot — fakes a Linux root filesystem without real root |
| proot-distro                 | Termux tool to install full Linux distros via proot                |
| bionic libc                  | Android's C library — incompatible with most Linux binaries        |
| glibc                        | GNU C library — what Linux distros and Jellyfin require            |
| .NET runtime                 | What Jellyfin is built on — needs glibc + specific syscalls        |
| DOTNET_EnableWriteXorExecute | .NET security flag — must be disabled on Android kernels           |

---

## ❌ PHASE 1 — Termux Native Install

### What we tried

```
pkg install jellyfin-server
```

### What happened

```
You must install .NET runtime
libhostfxr.so not found
```

### Root cause

- Termux runs on Android's **bionic libc**
- Jellyfin needs **glibc** (GNU C library)
- .NET runtime is not available natively in Termux
- Even if installed, binaries compiled against glibc won't run in bionic

### Conclusion

❌ **Dead end.** Termux-native Jellyfin is fundamentally impossible without proot.

---

## ❌ PHASE 2 — proot Ubuntu (Resolute / 26.04)

### What we tried

```
pkg install proot-distro
proot-distro install ubuntu
proot-distro login ubuntu
apt install jellyfin
```

### What happened

```
jellyfin-ffmpeg7 depends on libbluray2 but it is not installable
```

### Root cause

- `proot-distro install ubuntu` pulled **Ubuntu 26.04 "Resolute Raccoon"** (bleeding edge, just released June 2026)
- Jellyfin's repo targets Ubuntu 20.04 / 22.04 (jammy) / 24.04 (noble)
- `libbluray2` was replaced by `libbluray3` / `libbluray2t64` in 26.04
- No Jellyfin build exists for resolute

### Things tried to fix it

```bash
# Tried adding bookworm repo as fallback
echo "deb [signed-by=...] https://repo.jellyfin.org/ubuntu noble main" | tee /etc/apt/sources.list.d/jellyfin.list

# Tried installing missing lib directly
apt install libbluray2 -y
# → "Package has no installation candidate"

apt install libbluray3 -y
apt install libbluray2t64 -y
# → Neither resolved the jellyfin-ffmpeg7 dependency chain

# Tried equivs fake package
apt install equivs -y
equivs-build libbluray2-fake.control
dpkg -i libbluray2_99.0_all.deb
# → Still failed on other missing deps (libvpx7, libx265-199, libicu72)
```

### Conclusion

❌ **Ubuntu 26.04 = dependency hell.** Too new, Jellyfin has no support for it.  
**Key lesson:** Always verify distro version before starting — `cat /etc/os-release`

---

## ❌ PHASE 3 — proot Debian Trixie (13) — First Install Attempt

### What we tried

```
proot-distro install debian
proot-distro login debian
# Got Debian 13 "trixie"

curl -fsSL https://repo.jellyfin.org/debian/jellyfin_team.gpg.key | gpg --dearmor | tee /usr/share/keyrings/jellyfin-archive-keyring.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/jellyfin-archive-keyring.gpg] https://repo.jellyfin.org/debian bookworm main" | tee /etc/apt/sources.list.d/jellyfin.list
apt update
apt install jellyfin -y
```

### What happened

```
jellyfin-ffmpeg7 : Depends: libvpx7 (>= 1.12.0) but it is not installable
                   Depends: libx265-199 (>= 3.5) but it is not installable
jellyfin-server  : Depends: libicu72 but it is not installable
```

### Root cause

- Jellyfin packages are built for **Debian 12 (bookworm)**
- Trixie (13) replaced `libvpx7` → `libvpx9`, `libicu72` → `libicu76`, `libx265-199` → newer ABI
- These old library versions simply don't exist in trixie's repos

### Fix that worked

```bash
# Added bookworm as an additional source just for the missing libs
echo "deb http://deb.debian.org/debian bookworm main" | tee /etc/apt/sources.list.d/bookworm.list
apt update
apt install libvpx7 libx265-199 libicu72 -y
# ✅ These installed successfully from bookworm repo
apt install jellyfin -y
# ✅ Installed successfully — 97.5 MB download
```

### Packages installed

```
jellyfin-ffmpeg7  jellyfin-server  jellyfin-web
libjemalloc2  libtheora0  sudo
```

### Conclusion

✅ **Install succeeded** by mixing trixie + bookworm repos for missing libs.  
**Key lesson:** You can pull specific packages from older Debian releases using additional apt sources.

---

## ❌ PHASE 4 — Segmentation Fault on Launch

### What we tried

```bash
jellyfin
# → Segmentation fault

./jellyfin.sh   # (startup script with memory limits)
# → Segmentation fault

./jellyfin.sh   # (tried twice)
# → Segmentation fault
```

### What happened

Jellyfin installed fine but crashed immediately on every launch attempt.

### Root cause

- Android kernels restrict **W^X (Write XOR Execute)** memory pages as a security policy
- .NET runtime requires the ability to JIT-compile — it needs to write to memory pages and then execute them
- By default `.NET` has `DOTNET_EnableWriteXorExecute=1` (enforced)
- On Android kernels this causes an immediate segfault when .NET tries to JIT

### Things tried that did NOT fix it

```bash
# Script with memory limits only — still segfaulted
#!/bin/sh
export DOTNET_GC_SERVER=0
export DOTNET_GC_CONCURRENT=1
export DOTNET_GCHeapHardLimit=800000000
jellyfin --webdir /usr/share/jellyfin/web --nonetchange
# → Segmentation fault
```

### The fix

```bash
export DOTNET_EnableWriteXorExecute=0
export DOTNET_GC_SERVER=0
jellyfin --nonetchange --webdir /usr/share/jellyfin/web 2>&1 | head -50
# ✅ Jellyfin started successfully
```

**`DOTNET_EnableWriteXorExecute=0`** is the critical flag — disables the W^X enforcement so .NET can JIT on Android's kernel.

### Conclusion

✅ **Segfault fixed.** This flag is **mandatory** for running any .NET app on Android/proot.  
**Key lesson:** Always add `DOTNET_EnableWriteXorExecute=0` for .NET apps on Android kernels.

---

## ✅ FINAL WORKING SOLUTION

### Environment

|Item|Value|
|---|---|
|Device|Redmi (ARM64)|
|Termux|Latest from F-Droid|
|proot distro|Debian 13 (trixie)|
|Extra apt source|Debian 12 (bookworm) for missing libs|
|Jellyfin version|10.11.11+deb12|

### Complete install commands (from scratch)

**In Termux:**

```bash
pkg update -y && pkg upgrade -y
pkg install proot-distro -y
proot-distro install debian
proot-distro login debian
```

**Inside Debian proot:**

```bash
# Update
apt update && apt upgrade -y

# Dependencies
apt install curl gnupg apt-transport-https ca-certificates ffmpeg -y

# Add Jellyfin GPG key
curl -fsSL https://repo.jellyfin.org/debian/jellyfin_team.gpg.key | gpg --dearmor | tee /usr/share/keyrings/jellyfin-archive-keyring.gpg > /dev/null

# Add Jellyfin repo
echo "deb [signed-by=/usr/share/keyrings/jellyfin-archive-keyring.gpg] https://repo.jellyfin.org/debian bookworm main" | tee /etc/apt/sources.list.d/jellyfin.list

# Add bookworm as fallback for missing libs (needed on trixie)
echo "deb http://deb.debian.org/debian bookworm main" | tee /etc/apt/sources.list.d/bookworm.list

apt update

# Install missing libs from bookworm first
apt install libvpx7 libx265-199 libicu72 -y

# Install Jellyfin
apt install jellyfin -y
```

### Startup script — jellyfin.sh

```bash
nano jellyfin.sh
```

**4 GB RAM:**

```sh
#!/bin/sh
export DOTNET_EnableWriteXorExecute=0
export DOTNET_GC_SERVER=0
export DOTNET_GC_CONCURRENT=1
export DOTNET_GCHeapHardLimit=800000000
export DOTNET_GCHeapHardLimitPercent=50
jellyfin \
  --webdir /usr/share/jellyfin/web \
  -c "$HOME/.config/jellyfin" \
  -C "$HOME/.cache/jellyfin" \
  -d "$HOME/.local/share/jellyfin" \
  -l "$HOME/.local/share/jellyfin/log" \
  --nonetchange
```

**6 GB RAM:**

```sh
#!/bin/sh
export DOTNET_EnableWriteXorExecute=0
export DOTNET_GC_SERVER=0
export DOTNET_GC_CONCURRENT=1
export DOTNET_GCHeapHardLimit=1200000000
export DOTNET_GCHeapHardLimitPercent=50
jellyfin \
  --webdir /usr/share/jellyfin/web \
  -c "$HOME/.config/jellyfin" \
  -C "$HOME/.cache/jellyfin" \
  -d "$HOME/.local/share/jellyfin" \
  -l "$HOME/.local/share/jellyfin/log" \
  --nonetchange
```

**8 GB RAM:**

```sh
#!/bin/sh
export DOTNET_EnableWriteXorExecute=0
export DOTNET_GC_SERVER=0
export DOTNET_GC_CONCURRENT=1
export DOTNET_GCHeapHardLimit=2000000000
export DOTNET_GCHeapHardLimitPercent=60
jellyfin \
  --webdir /usr/share/jellyfin/web \
  -c "$HOME/.config/jellyfin" \
  -C "$HOME/.cache/jellyfin" \
  -d "$HOME/.local/share/jellyfin" \
  -l "$HOME/.local/share/jellyfin/log" \
  --nonetchange
```

```bash
chmod +x jellyfin.sh
./jellyfin.sh

```

### Access

```
http://localhost:8096
http://<phone-mdns-name>:8096   # e.g. http://redmi:8096
```

### Every time you want to start Jellyfin

```bash
# In Termux:
proot-distro login debian

# Inside Debian:
./jellyfin.sh
```

---

## 🔑 Key Flags Explained

|Flag|Why it's needed|
|---|---|
|`DOTNET_EnableWriteXorExecute=0`|**Most critical.** Disables W^X enforcement so .NET JIT works on Android kernel|
|`DOTNET_GC_SERVER=0`|Disables server GC mode — not suitable for single-process mobile environment|
|`DOTNET_GC_CONCURRENT=1`|Enables concurrent GC — reduces pause times on limited RAM|
|`DOTNET_GCHeapHardLimit`|Caps .NET heap — prevents Jellyfin from consuming all RAM and getting OOM-killed|
|`DOTNET_GCHeapHardLimitPercent`|Secondary RAM cap as percentage of total|
|`--nonetchange`|Tells Jellyfin not to touch network interface config — required in proot|
|`--webdir`|Explicit path to web UI files — needed because systemd isn't running to set defaults|

---

## ⚠️ Known Limitations

|Limitation|Detail|
|---|---|
|No hardware transcoding|proot can't access Android MediaCodec — CPU only|
|No systemd|Services don't auto-start — must run `./jellyfin.sh` manually every time|
|Wakelock required|Android will kill Termux in background — enable wakelock from Termux notification|
|Transcoding is slow|For smooth playback use **direct play** — access via browser, not the Android app|
|App vs browser|Jellyfin Android app may force transcoding — use Chrome at `localhost:8096` instead|

---

## 📚 Lessons Learned

1. **Always check `cat /etc/os-release` before doing anything** — distro version determines everything
2. **Installation success ≠ runtime success** — especially with .NET on non-standard kernels
3. **`DOTNET_EnableWriteXorExecute=0` is mandatory** for any .NET app on Android/proot
4. **Pipe crash output to see real errors:** `command 2>&1 | head -50`
5. **Mix apt sources carefully** — pulling bookworm libs into trixie works but can cause conflicts if overdone
6. **`--nonetchange` is required** in proot — without it Jellyfin tries to modify network interfaces and fails
7. **Use browser over app** for localhost self-hosted Jellyfin on Android

---

## 🗂️ Final Setup Summary

```
Android (Redmi)
└── Termux
    ├── proot-distro
    │   └── Debian trixie (13)
    │       ├── apt sources: trixie + bookworm (for compat libs)
    │       └── Jellyfin 10.11.11
    │           └── Started via jellyfin.sh (DOTNET_EnableWriteXorExecute=0)
    └── Melodious (own project)
        ├── FastAPI backend
        └── React frontend
```

Access Jellyfin at: `http://localhost:8096` or `http://redmi:8096`