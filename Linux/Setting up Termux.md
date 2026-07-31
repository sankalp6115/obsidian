# 1. Install Termux (important)

Do **not** use Play Store version (outdated).
Install from:
-   F-Droid
-   Or GitHub (official Termux releases)

------

# 2. First startup (basic update)

Open Termux and run:

```
pkg update && pkg upgrade -y
```

Then allow storage access:

```
termux-setup-storage
```

# 3. Install essential packages
These are the core tools:

```
pkg install -y git curl wget nano vim openssh python nodejs clang make cmake pkg-config libffi openssl zip unzip tar htop neofetch fastfetch
```

# 3.1 Tur Repo

Gives extra packages
```bash
pkg install tur-repo
pkg install python3.11
alias python=python3.11
alias pip='python3.11 -m pip'
```

# 4. Fix common issues (recommended)
Set proper environment:

```
echo 'export EDITOR=nano' >> ~/.bashrc
echo 'alias ll="ls -la"' >> ~/.bashrc
source ~/.bashrc
```

# 5. Setup Python environment

```
pip install --upgrade pip virtualenv
```

Some Libraries with versions:
```txt
fastapi==0.95.2
pydantic==1.10.24
uvicorn==0.22.0
python-multipart
mutagen
```

# 6. Setup Node.js (optional dev work)

```
npm install -g npm yarn pnpm
```

# 7. Setup SSH (remote access)

Start SSH server:
```
pkg install openssh
sshd
```

Check IP:

```
ip a
```

**Termux runs ssh on port 8022.**
Connect from another device:
```
ssh username@your_ip -p 8022
```

# 8. Customize terminal (looks + usability)

Install zsh + oh-my-zsh:
```
pkg install zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Optional fonts (for better UI):
-   Install a Nerd Font in your terminal app

# 9. Useful extra tools

### Networking
```
pkg install nmap hydra tcpdump netcat
pip install scapy
```

### Development
```
pkg install golang rust
pkg install fzf ripgrep bat htop tmux
```

### File management
```
pkg install ranger
```

# 10. (Optional) Proot Linux (full Linux inside Termux)

If you want Ubuntu/Kali:
```
pkg install proot-distro
proot-distro install ubuntu
proot-distro login ubuntu
```

# 11. Keep Termux clean
Update regularly:

```
pkg update && pkg upgrade
```

Clean cache:
```
pkg autoclean
```

# 12. Termux SSH Notification
```bash
#!/data/data/com.termux/files/usr/bin/bash
termux-media-player play "$HOME/system_scripts/login.mp3"
termux-toast "SSH Login Detected"
if [ -n "$SSH_CLIENT" ]; then
    set -- $SSH_CLIENT
    client="$1:$2"
    echo "$client"
    termux-notification --title "SSH Login" --content "$client SSHed"
fi
exec "$SHELL" -l
```

Whenever someone sshes into server , the server will send notification showing client ip and port, give toast, and play an alert sound.


```txt
Tried starting homelab in android downloads folder, tried to run a shell file, but it stopped me to run the file bcz of android access problems. So dont try doing it.

- ❌ Android storage permissions are more restrictive.
- ❌ Many Linux features don't work properly (permissions, symlinks, executables).
- ❌ Every app has to go through Termux's storage bridge.
- ❌ Slower I/O in many cases.
```


### 13. cron job to make consistent backup periodically
### 14. Change phone desktop wallpaper using termux-wallpaper

### 15.Custom Script to setup a recycle bin in termux
# Custom Trash Function
trash() {
    # Path to your local trash folder
    local TRASH_DIR="$HOME/.local/share/Trash"
    
    # Ensure the trash folder exists every time you delete
    mkdir -p "$TRASH_DIR"

    # If no files are specified, show usage
    if [ $# -eq 0 ]; then
        echo "Usage: trash <file_or_directory>"
        return 1
    fi

    # Move files to the trash folder
    mv -f "$@" "$TRASH_DIR/"
    echo "Moved to trash: $@"
}

# Empty trash command
empty-trash() {
    local TRASH_DIR="$HOME/.local/share/Trash"
    if [ -d "$TRASH_DIR" ]; then
        rm -rf "$TRASH_DIR"/*
        echo "Trash emptied."
    else
        echo "Trash is already empty."
    fi
}



Programs
cowsay
neofetch
fastfetch
fortune
cava

