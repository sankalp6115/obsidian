Shell Scripting is a way to interact with computers using pieces of codes and commands just like we do with GUI.

Bash scripts can be used to automate tedious, repetitive tasks saving time, reducing confusion, human effort and mistakes.
Linux uses bash by default.
In windows , we need to use WSL.

Mac comes with bash, but by default uses zsh:
To activate bash:
```bash
sankalpomar@sankalp-macbook-air bash-scripting % bash
The default interactive shell is now zsh.
To update your account to use zsh, please run `chsh -s /bin/zsh`.
For more details, please visit https://support.apple.com/kb/HT208050.
bash-3.2$ 

```


Use Case:
If an engineer needs to check logs for any problems, errors in a service, he needs to look up logs, run commands like:
```bash
grep "[ERROR]" "server_logs.log"
```
If he does this everyday, he wastes a lot of time, just juggling through different commands, and needs to memorise the syntax, flags everything.

Problems:
Repeat same commands over and over
Wastes time
No aggregate summary, all logs scattered
Have to mentally track everything
Easy to lose progress if interrupted

Using shell script:
Logic defined once
Run complete script with single command
No manual repetition
Automate task
Consistent execution
Acts as documentation
Error handling

Shell file is a text file with linux commands.
Saves with .sh extension

## Raw working process:

- Create file using 
```bash
touch filename.sh
```

- Edit using vim,nano,neovim...

- Make executable using:
```bash
chmod +x <filename.sh>
```

- Run using:
```bash
./filename.sh
```

If we write a script and save it without the .sh extesnion, i still works, linux will still execute it, .sh extension is a convention, to classify script files from normal files and human-friendly.

The commands like touch, cd, mkdir, are all shell scripts that work on arguments.

Shell programs like bash,zsh,shell all use .sh as extension.
The executor can only tell them apart by **Shebang lines**.

Shebang
AKA Hashbang
A character sequence at the very top of script file, that tells the OS which interpreter to use to run the file.

bash -> #!/bin/bash
POSIX shell -> #!/bin/sh

Formmatting and Readability
If script is written normally, executing it ->
It does not have much context
Bad Readablity
Start and end of command
Hard to understand without referring to script
Not human-readable

- Use echo command:

```bash
echo -e "\n======================="
echo -e "\nStarting log analysis"
echo -e "\n======================="
```

- Using absolute paths:
```bash
/Users/server/logs/server_logs.log ✅
/Users/server/logs/server_logs.log ❌
```

- Variables:

```bash
LOG_DIRECTORY="/Users/server/logs"
SERVER_LOG="server_logs.log"
find $LOG_DIRECTORY -name "*.log" -mtime -1
```

*You are not allowed to have spaces around equals in bash*
```bash
VAR="HELLO WORLD" ✅
VAR = "HELLO WORLD" ❌
```

- Arrays:
0-Indexed
```bash
ERROR_PATTERNS=("ERROR","FATAL","CRITICAL")

Accessing:
${ERROR_PATTERNS[0]}
```

- Command Substitution:
```bash
LOG_FILE=$(find $LOG_DIR -name "*.log" -mtime -1)
```
Stores command output as a variable

- Loops:
Execute a set of commands multiple times
```bash
for LOG_FILE in $LOG_FILES; do
#commands
#commands
#commands
done

```
Makes code cleaner, smaller and more understandable

```bash
for PATTERN in ${ERROR_PATTERNS[@]}; do

[@] -> Array subscript means each item is its separate entity
[*] -> expands all items into a large string
```

- Writing to file:
```bash
FILE="/Users/server/logs/server_logs.log"
echo "HELLO WORLD" >> $FILE

Redirection Operators:
> : Overwrite previous content with new content
>> : Appends new output to existing items of file
```

- Conditionals:
```bash
ERROR_COUNT = grep -c "ERROR" "LOG_FILE"
echo $ERROR_COUNT >> "$REPORT_FILE"
if ["$ERROR_COUNT" -gt 10]; then
	echo "Action required, too many error $PATTERNS found in $LOG_FILE"
```

### Command redirection
`2>>` is a **shell redirection operator**.
```
termux-media-player play "$HOME/system_scripts/login.mp3" 2>> ~/log/error.log
```
- `termux-media-player play "$HOME/system_scripts/login.mp3"` → Run the command.
- `2>> ~/log/error.log` → Append anything written to **stderr (standard error)** to `~/log/error.log`.

### What does `2` mean?

Every process has three standard streams:

| Number | Stream | Purpose              |
| ------ | ------ | -------------------- |
| `0`    | stdin  | Input to the program |
| `1`    | stdout | Normal output        |
| `2`    | stderr | Error messages       |