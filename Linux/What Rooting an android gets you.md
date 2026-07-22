## 1. Full file system control (not just user storage)

You can read/write **system partitions**:

- Modify `/system`, `/vendor`, `/data`
- Remove preinstalled apps _completely_ (not just disable)
- Replace system files (fonts, boot animation, audio configs)

**Example:**  
Delete a system app like YouTube that normally cannot be uninstalled.

---

## 2. Real system-level ad blocking

Not app-based blockers—**device-wide blocking**:

- Modify `/etc/hosts`
- Block ads in:
    - apps
    - browsers
    - games

Tools like AdAway use this.

---

## 3. CPU, GPU, and battery tuning

You get control over hardware behavior:

- Change CPU governor (performance vs battery)
- Undervolt / overclock
- Control thermal throttling

**Use case:**  
Turn an old phone into:

- a low-power always-on server
- or a high-performance emulator device

---

## 4. Advanced automation (beyond normal apps)

Apps like Tasker become much more powerful:

- Toggle system settings not normally exposed
- Run scripts as root
- Automate deep system actions (e.g., kill background services aggressively)

---

## 5. Full app control (true backups & modification)

With root, apps like Titanium Backup can:

- Backup **app + data + internal state**
- Restore apps exactly as they were
- Freeze apps completely (not just disable)

---

## 6. Network control (serious level)

You can:

- Use firewall apps with full control (like AFWall+)
- Block internet per app at kernel level
- Inspect and manipulate traffic deeply

---

## 7. Custom modules (modular system tweaks)

Using Magisk:

You can install modules that:

- Enable system-wide dark mode tweaks
- Spoof device identity
- Add camera features
- Modify audio engine (Dolby, Viper4Android)

---

## 8. Run Linux-like environments

You can turn your phone into a mini server:

- Run SSH server
- Host web apps (Flask, Node)
- Use tools like `htop`, `tmux`

With tools like Termux + root → near full Linux experience.

---

## 9. Custom ROMs (biggest upgrade potential)

You can replace Android entirely:

- Install LineageOS or others
- Get:
    - newer Android version on old phone
    - no bloatware
    - better privacy

---

## 10. Deep UI & behavior modification

You can:

- Change status bar behavior
- Modify lockscreen
- Add gestures system-wide
- Patch apps (e.g., remove restrictions)

---

# What you _don’t_ magically get

Be realistic:

- No automatic performance boost (you have to tune it)
- No guaranteed battery improvement
- No “hacker-level” abilities unless you build them

---

# When rooting is actually worth it (for your case)

Since you said **spare phone**, here are _high-value uses_:

### 1. Homelab node

- Run backend services (FastAPI, Node)
- Use it as always-on monitoring agent

### 2. Network control device

- Ad-blocking DNS + firewall node
- Traffic analyzer

### 3. Experimentation device

- Learn Android internals
- Break and fix things safely

### 4. Custom ROM testing

- Understand OS-level architecture

---

# Brutal reality check

Rooting is only useful if you **plan to actively use these capabilities**.

If you:

- won’t tweak kernel
- won’t run services
- won’t modify system

→ then it becomes pointless overhead + instability risk.

---

# Bottom line

Root gives you:

> **control over Android as a system, not just as a user**

But control only matters if you _exercise it with intent_.

---

If you want, I can suggest **3 concrete projects** using your rooted spare phone that align with your backend + homelab direction.