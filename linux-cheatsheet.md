# Linux Basics Cheat Sheet (Aiman's Edition)
Updated after Linux Journey command line section, Text Fu, and cmdchallenge battles

## Navigation
| Command | What it does | Example |
|---|---|---|
| pwd | Shows where I am (current directory) | pwd |
| cd | Change directory | cd /home, cd .., cd ~ |
| ls | List files and folders | ls, ls -l, ls -a, ls -la, ls -h |

### ls flags
- l = long format (permissions, owner, size, date)
- a = show hidden files (dotfiles)
- h = human readable sizes (K, M, G)

## Working with Files
| Command | What it does | Example |
|---|---|---|
| touch | Create an empty file (updates timestamp if file exists) | touch notes.txt |
| mkdir | Create a directory | mkdir folder, mkdir -p a/b/c |
| cp | Copy a file or folder | cp file.txt backup/, cp -r folder/ backup/ |
| mv | Move OR rename a file | mv a.txt b.txt, mv file.txt docs/ |
| rm | Delete a file permanently | rm file.txt, rm -r folder/ |
| file | Tells what type a file really is | file mysteryfile |
| ln -s | Create a symbolic link (shortcut) to a file | ln -s target linkname |

### cp / mv / rm flags
- r = recursive, needed for directories
- i = interactive, asks before overwriting or deleting
- f = force, no prompts
- n = never overwrite existing files
- u = update only, copy if source is newer
- v = verbose, show each file as it happens
- p = preserve permissions and timestamps
- b = make a backup before overwriting

### rm warning
No recycle bin. rm deletes forever. Think twice, especially with rm -rf

### symbolic links
- Order is: target first, link name second
- Verify with ls -l, shows linkname -&gt; target

## Reading Files
| Command | What it does | Example |
|---|---|---|
| cat | Print whole file to screen | cat readme |
| less | Scroll through a file page by page | less bigfile.log |
| head | Show the FIRST lines of a file | head -n 5 file.txt |
| tail | Show the LAST lines of a file | tail -n 5 file.txt |

### less controls
- space = next page
- b = back one page
- /word = search for word
- q = quit

## Text Editing
| Command | What it does | Example |
|---|---|---|
| nano | Simple text editor inside the terminal | nano notes.txt |

### nano controls (^ = Ctrl)
- Ctrl+O = save, then Enter to confirm
- Ctrl+X = exit
- Ctrl+W = search inside the file
- Ctrl+K = cut the current line
- Ctrl+U = paste

## Finding Files
| Command | What it does | Example |
|---|---|---|
| find | Search for files by name, size, type, anywhere below me | find . -name "*.txt" |
| which | Shows where a command lives | which python3 |

### find flags
- name = search by name (use quotes around patterns with wildcards)
- type f = files only, type d = directories only
- size = filter by size, c for bytes, k for KB, + bigger, - smaller
- maxdepth 1 = only current directory, do not go deeper
- mindepth 1 = do not include the starting directory itself
- delete = delete everything found (use with care)
- exec = run a command on each result, {} is the placeholder

### find examples
- find . -type f -size 1033c = files of exactly 1033 bytes here and below
- find . -name "*.log" = all .log files recursively
- find . -type f -name "*.doc" -delete = delete all .doc files recursively
- find . -mindepth 1 -delete = delete EVERYTHING here including dotfiles
- find . -maxdepth 1 -type f = only files in this directory, no subdirectories

## Searching Inside Files (grep)
| Command | What it does | Example |
|---|---|---|
| grep | Show lines in a file that match a string | grep "error" log.txt |
| grep -r | Search recursively through all files in folders | grep -r "password" . |
| grep -i | Ignore uppercase or lowercase | grep -i "error" log.txt |
| grep -n | Show line numbers | grep -n "500" access.log |
| grep -v | Invert, show lines that do NOT match | grep -v "404" access.log |
| grep -l | List only FILENAMES that contain a match (lowercase L) | grep -l "500" * |
| grep -h | Hide the filename prefix, show only the matching lines | grep -h "500" * |
| grep -o | Print ONLY the matching part, not the whole line | grep -o "GET" access.log |
| grep -E | Extended regex mode, enables advanced patterns | grep -E '[0-9]{1,3}' file |

