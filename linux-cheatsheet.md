# Linux Basics Cheat Sheet (Aiman's Edition)
Covers: Linux Journey command line section, Text Fu section (lessons 1 to 13), cmdchallenge battles, and all labs

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
- Verify with ls -l, shows linkname -> target

## Reading Files
| Command | What it does | Example |
|---|---|---|
| cat | Print whole file to screen | cat readme |
| less | Scroll through a file page by page | less bigfile.log |
| head | Show the FIRST 10 lines of a file by default | head -n 5 file.txt |
| tail | Show the LAST 10 lines of a file by default | tail -n 5 file.txt |

### head flags
- n = number of lines
- c = number of bytes instead of lines
- q = quiet, suppress filename headers when using multiple files
- v = verbose, show the header even for one file

### tail flags
- n = number of lines
- n +N = start at line N and print to the end (tail -n +5 file)
- f = follow, watch new lines appear live in a log (Ctrl+C to stop)
- F = follow by name, survives log rotation, reopens the file if replaced

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
- find . -maxdepth 1 -type f = only files in this directory

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
- Example: grep -rh "500" -r --include="access.log*" = search for 500 only inside files starting with access.log, recursively

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

## Streams (stdin, stdout, stderr)
Every command has three streams:
- stdin (0) = input the command reads, default is the keyboard
- stdout (1) = normal output, default is the screen
- stderr (2) = error messages, default is the screen

## Output Redirection
| Operator | What it does | Example |
|---|---|---|
| > | Send stdout to a file, OVERWRITES if file exists | ls -l > file_list.txt |
| >> | Send stdout to a file, APPENDS to the end | echo log >> activity.log |
| 2> | Send stderr (errors only) to a file | find / -name x 2> errors.txt |
| 2>> | Append stderr to a file | command 2>> errors.log |
| &> | Send BOTH stdout and stderr to a file | command &> all_output.txt |
| &>> | Append both stdout and stderr | command &>> full.log |
| > file 2>&1 | Old style for both streams (seen in older scripts) | command > file 2>&1 |
| 2>/dev/null | Throw errors away, show only clean output | find / -name x 2>/dev/null |

## Input Redirection
| Operator | What it does | Example |
|---|---|---|
| < | Feed a file as stdin to a command | sort < items.txt |
| wc < file | vs wc file: with < the command gets a stream, no filename to print | wc -l < items.txt |
| cat < in > out | Chain input and output redirection | cat < a.txt > b.txt |

## Pipes and tee
| Command | What it does | Example |
|---|---|---|
| \| | Pipe, connect stdout of left command to stdin of right command | cat log \| grep error |
| tee | Save output to a file AND show it on screen at the same time | ls -l \| tee file_list.txt |
| tee -a | Append instead of overwrite | date \| tee -a activity.log |
| tee in pipeline | Save the intermediate result and keep processing | ls /etc \| tee listing.txt \| grep conf |

