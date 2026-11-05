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
Clears previous content and pushes new content to file.

#### >>
Redirection is used to redirect output of a command to a file.
Appends new content to previosu content.

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

### Linux Pipelines
`set -euo pipefail`

This combines all three.

```
set -euo pipefail
```

means

- `-e` → Exit on command failure.
- `-u` → Error on undefined variables.
- `-o pipefail` → Pipelines fail if **any** command fails.

This is the most common header you'll see in professional Bash scripts


### Default Variable usage

```
${VAR:-default}
```
"If `VAR` is unset or empty, use `default` instead."


### Weather info
```bash
curl wttr.in/kanpur
sankalp@ROG:~$ curl wttr.in/kanpur
Weather report: kanpur

      \   /     Clear
       .-.      +25(24) °C
    ― (   ) ―   → 8 km/h
       `-’      10 km
      /   \     0.0 mm
                                                       ┌─────────────┐
┌──────────────────────────────┬───────────────────────┤  Tue 03 Nov ├───────────────────────┬──────────────────────────────┐
│            Morning           │             Noon      └──────┬──────┘     Evening           │             Night            │
├──────────────────────────────┼──────────────────────────────┼──────────────────────────────┼──────────────────────────────┤
│     .-.       Haze           │     \   /     Sunny          │     \   /     Clear          │     \   /     Clear          │
│      __)      27 °C          │      .-.      +30(29) °C     │      .-.      +25(26) °C     │      .-.      +24(25) °C     │
│     (         ↘ 10-13 km/h   │   ― (   ) ―   → 12-13 km/h   │   ― (   ) ―   → 8 km/h       │   ― (   ) ―   → 6 km/h       │
│      `-’      5 km           │      `-’      8 km           │      `-’      10 km          │      `-’      8 km           │
│       •       0.0 mm | 1%    │     /   \     0.0 mm | 1%    │     /   \     0.0 mm | 1%    │     /   \     0.0 mm | 1%    │
└──────────────────────────────┴──────────────────────────────┴──────────────────────────────┴──────────────────────────────┘
                                                       ┌─────────────┐
┌──────────────────────────────┬───────────────────────┤  Wed 04 Nov ├───────────────────────┬──────────────────────────────┐
│            Morning           │             Noon      └──────┬──────┘     Evening           │             Night            │
├──────────────────────────────┼──────────────────────────────┼──────────────────────────────┼──────────────────────────────┤
│     \   /     Sunny          │     \   /     Sunny          │               Overcast       │               Overcast       │
│      .-.      26 °C          │      .-.      +30(28) °C     │      .--.     25 °C          │      .--.     +24(25) °C     │
│   ― (   ) ―   ↗ 9-11 km/h    │   ― (   ) ―   → 10-12 km/h   │   .-(    ).   → 4 km/h       │   .-(    ).   ↗ 4 km/h       │
│      `-’      7 km           │      `-’      5 km           │  (___.__)__)  7 km           │  (___.__)__)  6 km           │
│     /   \     0.0 mm | 2%    │     /   \     0.0 mm | 1%    │               0.0 mm | 10%   │               0.0 mm | 10%   │
└──────────────────────────────┴──────────────────────────────┴──────────────────────────────┴──────────────────────────────┘
                                                       ┌─────────────┐
┌──────────────────────────────┬───────────────────────┤  Thu 05 Nov ├───────────────────────┬──────────────────────────────┐
│            Morning           │             Noon      └──────┬──────┘     Evening           │             Night            │
├──────────────────────────────┼──────────────────────────────┼──────────────────────────────┼──────────────────────────────┤
│     .-.       Smoky haze     │     \   /     Sunny          │     \   /     Clear          │     .-.       Haze           │
│      __)      26 °C          │      .-.      +30(28) °C     │      .-.      26 °C          │      __)      +24(25) °C     │
│     (         ↗ 8-10 km/h    │   ― (   ) ―   → 9-10 km/h    │   ― (   ) ―   ↘ 2 km/h       │     (         → 0-1 km/h     │
│      `-’      8 km           │      `-’      6 km           │      `-’      6 km           │      `-’      5 km           │
│       •       0.0 mm | 1%    │     /   \     0.0 mm | 0%    │     /   \     0.0 mm | 1%    │       •       0.0 mm | 1%    │
└──────────────────────────────┴──────────────────────────────┴──────────────────────────────┴──────────────────────────────┘
Location: Kanpur, Kanpur Nagar, Uttar Pradesh, 208012, India [26.4609135,80.3217588]

Follow @igor_chubin for wttr.in updates
```


### File System
Starting pt is root dir
| Directory | Purpose |
| --------- | --------- |

| Directory | Purpose                                            |
| --------- | -------------------------------------------------- |
| /bin      | Essential user command binaries                    |
| /boot     | Static files of bootloader                         |
| /dev      | Device files                                       |
| /etc      | Editable text config, host specific system configs |
| /home     | all users (other than root) directories live here  |
| /lib      | shared libraries                                   |
| /media    | removable media                                    |
| /mnt      | temporary mounted fs                               |
| /opt      | add-on app software package                        |
| /sbin     | system binaries                                    |
| /srv      | data for svc from system                           |
| /tmp      | temporary fs                                       |
| /usr      | user utilities and applications                    |
| /proc     | process info                                       |
| /var      | changing files like logs etc live here             |
|           |                                                    
/root: ultra-secure private suite reserved for root user (Superuser) (Admin)

### Absolute and Relative Paths
absolute: full path from root directory
e.g. 
```bash
cd /home/manish/devops
#current dir: /home/manish
```
relative: incomplete path from current directory
```bash
cd devops
#current dir: /home/manish
```