### grep file filtering flags (need -r to work)
- include = only search files matching a pattern
- exclude = skip files matching a pattern
- Example: grep -rh "500" -r --include="access.log*" = search for 500 only inside files starting with access.log, recursively, hide filenames

### basic regex for grep -E
- [0-9] = any digit, [a-z] = any lowercase letter, [A-Z] = any uppercase
- {1,3} = repeat previous thing 1 to 3 times
- . = any single character
- ^ = start of a line (careful: IPs appear mid-line, so ^ would skip them)
- $ = end of a line
- Example IP pattern: [0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}
- Note: a real dot must be escaped with backslash, otherwise it means any character

## Permissions
| Command | What it does | Example |
|---|---|---|
| chmod | Change file permissions | chmod 755 script.sh |
| chown | Change file owner | chown user file.txt |

### Reading ls -l output
-rwxr-xr-x 1 owner group size date filename

Three groups of three, r = read, w = write, x = execute
- First group = owner, second = group, third = everyone else
- On directories, x = enter the directory (sometimes called search or traverse permission)

### chmod number method
- r = 4, w = 2, x = 1, add them up
- 755 = owner rwx, everyone else r-x
- 644 = owner rw-, everyone else r--
- Example: chmod 644 file.txt

## Text Power (Text Fu)
| Command | What it does | Example |
|---|---|---|
| &gt; | Redirect output to a file (overwrites) | echo hi &gt; file.txt |
| &gt;&gt; | Redirect output, APPENDS to file | echo more &gt;&gt; file.txt |
| \| | Pipe, send output of one command into another | cat log \| grep error |
| sort | Sort lines alphabetically | sort names.txt |
| uniq | Remove duplicate lines (use after sort) | sort names.txt \| uniq |
| uniq -c | Count how many times each line appears | sort names.txt \| uniq -c |
| wc | Count lines, words, characters | wc -l file.txt |
| cut | Cut out columns of text | cut -d: -f1 /etc/passwd |
| tr | Replace or delete characters | tr a-z A-Z |
| echo | Print text | echo hello |

### wc flags
- l = lines only, w = words only, c = bytes only

### counting with pipes
- find . -maxdepth 1 -type f \| wc -l = count files in current directory (including dotfiles)
- grep "GET" access.log \| wc -l = count matching lines
- sort \| uniq -c \| sort -rn = most common lines first

## Users and sudo
| Command | What it does | Example |
|---|---|---|
| whoami | Shows which user I am | whoami |
| id | Shows my user ID and all groups I belong to | id |
| sudo | Run a command as root (superuser) | sudo apt update |
| su | Switch to another user | su - root |

### sudo notes
- sudo -l = list what I am allowed to run as root (first check in privilege escalation)
- sudo asks for MY password, not the root password

## Package Management (installing tools)
| Command | What it does | Example |
|---|---|---|
| apt update | Refresh the list of available packages | sudo apt update |
| apt install | Install a program or tool | sudo apt install nmap |
| apt remove | Uninstall a program | sudo apt remove nmap |

### notes
- Always run apt with sudo
- apt works on Ubuntu and Debian. Other distros use different tools (dnf, pacman)

## Environment Variables
| Command | What it does | Example |
|---|---|---|
| env | Show all environment variables | env |
| export | Set or create an environment variable | export PATH=$PATH:/opt/tools |
| $VARIABLE | Use a variable, $ before the name | echo $PATH |

### notes
- $PATH = folders where the shell looks for commands. Command not found usually means the tool is not in PATH
- Common ones: $HOME (my home folder), $USER, $SHELL
- Never put passwords or API keys in commands, they get saved in history

## Archives (zip files of Linux)
| Command | What it does | Example |
|---|---|---|
| tar | Pack or unpack .tar, .tar.gz archives | tar -xf tools.tar.gz |
| gzip | Compress a file | gzip file.txt (makes file.txt.gz) |
| gunzip | Decompress a .gz file | gunzip file.txt.gz |

### tar flags
- x = extract
- f = file (always needed, comes last, then the filename)
- z = also handle gzip compression (.tar.gz files)
- v = verbose, show files as they extract
- Memory trick: eXtract Ze File = xzf
- Examples: tar -xf archive.tar | tar -xzf archive.tar.gz | tar -czf backup.tar.gz folder/

