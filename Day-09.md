# Day 9: Advanced Text-Fu: Regex, Vim, Nano and Emacs

The Advanced Text-Fu section on Linux Journey:
- Lesson 1: regex
- Lesson 2: text editors
- Lesson 3: Vim
- Lesson 4: Vim search patterns
- Lesson 5: Vim navigation
- Lesson 6: Vim inserting and appending text
- Lesson 7: Vim editing
- Lesson 8: Vim saving and quitting
- Lesson 9: Emacs

Lab completed: Edit Text Files in Linux with Vim and Nano

Roadmap: the vim item is done, which completes 1A Linux.

## What I learned

### Lesson 1: regex
- Regular expressions (regex) describe text patterns. grep, sed and awk use regex, but the supported syntax can differ, so always identify the tool and the regex flavor
- Regex is different from shell pathname expansion. In a regex, `*` repeats the preceding item. In a shell glob, `*` is itself a wildcard for a string of pathname characters
- Outside a bracket expression, `^` at the beginning of a pattern anchors the match at the beginning of a line:

```text
^by
```

- `$` matches at the end of a line:

```text
seashore$
```

- Combine both anchors when the entire line must fit the pattern:

```text
^by the seashore$
```

- The dot matches one character in ordinary line-oriented regex mode:

```text
b.
```

- A bracket expression matches one character from a specified set:

```text
s[ae]lls
```

- When `^` is the first character after `[`, it negates the set:

```text
s[^e]lls
```

- Ranges describe the characters between two endpoints:

```text
d[a-c]g
```

- Character classes such as `[[:lower:]]`, `[[:upper:]]` and `[[:digit:]]` often express intent more clearly
- In both BRE and ERE, `*` means zero or more repetitions of the preceding item:

```text
seashells*
```

- This matches seashell followed by zero or more additional s characters
- In ERE mode with `grep -E`, common operators include:
  - `+`: one or more repetitions
  - `?`: zero or one repetition
  - `|`: either the expression on the left or the right
  - `(...)`: group expressions
- Example:

```bash
$ grep -E '^(cat|dog)s?$' animals.txt
```

Extra notes:
- BRE is the default grep flavor and ERE is `grep -E`. For anything beyond simple patterns, use `-E`
- A literal dot needs a backslash (`\.`), or use `grep -F` for plain text
- `.*` means any run of characters
- `{n,m}` repeats the previous item n to m times in ERE: `grep -E '[0-9]{1,3}' file.txt`
- Remove blank lines with `grep -v '^$' file.txt`. Hide comment lines with `grep -v '^#' file.conf`

### Lesson 2: text editors
- Do not assume Vim or Emacs is installed. Check command resolution in the current shell:

```bash
$ command -v vim
/usr/bin/vim
$ command -v emacs
/usr/bin/emacs
```

- Vim is a modal editor. The same key can mean different things depending on the current mode:
  - Normal mode interprets keys as navigation and editing commands
  - Insert mode inserts typed text
  - Command-line mode accepts commands such as writing or quitting
- Emacs commonly uses key combinations and named commands within an extensible environment. Files are visited in buffers, and major and minor modes customize behavior for different content and tasks. Emacs can run in a terminal or a graphical frame
- Many command-line programs check `VISUAL` or `EDITOR` when they need to start an editor. To choose Vim for commands launched from the current Bash session and its children:

```bash
$ export VISUAL=vim
$ export EDITOR="$VISUAL"
```

- Learn on a disposable file in a directory I own:

```bash
$ printf 'first line\nsecond line\n' > editor-practice.txt
$ vim editor-practice.txt
```

- Avoid starting with system configuration or another user's data. Make a backup before changing an important file, know how to save and exit, and review the result with a read-only command such as `cat` or `diff`

### Lab: Edit Text Files in Linux with Vim and Nano
- The primary way to move the cursor in vi is with the `h`, `j`, `k` and `l` keys. This is a core skill for any vi user:
  - `h` moves the cursor one character to the left
  - `l` moves the cursor one character to the right
  - `j` moves the cursor one line down
  - `k` moves the cursor one line up
- Use nano when:
  - I am new to Linux text editing
  - I am making quick, simple edits
  - I work on configuration files occasionally
  - I prefer a straightforward, intuitive interface
- Use vi or vim when:
  - I am doing extensive programming or text manipulation
  - I am working on remote servers (vi is almost always available)
  - I need advanced features like macros, plugins, or complex search and replace
  - Efficiency and speed matter once I have learned the commands

### Lesson 3: Vim (Vi Improved)
- Vim is a configurable text editor. The name means Vi Improved
- It keeps the modal editing model of the original vi editor and adds multilevel undo, syntax support, scripting and an extensive help system
- Help tags are precise, so punctuation can matter. `Ctrl+]` on a help link follows it and `Ctrl+T` returns

### Lesson 4: Vim search patterns
- In Normal mode, type `/`, enter a pattern and press Enter. Vim moves to the next match after the cursor:

```text
/pretty
```

- Searches use Vim's regular expression syntax, so characters such as `.`, `*`, `[` and `\` can have special meaning. Start a pattern with `\V` when the rest should be treated as very nomagic (plain text), or escape special characters deliberately
- `?` followed by a pattern and Enter moves to the preceding match before the cursor:

```text
?pretty
```

- This does not mean "the final match in the file". The result depends on the current cursor position. With Vim's default `wrapscan` setting a search can wrap at the beginning or end. `:set nowrapscan` turns the wrapping off
- After either kind of search:
  - `n` repeats in the original search direction
  - `N` repeats in the opposite direction
- In Normal mode, with the cursor on a word:
  - `*` searches forward for that whole word
  - `#` searches backward for that whole word
