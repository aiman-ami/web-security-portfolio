# Day 4: Managing Files Like a Boss

- Linux Journey command line lessons:
  - Lesson 9: history (recall & reuse past commands)
  - Lesson 10: cp (copy files/directories)
  - Lesson 11: mv (move + rename)
  - Lesson 12: mkdir (make directories)
  - Lesson 13: rm (remove files/directories)
- LabEx labs completed:
  - cp command: file copying
  - mv command: file moving and renaming
  - rm command: file removing
- Handwritten notes made on all commands and flags

## What I learned

### history
- ↑ arrow recalls earlier commands for review/editing
- `!!` → run the most recent command again
- `!102` → run command number 102 from history
- `!cat` → run the most recent command that started with "cat"
- `history -c` clears the in-memory list | `history -w` writes to ~/.bash_history | `history -d &lt;offset&gt;` deletes an entry
- `clear` only clears the display. it does NOT erase bash history
- ⚠️ Commands get stored in history. never type passwords/tokens/secrets directly into commands

### cp
- Copies files/directories, leaving the source in place: `cp [option] source destination`
- Wildcards: `*` any sequence of chars | `?` any single char | `[ ]` any one char in brackets
  - e.g. `cp *.jpg /home/pete/pictures`
- Copying a directory needs recursion: `cp -r pim/ /home/pete/`
- `-a` = backup style, preserves links + attributes: `cp -a project/ project-backup/`
- Existing destination gets replaced by default control it with:
  - `-i` ask before overwrite | `-n` never overwrite | `-u` copy only if source is newer/missing
  - `-p` preserve permissions/ownership | `-f` force (removes destination first) | `-v` verbose

### mv
- Renames OR moves files (same command, two jobs)
- Same overwrite flags as cp: `-i` confirm, `-n` don't overwrite, `-b` make backup, `-v` verbose

### mkdir
- `ls -ld documents` to inspect an existing directory
- `-p` creates missing parent directories along the path
- `-m MODE` sets permissions at creation: `mkdir -m 755 public` → owner gets read/write/execute

### rm
- Removes files; `rm -r` for directories
- `-i` asks before deleting | `-f` ignores missing files and suppresses all prompts

## Key takeaway
- rm is permanent. no recycle bin in Linux. Think before hitting Enter, especially with `-rf`
