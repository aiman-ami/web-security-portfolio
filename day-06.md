# Day 6: Practice Day and Text Fu

- Started with cmdchallenge to test everything I learned this week
- Solved the challenges for commands I knew, then hit walls on new ones and learned each one on the spot
- New commands and traps I learned from the challenges:
  - tail and head with the n flag, print last or first lines of a file
  - ln -s for symbolic links, order is target first, link name second
  - find with delete, mindepth and maxdepth
  - the hidden files trap, the * wildcard never matches dotfiles, so rm -rf * always leaves them behind
  - the .[!.]* pattern targets dotfiles without touching . and ..
  - grep flags l, h, o, E and the rule that include only works together with r
  - basic regex, [0-9] for digits, {1,3} for repeats, ^ anchors to start of line
  - find -name "*.doc" -delete deletes files recursively by extension
  - grep -l lists filenames only, grep -o prints only the matching part
  - counting with pipes, find . -maxdepth 1 -type f | wc -l counts files in a directory

- Then started the Text Fu section of Linux Journey:
  - Lesson 1: stdout, stderr, stdin. The &gt; and &gt;&gt; operators, 2&gt; for errors, &&gt; for both, tee, and input redirection with &lt;
  - Lab redirecting input and output: &gt; overwrites without warning, &gt;&gt; appends, 2&gt; catches errors only, &&gt; catches both, tee splits output so I see it and save it, &lt; feeds a file as input
  - Lesson 2: stdin deep dive. File descriptors 0, 1, 2. wc file vs wc &lt; file, the difference between an argument and a stream. cat &lt; in &gt; out chains two redirections
  - Lesson 3: stderr and /dev/null, the black hole device that discards anything written to it
  - Lesson 4: pipes and tee. The | operator connects stdout of the left command to stdin of the right, tee -a appends, tee can sit in the middle of a pipeline to save intermediate output and keep processing
  - Lesson 5: environment variables. Every process inherits name value strings from its parent process. export marks a variable to be inherited. PATH is the colon separated list of folders the shell searches for commands. Never replace PATH by accident and never add untrusted writable directories. Inline assignment like LANG=C sort names.txt affects only that one command. env -i starts a command with an empty environment
  - Lab manage shell environment: echo $$ shows my shell PID, ps -f shows detailed info including PPID, local vs environment variables, aliases do not inherit into child shells by default, set -o allexport auto exports new variables, .bashrc and .zshrc make settings permanent, noclobber prevents accidental overwrite with &gt;
  - Lesson 6: cut. cut -c picks characters, cut -f picks fields, cut -d sets a custom delimiter, cut -s skips lines without the delimiter, reads from stdin so it fits naturally in pipes
  - Lesson 7: paste. Joins lines from files as columns, -d sets the separator, -s joins a whole file into one line, - reads from stdin, uneven files produce empty fields
  - Lesson 8 and 9: head and tail. head -n and tail -n set line counts, head -c uses bytes, tail -n +N starts at line N and prints to the end, tail -f follows a file live which is the classic log watching command, tail -F follows by name so it survives log rotation

- Decision of the day: handwritten notes take too much time. From now on my notes go straight into the repo md files and the cheat sheet

## Key takeaways
- When a task says recursively, think find or grep -r
- cut picks columns, grep picks lines, pipes connect everything
- 2&gt;/dev/null silences error floods, tee saves output while I watch it live
- tail -f on a log file is how I will watch things happen in real time as a pentester
- The hidden files trap got me twice today, dotfiles escape ls and wildcards, ls -a and find beat them
