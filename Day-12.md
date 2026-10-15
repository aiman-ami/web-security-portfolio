# Day 12: Bandit 12 to 15, DNS, ping, dig and SSH config

- Bandit levels 12, 13, 14 and 15
- DNS lookups with `dig`
- `ping` to check whether a host is up
- SSH config file (`~/.ssh/config`)

Roadmap: 1C Networking. Ticked IP address, ports, DNS (L2), `ping` and `dig` (L2), and SSH: keys, ports, config basics (L2).

## What I learned

### Bandit level 12: a file packed in layers

Idea: data.txt is a hexdump of a file that was compressed many times. Each layer has to be identified and unpacked until the last file.

```bash
$ mkdir /tmp/abcd
$ cp data.txt /tmp/abcd
$ cd /tmp/abcd
$ xxd -r data.txt > data
$ file data
data: gzip compressed data
$ mv data data.gz
$ gzip -d data.gz
$ file data
data: bzip2 compressed data
$ mv data data.bz2
$ bzip2 -d data.bz2
$ tar xf data
```

How it works: `xxd -r` reverses the hexdump into the real file. `file` says what kind of file it is, so I know which tool to use next. `gzip` and `bzip2` need their own file ending, so I rename with `mv` first. `tar xf` extracts an archive and creates a new file. I repeated check, rename, unpack until the last file was plain text.

Extra notes:

- `tar` does not need a rename, only `gzip` and `bzip2` do.
- Working in a copy in `/tmp` keeps the original file untouched.

### Bandit level 13: a key instead of a password

Idea: the level gives an SSH private key for the next user. Logging in from the server to itself (`localhost`) is blocked, so the key has to be copied to my own machine first.

```bash
$ scp -P 2220 bandit13@bandit.labs.overthewire.org:sshkey.private .
$ chmod 600 sshkey.private
$ ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
```

How it works: `scp` copies a file over SSH. `chmod 600` makes the key readable and writable by me only, because SSH refuses a key that others can read. `-i` means use this key file to log in.

My notes: I could not log in to the server from itself, so I used `scp` to copy the key and logged in from my own machine.

Extra notes:

- `ssh` uses a lowercase `-p` for the port, and `scp` uses a capital `-P`.

### Bandit level 14: talking to a port

Idea: send the current level's password to port 30000 on localhost and the service there answers with the next password.

```bash
$ nc localhost 30000
```

How it works: `nc` (netcat) opens a plain connection to a port. The cursor waits with no prompt. I type the password, press Enter, and the program on that port replies.

My notes: a computer has numbered doors called ports, and `nc` lets us talk directly to the program behind one and write to it.

Extra notes:

- `localhost` means this same machine.
- A browser does the same thing on port 443 for HTTPS, but sends a full HTTP request for me.

### Bandit level 15: the same, with TLS

Idea: the same as level 14, but port 30001 only speaks TLS, the encryption behind `https://`. Plain `nc` does not work here.

```bash
$ openssl s_client -connect localhost:30001
```

How it works: `openssl s_client` is like `nc` with encryption added. `-connect` takes the host and port joined by a colon, with no spaces. Certificate text scrolls past, then the cursor waits. I type the password and press Enter.

Extra notes:

- I found the `-connect` option with `openssl s_client -help`.

### DNS: how a name becomes an address

```bash
$ dig example.com +short
$ dig example.com
example.com.            109     IN      A       104.20.23.154
example.com.            109     IN      A       172.66.147.243
```

(Answer section only, in two forms.)

- A domain name is the easy name for people. An IP address is the real address of the machine. DNS is the phone book that turns the name into addresses.
- The example gave two `A` records, so one name can have several IP addresses.
- The number 109 is the TTL, the seconds the answer may be remembered before asking again. `A` means the name maps to an IPv4 address.
- `dig` needed `sudo apt install bind9-dnsutils`. A 404 during the install was fixed with `sudo apt update` first.

Extra notes:

- DNS only returns the address. After that, the browser connects to that IP address on a port (443 for HTTPS) and the web server sends the page.

### ping: is the host up

```bash
$ ping -c 4 example.com
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
```

- `-c 4` sends 4 requests and stops.
- 0% packet loss means every request got a reply, so the host is up.
- `time 3005ms` is the total time of the run. The speed of one reply is on the last line (`rtt min/avg/max/mdev`). My average was 309 ms.

### SSH config basics

```bash
$ ls -la ~/.ssh
drwx------ ...
known_hosts
known_hosts.old
```

- `~/.ssh` is private to me (`drwx------`: only the owner has access).
- `known_hosts` is SSH's memory of servers I have connected to. `known_hosts.old` is its backup.
- There was no `config` file, so I created one with `vim ~/.ssh/config`:

```
Host bandit
    HostName bandit.labs.overthewire.org
    Port 2220
    User bandit15
```

```bash
$ ssh bandit
```

- SSH reads the config, finds the `Host bandit` block, and fills in the address, port and user. One short word replaces the long command.
- `Port 2220` tells SSH to connect to port 2220. Without it, SSH would use the default port 22 and the connection would fail.

## Key takeaways

- `file` shows what a file really is, so I know which tool unpacks it.
- `ssh -i` logs in with a key. The key must be `chmod 600`, and `scp` copies files over SSH.
- `nc` talks to a port in plain text. `openssl s_client -connect host:port` does the same with TLS.
- DNS turns a name into IP addresses and nothing more. The connection to the server is a separate step.
- `dig` shows the DNS answer, and `ping` checks whether the host answers.
- `~/.ssh/config` shortens a long SSH command to `ssh name`.
