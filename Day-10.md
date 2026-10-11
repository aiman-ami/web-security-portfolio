# Day 10: Command Challenges and Bandit Levels 3 to 10

- Command line challenges (cmdchallenge): solved the ones on topics I already knew, and learned new commands from the harder ones
- Bandit levels 3 to 10 (redid levels 3 to 5 after losing those passwords)

Roadmap: Bandit levels 5 to 10 are done (1B).

## Command line challenges

New commands from these: `sed`, `basename`, `dirname`, `bc`, `sh -c`, and the `${var%.*}` trick.

### Delete a phrase from all text files, recursively
```bash
find . -type f -name "*.txt" -exec sed -i 's/challenges are difficult//g' {} \;
```
- `find` picks the files, `sed` edits the text inside each one
- `-exec ... {} \;` runs the command once per file. `{}` becomes the file path, `\;` ends the command
- `sed -i 's/old/new/g'` replaces text in place. With nothing between the last two slashes, the phrase is deleted. `g` means every match on a line
- `rm -rf **/*.txt` is the wrong answer. It deletes the files instead of editing them, and `**` only works when `globstar` is on

### Print the sum of numbers in a file (one per line)
```bash
paste -sd+ sum-me.txt | bc
```
- `paste -sd+` joins all lines into one line using `+` as the separator (for example `3+5+10`)
- `bc` is a command line calculator. It reads the text and prints the result

### Print all files recursively without the leading directory path
```bash
find . -type f -exec basename {} \;
```
- `basename` strips the directory part and keeps only the file name
- `find . -type f -printf '%f\n'` does the same in one step with GNU find

### Remove the extension from all files, recursively
```bash
find . -type f -exec sh -c 'mv "$0" "${0%.*}"' {} \;
```
- `-exec` runs a program without a shell, so it cannot read shell syntax. `sh -c` starts a small shell to run the text
- With `sh -c 'text' something`, the first argument after the text becomes `$0`. Here that is the file path from `{}`
- `${0%.*}` removes the shortest match of `.anything` from the end of the value, so `./docs/report.txt` becomes `./docs/report`
- The single quotes stop my own shell from expanding `$0` too early
- A file with no extension would break this, because the only dot it finds is the one in `./`

### List files with spaces replaced by dots
```bash
ls | tr " " "."
```

### Print directories that contain .tf files, one per line, no duplicates
```bash
find . -name "*.tf" -exec dirname {} \; | sort -u
```
- `dirname` is the opposite of `basename`. It keeps only the directory part
- `sort -u` removes the duplicates, since a folder with three .tf files would otherwise appear three times

### Print files that start with a number, recursively
```bash
find . -type f -name "[0-9]*" -exec basename {} \;
```

## Bandit levels 3 to 10

### Level 3
- The file is hidden, so a normal `ls` does not show it
- `ls -a` shows dotfiles. Hidden file names cannot be guessed, so I read the exact name from the listing and then use `cat`
- `.` and `..` are the current and parent directory, not files to read

### Level 4
```bash
file ./*
```
- `file` tells what type each file really is. The human-readable file says `ASCII text`, the others say `data`
- File names start with a dash, so `./` in front is needed

### Level 5
```bash
find . -type f -size 1033c
```
- `c` means bytes in `-size`. The level's other clue (not executable) can narrow it down: `! -executable`

### Level 6
```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```
- I first wrote `33b` by mistake. In `find -size`, `b` means 512-byte blocks, not bytes, so nothing matched. The right letter is `c`
- `find /` searches the whole server and complains about every folder I cannot read. `2>/dev/null` throws those error messages away so only real matches show

### Level 7
```bash
grep "millionth" data.txt
```
- grep prints the whole line, so copy only the password part next to the word

### Level 8
```bash
sort data.txt | uniq -u
```
- `uniq -u` only prints lines that appear exactly once, and `uniq` only sees adjacent lines, so `sort` has to come first

### Level 9
```bash
strings data.txt | grep "=="
```
- The file is mostly binary junk. `strings` prints only the readable text, and `grep` keeps the lines with the `==` marker

### Level 10
```bash
base64 -d data.txt
```
- base64 is an encoding, not encryption. Anyone can reverse it with `-d`. It shows up constantly in tokens, headers and cookies in web security

## Key takeaways
- `find` picks the files, `-exec` runs a command on each. This pattern solved most of the challenges
- `sed` edits words and phrases, `tr` only swaps single characters
- In `find -size`, `c` is bytes, `b` is 512-byte blocks, `k` is kilobytes. One wrong letter finds nothing
- `2>/dev/null` hides the Permission denied flood when searching from `/`
- Hidden file names have to be found with `ls -a`, not guessed
- Passwords go in a notes file outside the repo, never in the repo
- I did levels 5 to 10 using only commands I already knew from Text Fu, plus `strings` and `base64` as new ones
