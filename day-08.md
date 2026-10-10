# Day 8: Text Fu Finished: uniq, wc, nl, grep

The last three Text Fu lessons on Linux Journey, which completes the Text Fu section:
- Lesson 14: uniq
- Lesson 15: wc and nl
- Lesson 16: grep

Lab completed: Linux uniq command, duplicate filtering

## What I learned

### uniq
- uniq compares each input line with the preceding line. It can collapse, count or select groups of adjacent equal lines
- It does not search the whole file for duplicates that are separated from each other
- `-c` puts the number of consecutive input lines in front of each output group:

```bash
$ uniq -c reading.txt
      2 book
      2 paper
      2 article
      1 magazine
```

- `-u` prints only the groups that contain exactly one line:

```bash
$ uniq -u reading.txt
magazine
```

- `-d` prints one representative line from each adjacent group that has more than one line:

```bash
$ uniq -d reading.txt
book
paper
article
```

- `uniq -D` prints every line from the repeated groups, whereas lowercase `-d` prints each repeated group's value once
- If neighboring values all differ, nothing is collapsed
- Sort first when changing the order is acceptable and I want equal complete lines grouped together:

```bash
$ sort reading.txt | uniq
article
book
magazine
paper
```

- uniq reads stdin when no input file is named, which is why it fits naturally after sort
- GNU options such as `-i` ignore case, while `-f`, `-s` and `-w` skip or limit the compared regions. Use them only when equality should be decided by part of each line

Lab extras (options not covered in the lab):
- `-u`: display only unique lines (lines that appear exactly once)
- `-i`: ignore case when comparing lines
- `-f N`: skip the first N fields when comparing lines
- `-s N`: skip the first N characters when comparing lines

Extra notes:
- `-w N` compares no more than N characters of each line
- Typical pipeline for the most common lines: `sort file | uniq -c | sort -rn`
- The second file operand of uniq is an OUTPUT file. `uniq a.txt b.txt` overwrites b.txt, so redirect with `>` when I only want to save the result
- Name clash with sort: `sort -u` means unique output, `uniq -u` means only the lines that were never repeated

### wc
- wc counts properties of text streams. It reads files or stdin and sends its results to stdout
- With no count option, wc prints the number of newline characters, words and bytes, followed by the filename when one was supplied:

```bash
$ printf 'red blue\ngreen\n' > colors.txt
$ wc colors.txt
 2  3 15 colors.txt
```

- From left to right:
  1. `2` newline characters, reported as lines
  2. `3` whitespace-delimited words
  3. `15` bytes in this ASCII example
- Select only the measurement I need:
  - `-l`: count newline characters
  - `-w`: count words
  - `-c`: count bytes
  - `-m`: count characters according to the current locale
- When several files are named, wc prints one result per file and a `total` line
- GNU `wc -L` reports the maximum display width of an input line

Extra notes:
- Bytes and characters are different. In UTF-8 a character such as e with an accent takes 2 bytes, so `-c` and `-m` give different numbers. In plain ASCII they match
- `-l` counts newline characters, so a last line without a newline at the end is not counted
- With input redirection (`wc -l < file`) or a pipe, wc has no filename to print, so only the number appears
- The columns always come out in the same order (lines, words, characters, bytes), no matter what order I write the flags in

### nl
- nl writes its input with generated line numbers. It reads files or stdin
- By default it numbers the nonempty lines in the logical body of its input
- Suppose `notes.txt` contains a blank second line:

```text
alpha

beta
```

- The blank line is preserved but gets no number:

```bash
$ nl notes.txt
     1	alpha

     2	beta
```

- `-ba` selects body numbering style `a`, which numbers all lines, blank ones too:

```bash
$ nl -ba notes.txt
     1	alpha
     2	
     3	beta
```

Lesson goals:
- Interpret the default lines, words and bytes columns from wc
- Select one count with `-l`, `-w`, `-c` or `-m`
- Distinguish byte counts from character counts
- Number nonempty lines with default nl behavior
- Number blank lines too with `nl -ba`