- Options that change case behavior:
  - `:set ignorecase` makes searches ignore case
  - `:set smartcase` makes an uppercase character restore case sensitivity when `ignorecase` is also set
  - `\c` inside a pattern forces that search to ignore case
  - `\C` forces that search to respect case
- When search highlighting is on, `:nohlsearch` clears the current highlights without deleting the search pattern. The next search or repeat can highlight matches again

Extra note: replace everywhere in a file with `:%s/old/new/g`

### Lesson 5: Vim navigation
- The foundational Normal mode motions:
  - `h`: one character left
  - `j`: one screen line down
  - `k`: one screen line up
  - `l`: one character right
- On a wrapped display line, `j` and `k` normally move by file lines. `gj` and `gk` move by displayed screen lines
- Word motions:
  - `w`: beginning of the next word
  - `b`: beginning of the current or previous word
  - `e`: end of the current or next word
- Positions on the current line:
  - `0`: column zero
  - `^`: first nonblank character
  - `$`: end of the line
- Larger jumps:
  - `gg`: first line
  - `G`: final line
  - `42G`: line 42
  - `Ctrl+F`: forward about one screen
  - `Ctrl+B`: backward about one screen

### Lesson 6: Vim inserting and appending text
- From Normal mode:
  - `i`: enter Insert mode before the cursor
  - `a`: enter Insert mode after the cursor
- Uppercase commands target meaningful positions on the current line:
  - `I`: enter Insert mode before the first nonblank character
  - `A`: enter Insert mode at the end of the line
- Also from Normal mode:
  - `o`: open a new line below the current line and enter Insert mode
  - `O`: open a new line above the current line and enter Insert mode

### Lesson 7: Vim editing
- The general form is:

```text
[count] operator [count] motion
```

- Common operators:
  - `d`: delete text
  - `c`: change text, then enter Insert mode
  - `y`: yank (copy) text
- Convenient shortcuts:
  - `x`: delete the character under the cursor
  - `dd`: delete the current line (linewise)
  - `3dd`: delete three lines starting with the current line
  - `cc`: change the current line and enter Insert mode
  - `r{char}`: replace the character under the cursor with {char}
  - `R`: enter Replace mode until Esc is pressed
- The `c` operator removes the selected text and enters Insert mode so I can type a replacement:
  - `ce`: change through the end of the word
  - `c$`: change through the end of the line
  - `cc`: change the complete current line
  - `ciw`: change the inner word under the cursor
  - `caw`: change a word text object, including the surrounding spacing as Vim defines it
- Vim calls copying "yanking" and pasting "putting":
  - `yw`: yank through a word motion
  - `yy`: yank the current line
  - `p`: put after the cursor for characterwise text, or below the current line for linewise text
  - `P`: put before the cursor, or above the current line
- Other Normal mode commands:
  - `u`: undo the most recent change
  - `Ctrl+R`: redo an undone change
  - `.`: repeat the most recent change at the current location when applicable
  - `J`: join the current line with the next line

### Lesson 8: Vim saving and quitting
- `:w copy.txt` writes the current buffer to another pathname while keeping the buffer's existing name
- `:saveas copy.txt` is used when the buffer should adopt the new pathname
- The basic four:
  - `:w` write (save)
  - `:q` quit
  - `:wq` write and quit
  - `:q!` quit and discard unsaved changes
- `:x` writes the buffer only if it is modified, then quits. In Normal mode, uppercase `ZZ` does the same:

```text
:x
ZZ
```

- When several windows or buffers are open, a command may close only the current window
- `:qa`, `:wqa` and `:qa!` act across all windows. Review every modified buffer before using an all-windows force command

### Lesson 9: Emacs
- GNU Emacs is an extensible text editor customized with Emacs Lisp. It supports plain text editing, programming modes, file and buffer management and many optional packages. I can learn its core editing commands without adopting every extension
- Start Emacs with its normal display selection:

```bash
$ emacs
```

- In a graphical session this may open a graphical frame. `-nw` (no window system) keeps Emacs inside the current terminal:

```bash
$ emacs -nw
```

- Emacs uses related but different objects:
  - A buffer holds text or other editor state. A visited file's content lives in a buffer
  - A window is an area within an Emacs frame that displays a buffer
  - A frame is a top-level Emacs display, such as a graphical frame or a terminal frame
- Emacs documentation uses compact notation:
  - `C-x` means hold Control and press x
  - `M-x` means hold Meta and press x (Alt commonly acts as Meta in modern terminals and desktops)
  - `C-x C-f` is a key sequence: press Control+x, then Control+f
- Inside Emacs, `C-h t` opens the interactive tutorial. It teaches movement, insertion, saving and quitting in a safe practice buffer
- `C-h` is the help prefix. `C-h C-h` shows help about using help

Extra notes:
- Basic survival keys: `C-x C-f` open a file, `C-x C-s` save, `C-x C-c` quit, `C-g` cancel the current command
- Emacs is awareness only (L1) for my goals. nano and vim are enough

## Key takeaways
- Regex describes patterns, and the flavor depends on the tool. grep uses BRE by default and ERE with `-E`
- A regex `*` repeats the previous item, a shell glob `*` matches filenames. They are not the same thing
- Vim has three modes. If I am lost, press Esc to get back to Normal mode
- The survival set: `i` to type, `Esc` to stop, `:wq` to save and quit, `:q!` to quit without saving
- Operators plus motions (`d`, `c`, `y` with `w`, `$`, `iw`) are the core idea of Vim editing
- On a remote server, vim is almost always there, so it is worth knowing at survival level
- Do not assume an editor is installed. Check with `command -v`