## Text Processing
| Command | What it does | Example |
|---|---|---|
| sort | Sort lines alphabetically | sort names.txt |
| sort -r | Reverse the order | sort -r names.txt |
| sort -n | Sort numerically (so 2 comes before 10) | sort -n scores.txt |
| sort -nr | Numeric, largest first | sort -nr scores.txt |
| sort -h | Sort human sizes such as 500, 1K, 2M | du -h \| sort -h |
| sort -k | Sort by a key (a field), write it as start,end | sort -k 2,2 file.txt |
| sort -t | Set the field separator used by -k | sort -t ':' -k 2,2n file.txt |
| sort -u | Sort and keep one line per equal key | sort -u names.txt |
| sort -o | Write the result to a file, safe even when it is the input file | sort -o names.txt names.txt |
| uniq | Remove ADJACENT duplicate lines, so sort first | sort names.txt \| uniq |
| uniq -c | Count how many times each line appears | sort names.txt \| uniq -c |
| wc | Count lines, words, characters | wc -l file.txt |
| cut -c | Select characters by position, starts at 1 | cut -c 1 file |
| cut -f | Select fields, default delimiter is tab | cut -f 2 file |
| cut -d | Set a custom delimiter for field mode | cut -d ':' -f 1 /etc/passwd |
| cut -s | Suppress lines that do not contain the delimiter | cut -s -d ';' -f 2 file |
| paste | Join lines from files as columns, default separator is tab | paste names.txt roles.txt |
| paste -d | Set a custom separator | paste -d ':' a.txt b.txt |
| paste -s | Serial mode, join all lines of a file into one line | paste -s words.txt |
| tr | Translate characters one for one, reads stdin only | tr a-z A-Z < names.txt |
| tr -d | Delete every character in the set | echo "abc123" \| tr -d '0-9' |
| tr -s | Squeeze each run of a character into one | echo "a    b" \| tr -s ' ' |
| tr -c | Complement the set (everything NOT in it), used with -d to keep only some characters | tr -cd '[:alnum:]\n' < file.txt |
| expand | Convert tabs to spaces | expand -t 4 file.txt |
| unexpand | Convert spaces to tabs | unexpand -t 4 file.txt |
| join | Combine lines of two SORTED files that share a key | join a.txt b.txt |
| split | Divide one file into smaller files | split -l 500 big.txt part- |
| date | Show current date and time, great for logs | date >> activity.log |
| echo | Print text | echo hello |

### cut and paste notes
- cut and paste read from stdin when no file is given, so both fit naturally in pipes
- a - operand means read that input position from stdin
- when paste input files have different lengths, missing values become empty fields
- cut picks columns, grep picks lines

### wc flags
- l = lines only, w = words only, c = bytes only

### counting with pipes
- find . -maxdepth 1 -type f \| wc -l = count files in current directory
- grep "GET" access.log \| wc -l = count matching lines
- sort \| uniq -c \| sort -rn = most common lines first

### sort notes
- sort never changes its input file, it prints the result to stdout
- Lexical order compares characters, so 10 comes before 2. Use -n for numbers
- -k START,END chooses a key. Write -k 2,2 to sort on field 2 only. A bare -k 2 runs to the end of the line
- Put the type letters after the key: -k 2,2nr = numeric and reversed on field 2
- -t sets the separator for -k (default is runs of blanks)
- -f ignores case, -b ignores leading blanks, -V sorts version numbers, -c only checks if input is already sorted
- LC_ALL=C gives plain byte order, so uppercase comes before lowercase (A B a b). Use it in scripts for repeatable results
- sort file > file empties the file before sort reads it. Use sort -o file file
- sort -u -k keeps only the first line of each group of equal keys
- Recon use: cat subs1.txt subs2.txt | sort -u merges and de-duplicates subdomain lists

### sort -u vs uniq
- sort -u sorts and removes duplicates in one step
- uniq removes only ADJACENT duplicates, so unsorted input keeps its duplicates
- sort | uniq gives the same result as sort -u for plain de-duplication
- uniq has extras: -c counts each line, -d shows only repeated lines, -u shows only lines that appear once
- Name clash: sort -u means unique output, uniq -u means only lines never repeated

### tr notes
- tr works on single characters, never on whole words. tr 'abc' 'xyz' swaps a to x, b to y, c to z, it does not replace the word abc
- tr reads stdin only, no file operands. Use a pipe or <
- Syntax: tr [OPTIONS] SET1 [SET2]. With two sets, characters map by position
- If SET2 is shorter, GNU tr repeats its last character (echo abc | tr abc xy gives xyy)
- Character classes: [:lower:] [:upper:] [:digit:] [:alpha:] [:alnum:] [:space:] [:punct:]
- tr -cd '[:alnum:]' also deletes the newline. Keep it with tr -cd '[:alnum:]\n'
- Escapes such as \n, \t and \r work inside the sets. Quote the sets so the shell does not expand them
- Windows line endings (CRLF) break scripts: tr -d '\r' < file > clean.txt
- One word per line: tr -s ' ' '\n' < file
- rot13: tr 'A-Za-z' 'N-ZA-Mn-za-m' (run it twice to get the original back, Bandit uses it)
- Changing words needs sed, which comes later

