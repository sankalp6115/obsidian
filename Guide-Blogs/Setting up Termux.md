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
pkg install -y git curl wget nano vim openssh python nodejs clang make cmake pkg-config libffi openssl zip unzip tar python nodejs htop neofetch
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