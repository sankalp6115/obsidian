## Core Shell Commands

- `cd`
- `pwd`
- `ls`
- `mkdir`
- `touch`
- `cp`
- `mv`
- `rm`
- `cat`  
- `less`
- `more`
- `head`
- `tail`
- `wc`
- `sort`
- `uniq`
- `cut`
- `tr`
- `tee`
- `find`
- `grep`
- `sed`
- `awk`
- `xargs`

---

## Shell Scripting Basics

- Shebang (`#!/bin/bash`, `#!/usr/bin/env bash`)
- Variables
- Environment variables
- `export`
- Command substitution (`$(...)`)
- Arithmetic expansion (`$((...))`)
- Conditionals (`if`, `elif`, `else`)
- Loops (`for`, `while`, `until`)
- Case statements (`case`)
- Functions
- Positional parameters (`$1`, `$2`, ...)
- Argument count (`$#`)
- All arguments (`$@`, `$*`)
- Script name (`$0`)
- Return values
- Exit codes
- `return`
- `exit`

---

## Operators and Condition Testing

- String comparisons
- Numeric comparisons
- File tests (`-f`, `-d`, `-e`, `-r`, `-w`, `-x`)
- Logical operators (`&&`, `||`, `!`)
- Test commands:
    - `test`
    - `[ ]`
    - `[[ ]]`

---

## Redirection and Pipes

- `>`
- `>>`
- `<`
- `<<` (heredoc)
- `<<<` (herestring)
- `2>`
- `2>>`
- `2>&1`
- `|`
- `tee`

---

## Quoting and Expansion

- Single quotes (`'`)
- Double quotes (`"`)
- Escaping (`\`)
- Word splitting
- Filename expansion (globbing)
- Brace expansion
- Tilde expansion
- Parameter expansion
- Arithmetic expansion
- Command substitution
- Process substitution

Examples:

- `*`
- `?`
- `[abc]`
- `[0-9]`
- `{1..10}`
- `{dev,prod,test}`

Parameter expansions:

- `${var}`
- `${var:-default}`
- `${var:=default}`
- `${var:+value}`
- `${var:?error}`
- `${var#pattern}`
- `${var##pattern}`
- `${var%pattern}`
- `${var%%pattern}`
- `${var//old/new}`

---

## Arrays

- Indexed arrays
- Associative arrays
- Array iteration
- Array length
- Array expansion

---

## File Permissions and Ownership

- `chmod`
- `chown`
- `chgrp`
- `umask`
- User permissions
- Group permissions
- Others permissions
- Symbolic permissions
- Numeric permissions (`755`, `644`, etc.)

---

## Processes and Jobs

- `ps`
- `top`
- `htop`
- `pgrep`
- `pkill`
- `kill`
- `killall`
- Background jobs (`&`)
- `jobs`
- `fg`
- `bg`
- `nohup`
- `disown`
- Signals
- Process IDs (PID)

---

## Shell Configuration

- `.bashrc`
- `.bash_profile`
- `.profile`
- `.zshrc`
- `.zprofile`
- Aliases
- Functions in shell config
- Prompt customization (`PS1`)
- `PATH`

---

## File Descriptors and Streams

- Standard input (`stdin`)
- Standard output (`stdout`)
- Standard error (`stderr`)
- File descriptors (`0`, `1`, `2`)
- Custom file descriptors
- `exec`

---

## Signal Handling

- `trap`
- Common signals:
    - `SIGINT`
    - `SIGTERM`
    - `SIGKILL`
    - `SIGHUP`
    - `SIGQUIT`

---

## Useful Utilities

- `curl`
- `wget`
- `jq`
- `tar`
- `gzip`
- `gunzip`
- `zip`
- `unzip`
- `ssh`
- `scp`
- `rsync`
- `du`
- `df`
- `mount`
- `uname`
- `which`
- `whereis`
- `realpath`
- `env`
- `printenv`
- `history`
- `time`

---

## Advanced/Optional Topics

- Named pipes (`mkfifo`)
- Coprocesses
- Process substitution (`<(...)`, `>(...)`)
- `getopts`
- Shell options (`set`, `shopt`)
- Strict mode:
    
    ```
    set -euo pipefail
    ```
    
- Subshells
- Group commands (`{ ...; }`)
- Command grouping (`( ... )`)
- Job control internals
- Shell startup order
- Login vs interactive shells
- Shell portability (`bash` vs `sh`)
- POSIX shell scripting