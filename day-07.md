# Day 7: Text Fu Continued: expand, join, sort, tr

Four more Text Fu lessons on Linux Journey:
- Lesson 10: expand and unexpand
- Lesson 11: join and split
- Lesson 12: sort
- Lesson 13: tr

## What I learned

### expand and unexpand
- A tab stores movement to the next tab stop, not a fixed number of spaces. How wide it looks depends on the current column and the tab-stop settings
- expand and unexpand convert between tabs and spaces while keeping those positions in mind
- `expand sample.txt` replaces tabs with the spaces needed to reach the tab stops and writes the result to stdout
- `-t NUMBER` places tab stops every NUMBER columns: `expand -t 4 sample.txt`
- `-i` converts only the tabs before the first nonblank character on each line
- expand does not edit its input file. Redirect to a different file to save: `expand sample.txt > result.txt`
- `unexpand` goes the other way. It replaces eligible spaces with tabs and keeps alignment at the chosen tab stops
- By default GNU unexpand converts only the initial blanks before the first nonblank character: `unexpand result.txt`
- `-a` considers suitable blanks throughout each line: `unexpand -a result.txt`

Extra notes:
- The default tab stop is 8 columns. If a file was built with 4 column stops, say so with `-t 4`, otherwise unexpand leaves four spaces as spaces
- `unexpand -t N` also switches on -a, so it converts blanks anywhere in the line
- `cat -A file` makes tabs visible as `^I` and line ends as `$`, which is the quickest way to see what is really in a file
- Never redirect to the file you are reading: `expand f.txt > f.txt` empties f.txt before expand starts
- Why it matters: Makefiles need real tabs, YAML does not allow tabs for indentation, and Python complains when tabs and spaces are mixed. cut -f and paste also use tab as the default separator, so turning tabs into spaces breaks them

### join
- join combines related records from two sorted text inputs, based on a shared field (the key)
- Both inputs must be ordered by the join field, using compatible comparison rules
- For the default field 1, prepare sorted copies with `sort -k 1,1`:

```bash
$ LC_ALL=C sort -k 1,1 people-raw.txt > people.txt
$ LC_ALL=C sort -k 1,1 surnames-raw.txt > surnames.txt
$ LC_ALL=C join people.txt surnames.txt
```

- `-1 FIELD` is the key field of the first file, `-2 FIELD` is the key field of the second file
- Example. The first input contains:

```
John 1
Jane 2
Mary 3
```

- The second contains:

```
1 Doe
2 Doe
3 Sue
```

- Sort the first file by field 2 and the second by field 1, then join:

```bash
$ join -1 2 -2 1 people.txt surnames.txt
1 John Doe
2 Jane Doe
3 Mary Sue
```

- `-t CHARACTER` sets the separator when a single nonblank character such as `:` separates the fields
- `-a 1` or `-a 2` also prints the unpaired lines from the first or second input. The default output has matched keys only

Extra notes:
- The output order is the key first, then the rest of file 1, then the rest of file 2. That is why `1 John Doe` starts with the number
- `-v 1` prints only the unpaired lines of file 1, which answers "what in A has no match in B?"
- `-o 0,1.2,2.2` picks the output columns (0 is the key, 1.2 is field 2 of file 1, 2.2 is field 2 of file 2)
- `-i` ignores case when comparing keys
- If the input is not sorted, join prints `is not sorted`, exits with an error and can miss matches, so always sort both files first with the same locale (LC_ALL=C on both)

### split
- split writes consecutive portions of one input into separate output files
- It is not the inverse of a key-based join. join combines records by key, split just cuts a file into pieces
- `split large.txt` writes up to 1000 lines per output file with the prefix x, giving names such as xaa, xab, xac
- `-l NUMBER` chooses the line count, and a final operand chooses the output prefix: `split -l 500 large.txt part-`
- `-b SIZE` divides by byte size. In GNU, the suffixes K, M and G are powers of 1024: `split -b 10M archive.bin chunk-`

Extra notes:
- `-d` gives numeric suffixes (part-00, part-01) instead of letters, and `-a N` sets the suffix length
- `--additional-suffix=.txt` keeps a file extension on every piece
- `-n 3` splits into 3 pieces instead of choosing a size
- A lone `-` as the input reads stdin: `cat big.txt | split -l 500 - part-`
- Put the pieces back together with `cat part-* > big.txt`. The suffixes sort in order, so the pieces come back in the right order. Check with `cmp` or `sha256sum`
- split does not touch the original file

### sort
- sort reads complete lines, orders them by the chosen comparison rules and writes the result to stdout
- It does not change the input file unless I explicitly choose an output operation
- Use a consistent locale such as `LC_ALL=C` when a script needs reproducible byte-order sorting
- `-r` reverses the comparison result
- Lexical order compares characters, so `10` comes before `2`. Use `-n` for normal numeric comparison
- `sort -nr scores.txt` compares numerically and puts the larger values first
- `-k START[,END]` chooses a key. Fields are separated by runs of blanks by default
- `-t ':'` sets the separator for colon separated records:

```bash
$ printf 'alice:30\nbob:8\ncarol:20\n' | sort -t ':' -k 2,2n
bob:8
carol:20
alice:30
```

- `-u` outputs one line for each equal comparison key. It sorts and removes duplicates under the selected comparison rules:

```bash
$ sort -u names.txt
```

- GNU `sort -o names.txt names.txt` safely handles its own output when I intentionally want the same file name:

```bash
$ sort -o names.txt names.txt
```

- Keep a backup, or write and verify a separate result, when the original data matters

#### sort -u vs sort | uniq
- `sort -u` sorts and removes duplicates in one step. Lines that compare equal under the chosen key count as duplicates
- `uniq` only removes ADJACENT duplicate lines, so its input must already be sorted. On unsorted data it leaves duplicates behind
- `sort | uniq` gives the same result as `sort -u` for plain duplicate removal
- Use `uniq` when I need extras that sort -u does not have: `uniq -c` counts each line, `uniq -d` shows only the lines that were repeated, `uniq -u` shows only the lines that appear once
- Careful with the name clash: `sort -u` means unique output, but `uniq -u` means only the lines that were never repeated

```bash
$ printf 'b\na\nb\na\n' | uniq
b
a
b
a
$ printf 'b\na\nb\na\n' | sort | uniq
a
b
$ printf 'b\na\nb\na\n' | sort -u
a
b
```

Extra notes on sort:
- Write keys as `-k 2,2` (start and end). A bare `-k 2` runs from field 2 to the end of the line and can sort on the wrong thing
- Put the type letters after the key: `-k 2,2nr` is numeric and reversed on field 2
- `-f` ignores case, `-b` ignores leading blanks, `-h` sorts human sizes such as 500, 1K, 2M (`du -h | sort -h`), `-V` sorts version numbers, `-c` only checks whether the input is already sorted
- In LC_ALL=C order, uppercase letters come before lowercase ones (A B a b)
- `sort file > file` is a trap. The shell empties the file before sort reads it, so the data is lost. Use `sort -o file file`
- With `-u -k`, only the first line of each group of equal keys is kept
- Recon use: `cat subs1.txt subs2.txt | sort -u` merges and de-duplicates lists of subdomains

### tr
- tr is short for translate. It translates, deletes or squeezes characters read from stdin
- It does not accept ordinary input file operands. Use a pipe or input redirection to give it data
- Syntax: `tr [OPTIONS] SET1 [SET2]`
- With two sets, characters in SET1 map by position to characters in SET2:

```bash
$ echo "hello world" | tr a-z A-Z
HELLO WORLD
```

- Translate one character into another:

```bash
$ echo "2026-06-23" | tr '-' '/'
2026/06/23
```

```bash
$ echo "abc123" | tr 'abc' 'ABC'
ABC123
```

- `-d` with one set removes every matching character:

```bash
$ echo "My address is 123 Main Street" | tr -d '0-9'
My address is  Main Street
```

```bash
$ echo "Hello, world!" | tr -d '[:punct:]'
Hello world
```

```bash
$ printf "one\ntwo\nthree\n" | tr -d '\n'
onetwothree
```

- `-s SET` replaces each run of a listed character with a single copy of that character:

```bash
$ echo "Hello      World,   how   are   you?" | tr -s ' '
Hello World, how are you?
```

```bash
$ printf "one\n\n\nTwo\n" | tr -s '\n'
one
Two
```

- Character classes make the intent clearer than hand written ranges in many locales:
  - `[:lower:]` lowercase letters
  - `[:upper:]` uppercase letters
  - `[:digit:]` digits
  - `[:alpha:]` letters
  - `[:alnum:]` letters and digits
  - `[:space:]` whitespace characters
  - `[:punct:]` punctuation characters
- `-c` complements SET1, meaning every character NOT in the set. With `-d` it keeps only the selected kinds of characters:

```bash
$ echo "user@example.com!" | tr -cd '[:alnum:]'
userexamplecom
```

- Several tr processes can be chained when separate stages are clearer:

```bash
$ echo "Hello,,,     world!!!" | tr -d '[:punct:]' | tr -s ' '
Hello world
```

```bash
$ printf "name\tlevel\npete\tbeginner\n" | tr '\t' ','
name,level
pete,beginner
```

- Because tr reads stdin, a file is provided with `<`:

```bash
$ tr '[:lower:]' '[:upper:]' < names.txt
```

Extra notes on tr:
- tr works on single characters, never on whole words. `tr 'abc' 'xyz'` swaps each letter separately (a to x, b to y, c to z), it does not replace the word "abc". Replacing words needs sed, which comes later
- If SET2 is shorter than SET1, GNU tr repeats the last character of SET2 (`echo abc | tr abc xy` gives `xyy`)
- `tr -cd '[:alnum:]'` also deletes the newline, which is why the output has no line ending. Keep it with `tr -cd '[:alnum:]\n'`
- Windows line endings (CRLF) break scripts and comparisons. Remove the carriage return with `tr -d '\r' < file > clean.txt`
- One word per line: `tr -s ' ' '\n' < file`, which pairs well with `sort | uniq -c | sort -rn`
- rot13 can be done with `tr 'A-Za-z' 'N-ZA-Mn-za-m'`. Running it twice returns the original text, and Bandit has a level that uses it
- Escapes such as `\n`, `\t` and `\r` work inside the sets. Quote the sets so the shell does not expand brackets or backslashes

## Key takeaways
- tr changes characters, cut picks columns, sort orders lines, join matches records, split cuts files. Pipes connect them all
- join only works on sorted input, and both files need the same sort rules
- uniq needs sorted input. sort -u does the sorting and the de-duplicating in one step
- Never redirect output onto the file being read. Use sort -o or a temporary file
- cat -A shows what is really inside a file: tabs, line endings and trailing spaces
