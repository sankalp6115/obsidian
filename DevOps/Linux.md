### Tools
tmux
wget
alpine requires repo # removal change

### rsync command
keeps content synced between two directories.
```bash
rsync -avz --progress -e "ssh -p 8022" \
  --exclude='node_modules' \
  --exclude='.git' \
  --exclude='dist' \
  /path/to/source/ user@alpine-ip:/path/to/destination/
```

### fswatch
Watches a folder for changes and executes a command on event.

### conda
![[Pasted image 20260514023342.png]]


## Commands

##### uname
Shows basic info about OS name and system hardware.

Options
-a : Shows all information
-s : Shows kernel name (same as default uname)
-r : Display kernel release version
-v : Shows kernel version
-m : Prints machine hardware architecture (arm64)
-n : Display network hostname
-o : Print OS name
-p : Show processor type (may return ```unknown``` for some systems.)


#### pwd
Shows where in the filesystem we are at right now
#### cd
Changes directory
#### ls
List items

Options
-l : Gives detailed view of files, with size, permissions etc

#### cat
Displays file contents
Reads one or multiple files and prints contents to terminal.
#### | 
"Pipe" is used to pass output of one command as input to another. Allows to chain multiple commands together.
#### grep
Used to search for text patterns in files or outputs. It filters and displays lines that match a specific string or pattern.

#### >
Redirection is used to redirect output of a command to a file.


eg. cat error.log | grep Database > db_errors.txt
Only outputs those lines from error.log file content which contain the word "Database" and redirects them to db_errors.txt file. 

#### cp \[ source \] \[destination\]
 Copies file or folders from source to destination.

#### wc
Word count command used to count number of lines, words, characters in a file or standard input.

Options
-l : Count lines
-w : Count words
-c : Count bytes
-m : Count characters
-L : Find the longest line length

eg. cat error_logs.txt | grep "connection refused" | wc -l
#### find
search for files and directories based on specific attributes like name, size, type, modification 
time, or permissions

eg. find / -name "*.conf *" \ grep db

#### diff
To compare and find difference between two files

eg. diff db.conf db.conf.backup

#### curl 
Client-URL
CLI-based tool to interact with apis, supports http, https, sftp, ftp ...
Used to transfer data to and from network servers and debugging.
eg. curl \<URL\>

Options
-I : Fetches only headers of a webpage

#### vim
Code editor in CLI-mode
Insert mode, Normal mode, Command mode
i - insert mode
esc - normal mode
: command mode

:q - quit 
:q! - quit without save
:w - save (write) the file
:wq or ZZ - save and quit
:x - save and quit (only if changes were made)

#### File permissions

![](../Assets/Pasted%20image%2020260617143712.png)

### chmod
Change mode command modifies file permissions
r(read) = 4
w(write) = 2
x(execute) = 1

chmod 777 file.txt - Full permissions(r,w,x) for all
chmod 755 file.sh - Owner : rwx & Others : rx
chmod 644 file.txt - Owner : rw & Others : r
chmod 600 private.txt - Only owner can read and write

 