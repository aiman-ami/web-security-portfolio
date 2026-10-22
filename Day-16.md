# Day 16: Bandit 25 to 32, Git from clone to push

- Bandit levels 25 to 32, solved mostly solo
- Escaped a restricted shell through a pager and vim
- Used git clone, log, show, branch, checkout, tag, add, rm and push

Roadmap: 1B Bandit levels 25 to 32 done. 1D Git: clone, log, show, branch, checkout, tag, add, commit, push, all used hands-on (L2, close to L3 given most of this was solo).

## What I learned

### Level 25: escaping a restricted shell

Idea: bandit26's login shell is not bash, so a normal login closes immediately. The shell is a text pager that quits once it finishes showing a file.

```bash
$ scp -P 2220 bandit25@bandit.labs.overthewire.org:bandit26.sshkey .
$ chmod 600 bandit26.sshkey
$ ssh bandit26@bandit.labs.overthewire.org -p 2220 -i bandit26.sshkey
```

Making the terminal window very small before connecting kept the pager from finishing, since it had no room to show the whole file at once. That left it stopped partway through, waiting for input.

```
v
:set shell=/bin/bash
:shell
```

How it works: pressing `v` inside the pager opens the current file in vim. Vim can run shell commands on its own, so `:set shell=/bin/bash` tells it which shell to use, and `:shell` opens that shell for real, breaking out of the restricted login entirely.

My notes: the key had to be copied to my own machine first, with the same localhost block as level 13.

### Level 26: setuid binary

Already logged in as bandit26 from the shell escape above, no new login needed.

```bash
$ ls -la
$ ./bandit27-do cat /etc/bandit_pass/bandit27
```

Same pattern as level 19 (`bandit20-do`): `bandit27-do` has the setuid bit, owned by bandit27, so running it reads bandit27's password file directly.

### Level 27: git clone over SSH

Run from my own machine, not inside a Bandit session, since the level goal said so.

```bash
$ mkdir /tmp/b27
$ cd /tmp/b27
$ git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
$ ls repo
$ cd repo
$ cat README
```

How it works: `git clone` copies a whole repository, history included, from a remote source. The password for logging into bandit27-git over SSH was the same as the bandit27 login password. The password for level 28 was sitting directly in the README.

### Level 28: git log and git show

```bash
$ git log
$ git show COMMIT_HASH
```

There were three commits. `git log` lists them in order, newest first. The password had been removed in the newest commit, so `git show` on the previous commit's hash revealed the file as it looked before the removal, which still had the password in it.

### Level 29: git branch and checkout

```bash
$ git log
$ git branch -a
$ git checkout dev
$ cat README.md
```

The password was not in any old commit this time. `git branch -a` listed all branches, including ones not currently checked out. `git checkout dev` switched to a branch called `dev`, and its own README.md held the password.

### Level 30: git tag

```bash
$ cat README.md
$ git tag
$ git show secret
```

The README only said "just an empty file... muahaha", a dead end on purpose. `git tag` listed the tags in the repo, one of them named `secret`. A tag points at a specific commit, and `git show` on a tag works the same as on a commit hash, revealing its content, which held the password.

### Level 31: git add, rm, commit, push

```bash
$ vim key.txt
$ cat .gitignore
$ rm .gitignore
$ git add key.txt
$ git commit -m "add key"
$ git push
```

How it works: the repo had a `.gitignore` file listing `*.txt`, which tells git to ignore any `.txt` file and never track it, so `key.txt` would not be picked up by `git add` while that rule existed. Deleting `.gitignore` removed the rule. `git add` stages the file, `git commit` saves it into the repo's history, and `git push` sends that commit back to the server. The password for level 32 was given after the push succeeded.

### Level 32: uppercase shell

```bash
$0
$ cat /etc/bandit_pass/bandit33
```

The shell on login only accepted uppercase input and converted everything else before running it, so normal lowercase commands like `cat` did not work directly. `$0` is a variable that always holds the name of the currently running shell program. Running `$0` on its own starts a fresh copy of that same shell, but one that behaves normally this time, which was enough to run `cat` properly afterward.

## Key takeaways

- A restricted shell can sometimes be escaped through a side door, like a pager's own editor command, rather than the login itself.
- Setuid binaries (the `s` in permissions) keep showing up as a shortcut to another user's files.
- `git clone` copies a full repository and its history, not just the current files.
- `git log` and `git show` can reveal content that was removed in a later commit, since old commits are never deleted automatically.
- `git branch -a` and `git checkout` move between different lines of work in the same repo, and each branch can hold different files.
- `git tag` marks a specific commit with a name, and works with `git show` just like a commit hash does.
- `.gitignore` stops matching files from being tracked, so a tracked file has to not be listed there first.
- `git add`, `git commit` and `git push` are the three steps to save a change and send it to the remote repository.
