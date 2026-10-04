# Linux Basics Cheat Sheet (Aiman's Edition)

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

## Finding Things
| Command | What it does | Example |
|---|---|---|
| find | Search for files by name, size, type | find . -name "*.txt" |
| grep | Search for TEXT inside files | grep "password" log.txt |
| which | Shows where a command lives | which python3 |

### find flags
- name = search by name (use quotes and wildcards)
- type f = files only, type d = directories only
- size = filter by size, c for bytes, k for KB, + means bigger, - means smaller
- Examples: find . -type f -size 1033c | find . -name "*.log"

### grep flags
- i = ignore uppercase or lowercase
- r = search recursively through folders
- n = show line numbers
- v = invert, show lines that do NOT match
- Example: grep -rni "error" /var/log

## Permissions
| Command | What it does | Example |
|---|---|---|
| chmod | Change file permissions | chmod 755 script.sh |
| chown | Change file owner | chown user file.txt |

### Reading ls -l output
-rwxr-xr-x 1 owner group size date filename

Three groups of three, r = read, w = write, x = execute
- First group = owner, second = group, third = everyone else
- On directories, x = enter the directory

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
| wc | Count lines, words, characters | wc -l file.txt |
| cut | Cut out columns of text | cut -d: -f1 /etc/passwd |
| tr | Replace or delete characters | tr a-z A-Z |
| echo | Print text | echo hello |

### wc flags
- l = lines only, w = words only, c = bytes only

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
- Up arrow = previous commands
- !! = run the last command again
- !102 = run command number 102 from history
- !cat = run the most recent command starting with cat
- history -c = clear history, history -w = save to file

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

## Links
| Command | What it does | Example |
|---|---|---|
| ln -s | Create a symbolic link (shortcut) | ln -s target linkname |

Order: target first, link name second. Verify with ls -l, shows target -&gt; linkname

## Wildcards
| Symbol | Matches |
|---|---|
| * | Any sequence of characters |
| ? | Any single character |
| [abc] | Any one character inside brackets |
| Example: cp *.jpg pics/ copies all jpg files |

## Golden Rules
1. Filenames starting with - need ./ in front, example: cat ./-file07
2. Filenames with spaces need quotes, example: cat "my file.txt"
3. No output usually means wrong directory. Run pwd first
4. When stuck: man commandname. The manual always knows
5. Never put passwords or API keys directly in commands, they get saved in history
6. rm has no undo. There is no recycle bin

## Text Editing
| Command | What it does | Example |
|---|---|---|
| nano | Simple text editor inside the terminal | nano notes.txt |

### nano controls (shown at the bottom of the screen, ^ = Ctrl)
- Ctrl+O = save (Write Out), then Enter to confirm
- Ctrl+X = exit (asks to save if changes exist)
- Ctrl+W = search inside the file
- Ctrl+K = cut the current line
- Ctrl+U = paste

## Text Power (Text Fu)
| Command | What it does | Example |
|---|---|---|
| &gt; | Redirect output to a file (overwrites) | echo hi &gt; file.txt |
| &gt;&gt; | Redirect output, APPENDS to file | echo more &gt;&gt; file.txt |
| \| | Pipe, send output of one command into another | cat log \| grep error |
| sort | Sort lines alphabetically | sort names.txt |
| uniq | Remove duplicate lines (use after sort) | sort names.txt \| uniq |
| wc | Count lines, words, characters | wc -l file.txt |
| cut | Cut out columns of text | cut -d: -f1 /etc/passwd |
| tr | Replace or delete characters | tr a-z A-Z |
| echo | Print text | echo hello |

### wc flags
- l = lines only, w = words only, c = bytes only

## Users and sudo
| Command | What it does | Example |
|---|---|---|
| whoami | Shows which user I am | whoami |
| id | Shows my user ID, group, and all groups I belong to | id |
| sudo | Run a command as root (superuser) | sudo apt update |
| su | Switch to another user | su - root |

### sudo notes
- sudo -l = list what commands I am allowed to run as root (first check in privilege escalation)
- sudo asks for MY password, not the root password
- Running commands as root = full power, use carefully

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
- $PATH = the list of folders where the shell looks for commands. Command not found usually means the tool is not in PATH
- Pentest relevance: PATH manipulation is a classic privilege escalation trick
- Common ones: $HOME (my home folder), $USER, $SHELL

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
- Examples: tar -xf archive.tar | tar -xzf archive.tar.gz | tar -czf backup.tar.gz folder/