### expand and unexpand notes
- A tab moves to the next tab stop, it is not a fixed number of spaces. Default tab stop is 8 columns
- expand -t N = tab stops every N columns. expand -i = convert only leading tabs
- unexpand converts only leading blanks by default. -a considers blanks anywhere in the line. -t N also turns on -a
- If a file was made with 4 column stops, use -t 4, otherwise unexpand leaves four spaces as spaces
- Neither command edits its input. Redirect to a different file, never to the same one
- cat -A file shows tabs as ^I and line ends as $
- Makefiles need real tabs, YAML does not allow tabs, Python complains about mixed tabs and spaces
- cut -f and paste use tab as the default separator, so tabs turned into spaces break them

### join notes
- join matches lines from two files that share a key field. Both files must be sorted on that field with the same rules
- Default key is field 1 of each file. -1 N and -2 N choose the key in file 1 and file 2
- Prepare the input: LC_ALL=C sort -k 1,1 a.txt > a.sorted (and the same for the second file)
- Example: join -1 2 -2 1 people.txt surnames.txt (people.txt sorted on field 2, surnames.txt on field 1)
- Output order: key, then the rest of file 1, then the rest of file 2
- -t CHAR sets a single character separator such as :
- -a 1 or -a 2 also prints unpaired lines from that file. -v 1 or -v 2 prints ONLY the unpaired lines
- -o 0,1.2,2.2 chooses the output columns (0 = key, 1.2 = field 2 of file 1). -i ignores case
- Unsorted input gives a "not sorted" warning and missing matches

### split notes
- split writes consecutive pieces of one file into new files. It does not change the original
- Default: 1000 lines per piece, prefix x, names xaa, xab, xac
- -l N = N lines per piece. -b SIZE = bytes per piece (K, M, G are powers of 1024). -n N = N pieces
- Last operand is the prefix: split -l 500 big.txt part-
- -d = numeric suffixes (part-00), -a N = suffix length, --additional-suffix=.txt keeps an extension
- A lone - reads stdin: cat big.txt | split -l 500 - part-
- Rebuild: cat part-* > big.txt, then check with cmp or sha256sum
- split cuts a file in pieces, join combines by key, they are not opposites

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
| env | Show all environment variables as NAME=value | env |
| export | Mark a variable to be inherited by child processes | export TEST=test |
| $VARIABLE | Use a variable, quote it to keep it as one argument | echo "$HOME" |
| echo $$ | Show the PID of my current shell | echo $$ |
| ps -f | Detailed process info including PPID (parent PID) | ps -f |
| source file | Run a file's commands in the current shell, reloads config | source ~/.bashrc |
| set -o name | Turn a shell option on | set -o noclobber |
| set +o name | Turn a shell option off | set +o noclobber |
| env -i COMMAND | Run a command with an empty environment | env -i bash |

### environment variable concepts
- Every process has an environment: name=value strings inherited from its parent process
- Bash expands $NAME or ${NAME} before running a command
- Local variables stay in the current shell. Exported variables get copied to every child process. A child can never change its parent's variables
- Inline assignment only affects one command: LANG=C sort names.txt
- set -o allexport = automatically export every variable defined after it
- .bashrc and .zshrc = startup files, anything in them runs every time a shell opens, this is where aliases, variables and options become permanent
- noclobber = prevents accidental overwrite of existing files with > (add set -o noclobber to your rc file to make it permanent)

### PATH rules
- PATH is a colon-separated list of directories the shell searches for commands
- echo $PATH to see it, printf '%s\n' "$PATH" also works
- Add a folder safely, keeping the old path: export PATH="/opt/coolapp/bin:$PATH"
- Do NOT replace PATH with only the new directory, normal commands stop resolving
- Do NOT add untrusted writable directories, an attacker could drop a malicious executable there that runs instead of the real command