## System and Help
| Command | What it does | Example |
|---|---|---|
| man | The manual, full documentation of any command | man find |
| whatis | One line summary of a command | whatis grep |
| help | Help for shell builtin commands | help cd |
| type | Tells if a command is builtin or a program | type cd |
| history | Shows my previous commands | history |
| alias | Create a shortcut for a command | alias ll='ls -la' |
| clear | Clear the screen (history NOT erased) | clear |
| exit | Close the shell | exit |

### history tricks
- Up arrow = scroll through previous commands
- !! = run the last command again
- !102 = run command number 102 from history
- !cat = run the most recent command starting with cat
- history -c = clear in-memory history, history -w = save to ~/.bash_history
- Command lines get stored in history, never type passwords or tokens directly

## Processes
| Command | What it does | Example |
|---|---|---|
| ps | Show running processes | ps aux |
| top | Live view of processes and CPU or RAM | top |
| kill | Kill a process by ID | kill 1234 |
| Ctrl+C | Stop the running command |  |

## Network and Download
| Command | What it does | Example |
|---|---|---|
| wget | Download a file from the internet | wget https://site.com/file.zip |
| curl | Transfer data, download or send requests | curl https://api.site.com |
| ssh | Connect to a remote machine securely | ssh user@host -p 2220 |

## Wildcards
| Symbol | Matches |
|---|---|
| * | Any sequence of characters (does NOT match dotfiles) |
| ? | Any single character |
| [abc] | Any one character inside brackets |
| .[!.]* | Dotfiles only, but NOT . and .. themselves |
| Example: cp *.jpg pics/ copies all jpg files |

### THE hidden files trap
- ls without -a does not show dotfiles
- The * wildcard does NOT match dotfiles either
- So rm -rf * always leaves hidden files behind
- Attackers hide files as dotfiles for exactly this reason
- Solutions: ls -a to see them, find . -mindepth 1 -delete to nuke everything

## Situations: What do I use?
| Situation | Command |
|---|---|
| I do not know where I am | pwd |
| A command gives no output or weird output | pwd first, I am probably in the wrong directory |
| I need to find a file by name, anywhere below here | find . -name "filename" |
| I need to find files of a certain size | find . -type f -size 1033c |
| The task says recursively | find or grep -r |
| Delete all files of one type everywhere below | find . -name "*.doc" -delete |
| Delete everything in a folder including hidden files | find . -mindepth 1 -delete |
| Count files in current directory only | find . -maxdepth 1 -type f \| wc -l |
| Search for a word inside files | grep "word" file.txt |
| Search for a word in ALL files including subfolders | grep -r "word" . |
| Which files contain this word (names only) | grep -rl "word" . |
| Show matching lines without file paths | grep -rh "word" files |
| Pull out only IPs, emails, tokens from text | grep -hoE 'pattern' file |
| Search only certain files by name | grep -r "word" --include="*.log" . |
| Case does not matter | grep -i |
| Count how many lines match | grep "word" file \| wc -l |
| Most common lines in a file | sort file \| uniq -c \| sort -rn |
| Filename starts with a dash | cat ./-file07 (./ in front) |
| Filename has spaces | cat "my file.txt" (quotes) |
| File is hidden | ls -a to see it |
| Read a huge file slowly | less file |
| See last lines of a log | tail -n 20 file |
| Redo the last command | !! |
| Command not found | tool missing or not in PATH, try which toolname |
| I forgot how a command works | man commandname |
| Need to install a tool | sudo apt install toolname |
| Need root power for one command | sudo command |
| Download a file | wget URL |
| Zip up a folder | tar -czf backup.tar.gz folder/ |
| Unzip a tarball | tar -xzf file.tar.gz |
| Need a shortcut for a long command | alias name='long command' |

## Golden Rules
1. Filenames starting with - need ./ in front, example: cat ./-file07
2. Filenames with spaces need quotes, example: cat "my file.txt"
3. No output usually means wrong directory. Run pwd first
4. When stuck: man commandname. The manual always knows
5. Never put passwords or API keys directly in commands, they get saved in history
6. rm has no undo. There is no recycle bin
7. The * wildcard never matches dotfiles. Hidden files need special handling
8. --include and --exclude only work together with -r
9. Recursive means find or grep -r
10. Every problem is: find it, read it, filter it, count it. Pipes connect the steps
