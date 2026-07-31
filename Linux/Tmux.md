## Table of Contents

1. [Core Concepts](https://claude.ai/chat/5093a5c2-aac7-4cf9-9cc0-984a40b9063b#core-concepts)
2. [Installation](https://claude.ai/chat/5093a5c2-aac7-4cf9-9cc0-984a40b9063b#installation)
3. [Basic Operations](https://claude.ai/chat/5093a5c2-aac7-4cf9-9cc0-984a40b9063b#basic-operations)
4. [Sessions](https://claude.ai/chat/5093a5c2-aac7-4cf9-9cc0-984a40b9063b#sessions)
5. [Windows](https://claude.ai/chat/5093a5c2-aac7-4cf9-9cc0-984a40b9063b#windows)
6. [Panes](https://claude.ai/chat/5093a5c2-aac7-4cf9-9cc0-984a40b9063b#panes)
7. [Modes](https://claude.ai/chat/5093a5c2-aac7-4cf9-9cc0-984a40b9063b#modes)
8. [Configuration](https://claude.ai/chat/5093a5c2-aac7-4cf9-9cc0-984a40b9063b#configuration)
9. [Advanced Features](https://claude.ai/chat/5093a5c2-aac7-4cf9-9cc0-984a40b9063b#advanced-features)
10. [Workflows](https://claude.ai/chat/5093a5c2-aac7-4cf9-9cc0-984a40b9063b#workflows)
11. [Shortcuts Cheat Sheet](https://claude.ai/chat/5093a5c2-aac7-4cf9-9cc0-984a40b9063b#shortcuts-cheat-sheet)

---

## Core Concepts

### What is tmux?

A **terminal multiplexer** that acts as a window manager for your terminal. It allows you to:

- Keep terminal sessions alive even after disconnecting SSH
- Split terminal into multiple panes
- Manage multiple windows within sessions
- Switch between projects without losing progress
- Work on remote servers without losing data on disconnect

### tmux Architecture

```
Terminal
   │
 tmux
   ├── Session A
   │    ├── Window 1
   │    ├── Window 2
   │    └── Window 3
   └── Session B
        ├── Window 1
        └── Window 2
```

### Key Benefits

- Sessions persist even if terminal closes or SSH disconnects
- Access multiple projects simultaneously
- Organize work with windows and panes
- Remote development becomes seamless
- Share terminal sessions with others

---

## Installation

### Ubuntu/Debian

```bash
sudo apt install tmux
```

### Arch Linux

```bash
sudo pacman -S tmux
```

### macOS

```bash
brew install tmux
```

### Verify Installation

```bash
tmux -V
```

---

## Basic Operations

### The Prefix Key

Almost every tmux shortcut begins with a **prefix** key.

**Default Prefix:** `Ctrl+b`

**How to read shortcuts:**

- `Prefix + c` means: Press `Ctrl+b`, release, then press `c`

### Create Your First Session

```bash
# Start tmux with default name
tmux

# Or explicitly create new session
tmux new

# Create named session
tmux new -s work
```

### Detach from Session

```bash
# Keep session running in background
Prefix + d
```

After detaching, you're back to your regular shell.

### Reattach to Session

```bash
# Reattach to most recent session
tmux attach

# Attach to specific session
tmux attach -t work
```

---

## Sessions

### Session Management Commands

#### Create Session

```bash
tmux new -s backend          # Create named session
tmux new -d -s build         # Create detached session
```

#### List Sessions

```bash
tmux ls
```

**Example output:**

```
backend
frontend
notes
```

#### Attach to Session

```bash
tmux attach -t backend
```

#### Rename Session

```bash
tmux rename-session -t old new
```

#### Kill Session

```bash
tmux kill-session -t backend      # Kill specific session
tmux kill-server                  # Kill all sessions
```

### Session Shortcuts

|Action|Shortcut|
|---|---|
|List sessions (interactive)|`Prefix + s`|
|Detach session|`Prefix + d`|
|Switch to session|`tmux switch -t backend`|

---

## Windows

Windows are like **terminal tabs** within a session.

### Window Management

#### Create Window

```bash
Prefix + c
```

#### Navigate Windows

```bash
Prefix + n          # Next window
Prefix + p          # Previous window
Prefix + w          # Choose window (interactive list)
```

#### Window List (Interactive)

```bash
Prefix + w
```

Displays all windows with numbers. Press window number to switch.

#### Rename Window

```bash
Prefix + ,
# Then type new name and press Enter
```

#### Kill Window

```bash
exit                # Exit the shell in window (kills window)
Prefix + &          # Force kill window (prompts for confirmation)
```

### Window Numbering

By default, windows are numbered starting at 0. You can configure to start at 1:

```bash
# In ~/.tmux.conf
set -g base-index 1
```

---

## Panes

Panes split a window into multiple areas, allowing you to view multiple sections of code/output simultaneously.

### Creating Panes

#### Vertical Split (side by side)

```bash
Prefix + %
```

#### Horizontal Split (top and bottom)

```bash
Prefix + "
```

### Pane Layout Example

```
+---------------------+
|        |            |
| vim    | logs       |
|        |            |
+--------+------------+
| shell               |
+---------------------+
```

### Navigating Between Panes

```bash
Prefix + ↑          # Move to pane above
Prefix + ↓          # Move to pane below
Prefix + ←          # Move to pane left
Prefix + →          # Move to pane right
```

### Resizing Panes

#### Method 1: Hold Prefix + Ctrl + Arrow

```bash
Prefix + Ctrl + ↑   # Increase height
Prefix + Ctrl + ↓   # Decrease height
Prefix + Ctrl + ←   # Decrease width
Prefix + Ctrl + →   # Increase width
```

#### Method 2: Hold Prefix + Alt + Arrow

```bash
Prefix + Alt + ↑
Prefix + Alt + ↓
Prefix + Alt + ←
Prefix + Alt + →
```

### Pane Layouts

#### Cycle Through Layouts

```bash
Prefix + Space
```

Automatically arranges panes in different layouts. Useful when dealing with many panes.

#### Common Layouts

- Even Horizontal
- Even Vertical
- Main Horizontal
- Main Vertical
- Tiled

### Zooming Panes

```bash
Prefix + z
```

Expand current pane to fill window. Press again to restore.

**Very useful for focusing on one pane temporarily.**

### Closing Panes

```bash
exit                # Exit shell in pane (kills pane)
Prefix + x          # Force kill pane (prompts for confirmation)
```

### Pane Information

```bash
tmux list-panes     # List all panes
```

---

## Modes

### Copy Mode

Used for scrolling and selecting text in tmux (since regular scroll doesn't work).

#### Enter Copy Mode

```bash
Prefix + [
```

#### Navigation in Copy Mode

```bash
↑↓←→              # Arrow keys to navigate
Page Up             # Scroll up
Page Down           # Scroll down
b                   # Jump word backward
w                   # Jump word forward
{                   # Jump paragraph backward
}                   # Jump paragraph forward
Ctrl + Home         # Go to top
Ctrl + End          # Go to bottom
```

#### Search in Copy Mode

```bash
/                   # Search forward
?                   # Search backward
n                   # Next result
N                   # Previous result
```

#### Exit Copy Mode

```bash
q                   # Quit copy mode
Escape              # Also exits copy mode
Enter               # Select and copy, then exit
```

#### Selecting Text

```bash
Space               # Start selection
Enter               # End selection and copy
```

### Paste Mode

#### List Buffers

```bash
Prefix + #
```

#### Paste from Buffer

```bash
Prefix + ]          # Paste last copied buffer
```

#### Advanced Buffer Management

```bash
tmux list-buffers   # List all buffers
tmux send-keys -t build "npm run build" Enter  # Send command to specific session
```

---

## Configuration

### Configuration File Location

```bash
~/.tmux.conf
```

### Basic Configuration Example

```bash
# Change prefix key
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# Mouse support
set -g mouse on

# History
set -g history-limit 100000

# Vi keybindings
setw -g mode-keys vi

# Window and pane numbering
set -g base-index 1
setw -g pane-base-index 1

# Auto-rename windows
set -g automatic-rename on

# Status bar
set -g status-position bottom
set -g status-bg black
set -g status-fg white
set -g status-right "%Y-%m-%d %H:%M"
```

### Applying Configuration

#### Reload Configuration

```bash
# Method 1: Reload from file
tmux source ~/.tmux.conf

# Method 2: Use command prompt
Prefix + :
source-file ~/.tmux.conf
```

### Common Configuration Options

#### Change Prefix Key

```bash
unbind C-b              # Unbind default
set -g prefix C-a       # Set new prefix
bind C-a send-prefix    # Send prefix to inner tmux
```

#### Mouse Support

```bash
set -g mouse on
```

Enables:

- Click to switch panes
- Drag to resize panes
- Scroll with mouse wheel
- Select text with mouse

#### Vi Key Bindings

```bash
setw -g mode-keys vi
```

#### Auto-Rename Windows

```bash
set -g automatic-rename on
```

Windows automatically rename based on running program.

#### Window Renumbering

```bash
set -g renumber-windows on
```

When a window closes, remaining windows renumber sequentially.

#### Status Bar Customization

```bash
# Position
set -g status-position top        # or bottom

# Colors
set -g status-bg black
set -g status-fg white

# Right-aligned content
set -g status-right "%Y-%m-%d %H:%M"

# Left-aligned content
set -g status-left "Session: #S"
```

### Recommended Complete ~/.tmux.conf

```bash
# === Prefix ===
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# === General ===
set -g mouse on
set -g history-limit 100000
setw -g mode-keys vi
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on

# === Status Bar ===
set -g status-position bottom
set -g status-right "%Y-%m-%d %H:%M"

# === Behavior ===
set -g automatic-rename on
set -g focus-events on
```

---

## Advanced Features

### Mouse Support

```bash
# Enable in config
set -g mouse on
```

**What you can do:**

- Click to switch panes
- Click to select windows
- Drag pane dividers to resize
- Scroll with mouse wheel
- Select text by dragging

### Synchronize Panes

Run the same command in multiple panes simultaneously.

#### Enable Synchronize

```bash
Prefix + :
setw synchronize-panes on
```

#### Disable Synchronize

```bash
setw synchronize-panes off
```

**Use case:** Running commands on multiple servers at once.

### Nested tmux

When SSH'd into another machine running tmux:

#### Send Prefix to Inner tmux

```bash
Prefix + Prefix
```

Example scenario:

```
Local laptop tmux
    ↓ SSH
Server tmux
```

To send command to server tmux, press Prefix twice.

### Command Prompt

```bash
Prefix + :
```

Provides command line to type tmux commands directly. Very powerful for advanced operations.

### Plugins with TPM (Tmux Plugin Manager)

#### Install TPM

```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

#### Add to ~/.tmux.conf

```bash
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'
set -g @plugin 'catppuccin/tmux'

run '~/.tmux/plugins/tpm/tpm'
```

#### Popular Plugins

|Plugin|Purpose|
|---|---|
|`tmux-resurrect`|Restore sessions after reboot|
|`tmux-continuum`|Auto-save and auto-restore sessions|
|`tmux-yank`|Better clipboard support|
|`catppuccin/tmux`|Popular theme|
|`tmux-prefix-highlight`|Visual prefix state indicator|
|`tmux-pain-control`|Better pane navigation|

#### Install/Update Plugins

```bash
Prefix + I       # Install new plugins
Prefix + U       # Update plugins
```

### Scripting Sessions

Create entire session layouts from scripts:

```bash
#!/bin/bash

# Create session
tmux new-session -d -s project

# Create windows
tmux new-window -t project -n editor
tmux new-window -t project -n server
tmux new-window -t project -n logs

# Split and run commands
tmux send-keys -t project:editor "nvim" Enter
tmux send-keys -t project:server "python manage.py runserver" Enter
tmux send-keys -t project:logs "tail -f app.log" Enter

# Attach
tmux attach -t project
```

### Useful Advanced Commands

#### Send Command to Session

```bash
tmux send-keys -t session "command" Enter
```

#### Capture Pane Output

```bash
tmux capture-pane -p
tmux capture-pane -p -t session:window.pane > output.txt
```

#### List All Panes

```bash
tmux list-panes
```

#### List All Windows

```bash
tmux list-windows
```

---

## Workflows

### Web Development Workflow

Session: `project`

```
Window 1: Editor
  ├── Pane 1: nvim (editor)
  └── Pane 2: git

Window 2: Backend
  └── python manage.py runserver

Window 3: Frontend
  └── npm run dev

Window 4: Logs
  └── tail -f app.log
```

**Everything stays running even if you disconnect.**

### SSH Workflow (Reliable Remote Development)

**Local machine:**

```bash
ssh myserver
```

**On remote server:**

```bash
tmux attach || tmux new -s main
```

**If Wi-Fi disconnects:**
1. Reconnect to Wi-Fi
2. SSH to server again
3. `tmux attach`
4. Everything is exactly where you left it!
### Multi-Server Management

Enable pane synchronization to run commands on multiple servers:

```bash
# Create multiple panes connecting to different servers
Prefix + %
ssh server1

# Then in new pane
ssh server2

# Enable sync
Prefix + :
setw synchronize-panes on

# Now commands run on both servers
```

---

## Shortcuts Cheat Sheet

### Essential Shortcuts

|Action|Shortcut|
|---|---|
|**Prefix**|`Ctrl+b` (or `Ctrl+a` if remapped)|
|**Detach**|`Prefix + d`|
|**Enter command**|`Prefix + :`|

### Sessions

|Action|Shortcut|
|---|---|
|List sessions|`Prefix + s`|
|Rename session|`tmux rename-session -t old new`|
|Kill session|`tmux kill-session -t name`|

### Windows

|Action|Shortcut|
|---|---|
|New window|`Prefix + c`|
|Next window|`Prefix + n`|
|Previous window|`Prefix + p`|
|List windows|`Prefix + w`|
|Rename window|`Prefix + ,`|
|Kill window|`Prefix + &` or `exit`|

### Panes

|Action|Shortcut|
|---|---|
|Split vertical|`Prefix + %`|
|Split horizontal|`Prefix + "`|
|Move to pane|`Prefix + ↑↓←→`|
|Zoom pane|`Prefix + z`|
|Kill pane|`exit` or `Prefix + x`|
|Cycle layouts|`Prefix + Space`|
|Resize pane|`Prefix + Ctrl + ↑↓←→`|

### Copy Mode

|Action|Shortcut|
|---|---|
|Enter copy mode|`Prefix + [`|
|Navigate|Arrow keys, PgUp, PgDn|
|Search forward|`/` then type|
|Search backward|`?` then type|
|Next result|`n`|
|Previous result|`N`|
|Exit|`q` or `Escape`|
|Copy selection|`Space` (start) + `Enter` (end)|
|Paste|`Prefix + ]`|

### Advanced

|Action|Shortcut|
|---|---|
|Open command prompt|`Prefix + :`|
|List buffers|`Prefix + #`|
|Send to inner tmux|`Prefix + Prefix`|

---

## Command Reference

### Session Commands

```bash
tmux new -s name              # Create named session
tmux new -d -s name           # Create detached session
tmux attach -t name           # Attach to session
tmux ls                        # List all sessions
tmux kill-session -t name      # Kill session
tmux kill-server              # Kill all sessions
tmux rename-session -t old new # Rename session
```

### Window Commands

```bash
tmux new-window -t session -n name     # Create window with name
tmux list-windows                       # List all windows
tmux kill-window -t session:n           # Kill window
```

### Pane Commands

```bash
tmux split-window -h                    # Horizontal split
tmux split-window -v                    # Vertical split
tmux list-panes                         # List all panes
tmux send-keys -t pane "cmd" Enter      # Send command to pane
tmux capture-pane -p                    # Capture pane output
```

### Configuration Commands

```bash
tmux source ~/.tmux.conf               # Reload config
tmux show-options -g                   # Show global options
tmux show-window-options                # Show window options
```

---

## Tips for Productivity

### 1. Organize by Project

Create one session per project with windows for different tasks.

### 2. Use Session Scripting

Create shell scripts that set up your entire workspace.

### 3. Enable Mouse

Makes navigation and resizing much faster.

### 4. Use Plugins

`tmux-resurrect` and `tmux-continuum` preserve your sessions.

### 5. Learn Copy Mode Well

It's essential for reading logs and selecting output without mouse.

### 6. Rename Windows

Keep window names descriptive for quick navigation.

### 7. Keep Sessions Alive

SSH into server and immediately attach to existing session or create new one. You can now safely disconnect without losing work.

### 8. Use Synchronize for Batch Operations

Perfect for deploying to multiple servers at once.

---

## Advanced Topics to Learn Next

Once comfortable with basics:

- **Session scripting:** Automate entire layout creation with shell scripts
- **Formats:** Use `#{...}` expressions for custom status bar
- **Hooks:** Trigger commands on events (session creation, pane changes)
- **Buffers:** Internal copy/paste system between panes
- **Control mode:** Integrate tmux with editors and automation
- **Popup windows:** (tmux 3.2+) Temporary floating terminals
- **Choose modes:** Interactive selectors for panes, windows, buffers

---

## Quick Start Checklist

- [ ] Install tmux
- [ ] Create first session: `tmux new -s work`
- [ ] Learn basic shortcuts: create, split, navigate
- [ ] Practice detaching and reattaching: `Prefix + d` then `tmux attach`
- [ ] Enable mouse in ~/.tmux.conf
- [ ] Practice copy mode: `Prefix + [` then navigate and copy
- [ ] Set up custom prefix key if desired
- [ ] Experiment with window and pane layouts
- [ ] Try SSH workflow with persistent sessions
- [ ] Install plugins (optional but recommended)

---

## Key Takeaway

Master **sessions, windows, panes, copy mode, and a few scripts**, and you'll be productive with tmux for nearly any development workflow. The time investment pays off enormously.