### security notes
- Environment variables can contain credentials, tokens, internal paths. Never paste full env output into public issues or logs without redacting
- Never put passwords or API keys in commands directly, they get saved in history

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
- Command lines get stored in history, never type secrets directly

## Processes
| Command | What it does | Example |
|---|---|---|
| ps | Show running processes | ps aux |
| ps -f | Full format, includes PPID | ps -f |
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
- Solutions: ls -a to see them, find . -mindepth 1 -delete to remove everything

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
| Watch a log file live as it grows | tail -f application.log |
| Watch a log that gets rotated | tail -F application.log |
| Skip the first 4 lines and print the rest | tail -n +5 file |
| Redo the last command | !! |
| Errors are flooding my output | add 2>/dev/null to the command |
| Save output to a file AND see it live | command \| tee file.txt |
| Add a timestamp to a log | date >> logfile |
| Extract one column from a file | cut -d ':' -f 1 file |
| Combine two files side by side as columns | paste -d ':' a.txt b.txt |
| Turn a list into one single line | paste -s words.txt |
| Sort numbers properly (2 before 10) | sort -n file |
| Sort by one column of a delimited file | sort -t ':' -k 2,2n file |
| Remove duplicate lines | sort -u file |
| Remove duplicates AND count them | sort file \| uniq -c |
| Sort a file and save over itself safely | sort -o file file |
| Change the case of text | tr a-z A-Z < file |
| Delete digits or punctuation | tr -d '[:punct:]' < file |
| Squeeze repeated spaces into one | tr -s ' ' < file |
| One word per line | tr -s ' ' '\n' < file |
| Remove Windows line endings (CRLF) | tr -d '\r' < file > clean.txt |
| Decode or encode rot13 | tr 'A-Za-z' 'N-ZA-Mn-za-m' |
| Convert tabs to spaces | expand -t 4 file |
| Convert spaces to tabs | unexpand -t 4 file |
| See hidden tabs and line endings | cat -A file |
| Match records from two files by a shared key | sort both files on the key, then join |
| Lines in file A with no match in file B | join -v 1 A B |
| Break a big file into pieces | split -l 500 big.txt part- |
| Put the pieces back together | cat part-* > big.txt |
| Command not found | tool missing or not in PATH, try which toolname |
| Add a folder to PATH safely | export PATH="/new/dir:$PATH" |
| I forgot how a command works | man commandname |
| Need to install a tool | sudo apt install toolname |
| Need root power for one command | sudo command |
| Download a file | wget URL |
| Zip up a folder | tar -czf backup.tar.gz folder/ |
| Unzip a tarball | tar -xzf file.tar.gz |
| Need a shortcut for a long command | alias name='long command' |
| Make an alias or variable permanent | add it to ~/.bashrc, then source ~/.bashrc |
| Stop myself overwriting files by accident | set -o noclobber |
| Reload shell config without closing terminal | source ~/.bashrc |

## Golden Rules
1. Filenames starting with - need ./ in front, example: cat ./-file07
2. Filenames with spaces need quotes, example: cat "my file.txt"
3. No output usually means wrong directory. Run pwd first
4. When stuck: man commandname. The manual always knows
5. Never put passwords or API keys directly in commands, they get saved in history
6. rm has no undo. There is no recycle bin
7. The * wildcard never matches dotfiles. Hidden files need ls -a or find
8. include and exclude only work together with grep -r
9. Recursive means find or grep -r
10. Every problem is: find it, read it, filter it, count it. Pipes connect the steps
11. cut picks columns, grep picks lines, pipes connect commands
12. Never replace PATH, always prepend: export PATH="/new/dir:$PATH"
13. join needs both inputs sorted on the join field with the same rules (LC_ALL=C on both)
14. uniq only removes ADJACENT duplicates. Sort first, or use sort -u
15. tr works on single characters, not words, and reads only stdin. Use < or a pipe
16. Never redirect output onto the file being read. sort file > file empties it. Use sort -o or a temp file
17. Write sort keys as -k 2,2 (start,end). A bare -k 2 runs to the end of the line