Extra notes on nl:
- Body styles for `-b`: `a` all lines, `t` only nonempty lines (the default), `n` no lines, `pREGEX` only lines that match the regex
- `-n ln`, `-n rn` and `-n rz` set the number format (left aligned, right aligned, right aligned with zeros), `-w N` sets the number width, `-s STRING` sets the separator after the number
- `-v N` sets the first number and `-i N` sets the step
- Other ways to number lines: `cat -n` numbers all lines, `cat -b` numbers only nonempty lines, `grep -n` shows the original line numbers of matches

### grep
- grep selects input lines that match a pattern. It can search named files or stdin, print matching context, count selected lines and report whether a match was found through its exit status
- Pass a pattern followed by one or more input files:

```bash
$ grep 'fox' sample.txt
```

- By default GNU grep reads the pattern as a basic regular expression and prints every selected line
- Quote patterns so the shell does not interpret spaces and metacharacters first
- `-F` treats the pattern as a fixed string instead of a regular expression:

```bash
$ grep -F 'price: $5.00' products.txt
```

- Three commonly used pattern modes in GNU grep:
  - Default: basic regular expressions
  - `-E`: extended regular expressions, with operators such as `|`, `+` and `?` working without backslashes
  - `-F`: fixed strings with no regular expression operators
- Anchors such as `^` and `$` match the beginning and end of a line. To match names ending in the literal suffix `.txt` in a list:

```bash
$ grep -E '\.txt$' filenames.txt
```

- `-e PATTERN` supplies a pattern explicitly. It helps when the pattern starts with `-`, because quoting alone does not stop option parsing:

```bash
$ grep -e '-v' settings.conf
```

- Common options:
  - `-i`: ignore case distinctions
  - `-n`: prefix selected lines with line numbers
  - `-v`: select lines that do not match
  - `-c`: print the count of selected lines for each input file
  - `-o`: print only each nonempty matching part rather than the full line
- Example, count the lines containing `fox`, ignoring case:

```bash
$ grep -ic 'fox' sample.txt
```

- `-c` counts selected lines, not the total number of matches inside them. A line containing `fox fox` adds one to the count
- To count nonoverlapping match occurrences with GNU grep, `grep -o PATTERN | wc -l` is one possible pipeline
- When no input file is named, grep reads stdin and fits naturally in a pipeline:

```bash
$ env | grep '^USER='
```

- `-r` searches readable files recursively below a directory:

```bash
$ grep -r 'listen_port' config/
```

- Exit status: `0` when at least one line is selected, `1` when no line is selected, `2` for an error. This lets scripts tell "no match" apart from an unreadable file or an invalid pattern
- Options such as `-q` suppress normal output and stop after a match is found, which is useful for condition checks
- Do not judge success from an empty display alone. `-q`, redirection, no match and an error can all produce little or no stdout, while their statuses differ

Extra notes on grep:
- More than one pattern: `grep -e error -e warning log.txt` (a line matching either is selected)
- `--` ends option parsing, so `grep -- '-v' file` also works for patterns starting with a dash
- `-w` matches whole words only, `-x` matches whole lines only
- Context around a match: `-A N` lines after, `-B N` lines before, `-C N` lines on both sides
- `-l` lists the files that contain a match, `-L` lists the files that do NOT
- `-s` hides error messages about missing or unreadable files
- Use `-F` whenever the text has characters such as `.`, `*` or `$` that I want taken literally
- `echo $?` right after a command shows its exit status
- In a script: `if grep -q 'root' /etc/passwd; then echo found; fi`
- `ps aux | grep nginx` also matches the grep command itself. Writing `grep '[n]ginx'` avoids that
- Hunting for secrets in code and config I am allowed to test: `grep -rniE 'password|secret|token|api[_-]?key' .`

## Text Fu complete
- Lessons 1 to 16 are done: redirection, pipes, environment variables, cut, paste, head, tail, expand, join, split, sort, tr, uniq, wc, nl, grep
- The cheat sheet now covers the full section

## Key takeaways
- uniq only sees adjacent lines, so sort first. grep picks lines, cut picks columns, tr changes characters, wc counts, nl numbers
- grep -c counts lines, not matches. grep -o piped into wc -l counts matches
- A pattern that starts with a dash needs -e. A pattern with special characters needs -F
- Check the exit status of grep (0, 1, 2) instead of trusting an empty screen
- wc -c counts bytes, wc -m counts characters, and they differ for non-ASCII text
