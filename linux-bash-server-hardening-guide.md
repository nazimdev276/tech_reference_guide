# The Complete Linux & Bash Mastery Guide
### From First Login to Production Hardening

> A comprehensive reference covering Linux fundamentals, Bash scripting, system administration, server management, and security hardening — from beginner basics to advanced production techniques.

---

## Table of Contents

1. [Introduction & How to Use This Guide](#part-1-introduction)
2. [Linux Fundamentals](#part-2-linux-fundamentals)
3. [The Shell & Bash Basics](#part-3-shell-and-bash-basics)
4. [Essential Command Line Tools](#part-4-essential-commands)
5. [Advanced Bash Scripting](#part-5-advanced-bash-scripting)
6. [Text Processing Mastery (grep, sed, awk, regex)](#part-6-text-processing)
7. [Process, Job & Resource Management](#part-7-process-management)
8. [Users, Permissions & Access Control](#part-8-users-permissions)
9. [systemd, Services & Logging](#part-9-systemd-services-logging)
10. [Package Management](#part-10-package-management)
11. [Storage, Disks & Filesystems](#part-11-storage-filesystems)
12. [Networking Deep Dive](#part-12-networking)
13. [Server Management](#part-13-server-management)
14. [Security Hardening](#part-14-security-hardening)
15. [Monitoring, Performance & Tuning](#part-15-monitoring-performance)
16. [Automation: Cron, Timers, Config Management](#part-16-automation)
17. [Backup & Disaster Recovery](#part-17-backup-recovery)
18. [Containers Primer](#part-18-containers)
19. [Troubleshooting Methodology](#part-19-troubleshooting)
20. [Advanced Tips, Tricks & One-Liners](#part-20-tips-tricks)
21. [Appendix: Cheat Sheets](#part-21-appendix)

---

<a id="part-1-introduction"></a>
## Part 1: Introduction & How to Use This Guide

This guide is written in layers. Each part opens with a short **beginner foundation** — just enough that nothing later feels unexplained — and then goes deep into the **advanced material** that actually matters once you're running real systems: production servers, hardened infrastructure, and automation at scale.

A few ground rules before we start:

- **Read commands, don't just paste them.** Every command below is explained. Understanding *why* a flag exists is what separates someone who can administer a server from someone who can only follow a tutorial.
- **Test in a VM or container first.** Several sections here touch firewall rules, SSH configuration, and kernel parameters. A mistake in these areas can lock you out of a remote box. Always keep a second session open when testing anything network- or auth-related.
- **Distro differences matter.** This guide is written distro-agnostic where possible, but calls out differences between Debian/Ubuntu (`apt`), RHEL/Fedora/Rocky (`dnf`/`yum`), and Arch (`pacman`) where they diverge meaningfully.
- **"Advanced" here means production-grade**, not obscure trivia. The advanced sections focus on things you'll actually use: writing robust scripts that don't silently fail, hardening a public-facing server, debugging a system under load, and automating repeatable operations.

---

<a id="part-2-linux-fundamentals"></a>
## Part 2: Linux Fundamentals

### 2.1 What Linux Actually Is

Linux is a **kernel** — the core program that talks to hardware, manages memory, schedules processes, and enforces permissions. What people call "Linux" (Ubuntu, Debian, Fedora, Arch) is a **distribution**: the kernel plus a package manager, init system, shell, and userland tools (GNU coreutils, systemd, etc.) bundled together.

Everything in Linux is designed around a few unifying ideas:

- **Everything is a file.** Devices, sockets, processes, and kernel parameters are all represented as files somewhere in the filesystem (`/dev`, `/proc`, `/sys`).
- **Small tools, composed together.** Instead of one giant program, Unix philosophy favors small single-purpose tools (`grep`, `sort`, `cut`) chained together with pipes.
- **Text is the universal interface.** Config files, logs, and command output are almost always plain text, which is why text-processing tools (Part 6) are so central to Linux mastery.

### 2.2 The Filesystem Hierarchy Standard (FHS)

| Path | Purpose |
|---|---|
| `/` | Root of the filesystem tree |
| `/bin`, `/usr/bin` | User executable binaries (often symlinked together on modern distros) |
| `/sbin`, `/usr/sbin` | System binaries, typically for root (`fdisk`, `iptables`) |
| `/etc` | System-wide configuration files |
| `/home` | User home directories |
| `/root` | Root user's home directory |
| `/var` | Variable data: logs (`/var/log`), mail, spool, databases |
| `/tmp` | Temporary files, usually cleared on reboot |
| `/opt` | Optional/third-party software packages |
| `/usr` | User-installed software and libraries (the bulk of the system) |
| `/lib`, `/usr/lib` | Shared libraries |
| `/proc` | Virtual filesystem exposing kernel/process info (not real files on disk) |
| `/sys` | Virtual filesystem exposing kernel/device/driver info, tunable at runtime |
| `/dev` | Device files (`/dev/sda`, `/dev/null`, `/dev/tty`) |
| `/boot` | Kernel images, bootloader config (GRUB) |
| `/mnt`, `/media` | Mount points for filesystems (manual vs removable media) |
| `/srv` | Data for services hosted on this system (web roots, repos) |

Knowing this layout is the difference between guessing and knowing exactly where to look when something breaks.

### 2.3 The Shell Prompt and Basic Navigation

```bash
pwd                 # print working directory
ls -la               # list all files, long format, including hidden
cd /var/log          # change directory
cd ~                 # go home
cd -                 # go to previous directory
tree -L 2            # show directory tree, 2 levels deep (install: tree package)
```

Absolute paths start with `/`; relative paths don't. `.` is the current directory, `..` is the parent. `~` expands to `$HOME`.

### 2.4 File Basics

```bash
touch file.txt              # create empty file / update timestamp
mkdir -p a/b/c               # create nested directories in one go
cp -r src/ dst/               # copy recursively
mv old.txt new.txt            # move/rename
rm -rf dir/                   # remove recursively, force (DANGEROUS — see 2.5)
cat file.txt                  # print file contents
less file.txt                 # paginated viewer (q to quit, / to search)
head -n 20 file.txt            # first 20 lines
tail -n 20 file.txt            # last 20 lines
tail -f /var/log/syslog        # follow file as it grows (live logs)
```

### 2.5 A Word on `rm -rf`

This is the single most dangerous command beginners run. `rm -rf /` (or worse, `rm -rf /$VAR` where `$VAR` is accidentally empty) deletes the entire filesystem with no confirmation. Modern GNU `rm` refuses `rm -rf /` by default, but `rm -rf /some/path/` where a variable expands to empty is still catastrophic. Always:

```bash
# Bad — if $TARGET is unset/empty, this becomes rm -rf /*
rm -rf "$TARGET"/*

# Safer — fail loudly if the variable is empty
set -u
: "${TARGET:?TARGET must be set}"
rm -rf "${TARGET:?}"/*
```

We'll cover `set -u` and parameter expansion guards like `${VAR:?}` in depth in Part 5.


### 2.6 Permissions Basics

Every file has an owner, a group, and permission bits for owner/group/others:

```bash
ls -l file.txt
# -rw-r--r-- 1 alice devs 1024 Sep  5 10:00 file.txt
```

Reading `-rw-r--r--`: type (`-` = file, `d` = dir, `l` = symlink), then three triplets: owner (`rw-`), group (`r--`), others (`r--`).

```bash
chmod 644 file.txt        # rw- r-- r--  (numeric: read=4, write=2, exec=1)
chmod u+x script.sh       # add execute for owner
chmod -R 755 dir/         # recursive
chown alice:devs file.txt # change owner and group
```

Numeric permissions are additive per triplet: `7 = rwx`, `6 = rw-`, `5 = r-x`, `4 = r--`. We go much deeper into special bits (setuid, setgid, sticky) and ACLs in Part 8.

---

<a id="part-3-shell-and-bash-basics"></a>
## Part 3: The Shell & Bash Basics

### 3.1 What a Shell Is

The shell is the program that reads what you type, interprets it, and runs commands. Bash (Bourne Again SHell) is the default on most Linux distros. Other shells exist (`zsh`, `dash`, `fish`) but Bash is the lingua franca for scripting because it's nearly universal.

### 3.2 Shebang and Making Scripts Executable

```bash
#!/usr/bin/env bash
echo "Hello, world"
```

```bash
chmod +x script.sh
./script.sh
```

`#!/usr/bin/env bash` is preferred over `#!/bin/bash` in portable scripts because it resolves `bash` via `PATH`, working even on systems where bash lives somewhere unusual (e.g. some BSDs, Nix).

### 3.3 Variables

```bash
name="Alice"
echo "Hello, $name"
echo "Hello, ${name}!"     # braces avoid ambiguity when concatenating text
readonly PI=3.14159         # constant
unset name                  # delete a variable
export MY_VAR="value"       # make available to child processes
```

**Always quote variable expansions** (`"$var"`, not `$var`) unless you specifically want word-splitting/globbing to happen. This single habit prevents the majority of real-world bash bugs.

### 3.4 Command Substitution and Arithmetic

```bash
now=$(date +%Y-%m-%d)          # preferred over legacy backticks `date`
count=$(ls | wc -l)
result=$(( 5 + 3 * 2 ))         # arithmetic expansion
((count++))                     # arithmetic command, useful in loops
```

### 3.5 Quoting Rules

| Quote | Behavior |
|---|---|
| `'single'` | Everything literal, no expansion at all |
| `"double"` | Variables and command substitution expand, but globbing/word-splitting suppressed |
| `` `backtick` `` | Legacy command substitution — avoid, use `$(...)` |
| unquoted | Word-splitting and globbing both happen — dangerous for filenames with spaces |

### 3.6 Control Structures

```bash
# if / elif / else
if [[ -f "$file" ]]; then
  echo "exists"
elif [[ -d "$file" ]]; then
  echo "is a directory"
else
  echo "missing"
fi

# for loop
for f in *.log; do
  echo "processing $f"
done

# C-style for loop
for ((i=0; i<10; i++)); do echo "$i"; done

# while loop
while read -r line; do
  echo "line: $line"
done < input.txt

# case statement
case "$1" in
  start) echo "starting" ;;
  stop)  echo "stopping" ;;
  *)     echo "usage: $0 {start|stop}" ;;
esac
```

Use `[[ ... ]]` (bash builtin test) instead of `[ ... ]` (POSIX `test`) whenever writing bash-specific scripts — it's safer with unquoted variables, supports `&&`/`||`/regex matching (`=~`), and avoids classic word-splitting pitfalls.

### 3.7 Functions

```bash
greet() {
  local name="$1"          # always use `local` inside functions
  echo "Hello, $name"
}
greet "Bob"
```

### 3.8 Exit Codes

Every command returns an exit code: `0` means success, non-zero means failure (the specific number often indicates the type of failure). Check with `$?`:

```bash
grep "pattern" file.txt
echo $?      # 0 if found, 1 if not found, 2 if error (e.g. file missing)
```


---

<a id="part-4-essential-commands"></a>
## Part 4: Essential Command Line Tools

This section is a fast-reference tour. Beginners should know these exist; everyone should know the flags marked **★** as daily-driver essentials.

### 4.1 File & Directory Operations

```bash
find . -name "*.log" -mtime +7 -delete        # ★ files older than 7 days, delete
find . -type f -size +100M                     # files bigger than 100MB
find . -perm -u+x                              # files executable by owner
find /var/log -name "*.log" -exec gzip {} \;   # run a command per match
locate filename.txt                             # fast search via prebuilt DB (updatedb)
du -sh */                                       # ★ size of each subdirectory
du -h --max-depth=1 | sort -rh                  # ★ sorted by size, human-readable
df -h                                            # ★ disk free space per filesystem
stat file.txt                                    # detailed metadata (inode, timestamps, perms)
ln -s /path/to/target linkname                   # ★ symbolic link
ln /path/to/target hardlink                      # hard link (same inode, same filesystem only)
rsync -avz --progress src/ user@host:/dst/       # ★ sync files, resumable, compressed
rsync -avz --delete src/ dst/                    # mirror (deletes extras in dst)
```

### 4.2 Viewing & Editing Text

```bash
nano file.txt          # beginner-friendly editor
vim file.txt            # ★ ubiquitous, worth learning: i (insert), Esc, :wq (save+quit), :q! (discard)
diff file1 file2        # ★ show differences
diff -u file1 file2      # unified diff format (used for patches)
patch < changes.diff      # apply a diff
wc -l file.txt            # ★ count lines
wc -c file.txt             # count bytes
sort file.txt               # ★ sort lines
sort -n -k2 file.txt          # numeric sort on column 2
uniq -c                        # ★ count consecutive duplicate lines (use after sort)
```

### 4.3 Compression & Archives

```bash
tar -czvf archive.tar.gz dir/     # ★ create gzip-compressed tarball
tar -xzvf archive.tar.gz          # ★ extract
tar -tzvf archive.tar.gz          # list contents without extracting
tar -cJvf archive.tar.xz dir/     # xz compression (smaller, slower)
zip -r archive.zip dir/
unzip archive.zip
```

### 4.4 Networking Basics

```bash
ping -c 4 google.com               # ★ connectivity test
curl -I https://example.com         # ★ headers only (HEAD request)
curl -o file.zip https://...         # download
curl -s -o /dev/null -w "%{http_code}\n" URL   # just the status code
wget https://example.com/file.tar.gz
ssh user@host                        # ★ remote shell
scp file.txt user@host:/path/         # copy over SSH
ip a                                   # ★ show interfaces & addresses (modern replacement for ifconfig)
ip route                                # show routing table
ss -tulnp                                # ★ show listening ports (modern replacement for netstat)
dig example.com                          # DNS lookup
nslookup example.com
traceroute example.com
```

### 4.5 System Info

```bash
uname -a           # kernel/version info
hostnamectl          # hostname, OS, kernel details (systemd systems)
uptime                # load average and uptime
free -h                # ★ memory usage
lscpu                   # CPU details
lsblk                    # ★ block devices/partitions tree
lsusb / lspci             # USB/PCI devices
whoami / id                # current user / uid, gid, groups
w                            # who's logged in and what they're doing
```

---

<a id="part-5-advanced-bash-scripting"></a>
## Part 5: Advanced Bash Scripting

This is where scripts stop being "a few commands in a file" and start being **software** — with error handling, argument parsing, logging, and predictable behavior under failure.

### 5.1 The Non-Negotiable Header

Every production bash script should start with:

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'
```

What each does:

- `set -e` — exit immediately if any command fails (non-zero exit code), instead of silently continuing.
- `set -u` — treat unset variables as an error instead of expanding to an empty string. Catches typos like `$FOOO` instantly.
- `set -o pipefail` — a pipeline's exit code is the last *non-zero* exit code in it, not just the final command's. Without this, `false | true` reports success.
- `IFS=$'\n\t'` — restricts word-splitting to newlines and tabs, not spaces, avoiding subtle bugs when looping over filenames with spaces.

Caveats: `set -e` doesn't trigger inside conditionals (`if cmd; then`), inside `&&`/`||` chains, or inside functions called as part of a condition. It also doesn't propagate into command substitutions the way people expect in older bash. Don't treat `set -e` as a substitute for real error handling — use it as a safety net, not the whole net.

### 5.2 Argument Parsing

Basic positional parsing:

```bash
echo "Script name: $0"
echo "First arg: $1"
echo "All args: $@"
echo "Number of args: $#"
shift          # drop $1, shift everything left
```

Robust flag parsing with `getopts` (short flags only):

```bash
while getopts ":f:v" opt; do
  case $opt in
    f) file="$OPTARG" ;;
    v) verbose=true ;;
    \?) echo "Invalid option: -$OPTARG" >&2; exit 1 ;;
    :) echo "Option -$OPTARG requires an argument" >&2; exit 1 ;;
  esac
done
shift $((OPTIND -1))
```

For long options (`--file=foo`, `--verbose`), hand-roll a loop:

```bash
while [[ $# -gt 0 ]]; do
  case "$1" in
    --file) file="$2"; shift 2 ;;
    --file=*) file="${1#*=}"; shift ;;
    --verbose) verbose=true; shift ;;
    --) shift; break ;;
    -*) echo "Unknown option: $1" >&2; exit 1 ;;
    *) break ;;
  esac
done
```

### 5.3 Arrays

```bash
files=(a.txt b.txt "c with spaces.txt")
echo "${files[0]}"            # first element
echo "${files[@]}"             # all elements, each as separate word (quote it!)
echo "${#files[@]}"              # length
files+=("d.txt")                  # append

for f in "${files[@]}"; do        # ★ always quote "${arr[@]}" in loops
  echo "$f"
done

# Associative arrays (bash 4+)
declare -A config
config[host]="localhost"
config[port]="8080"
for key in "${!config[@]}"; do
  echo "$key = ${config[$key]}"
done
```

### 5.4 Parameter Expansion — The Underused Powerhouse

```bash
"${var:-default}"     # use default if var is unset or empty (doesn't assign)
"${var:=default}"     # same, but also assigns default to var
"${var:?error msg}"   # exit with error message if var is unset/empty
"${var:+alt}"         # use alt value only if var IS set

"${var#pattern}"      # remove shortest match from front
"${var##pattern}"     # remove longest match from front
"${var%pattern}"      # remove shortest match from end
"${var%%pattern}"     # remove longest match from end

path="/home/user/file.tar.gz"
echo "${path##*/}"     # file.tar.gz   (basename)
echo "${path%/*}"      # /home/user    (dirname)
echo "${path%%.*}"     # /home/user/file  (strip all extensions)
echo "${path/user/admin}"  # substitute first match
echo "${path//\//_}"        # substitute all matches (replace all / with _)

str="hello"
echo "${str^^}"    # HELLO (uppercase)
echo "${str,,}"    # hello (lowercase)
echo "${#str}"      # 5 (length)
echo "${str:1:3}"    # ell (substring: offset 1, length 3)
```

Mastering this avoids dozens of unnecessary calls to `sed`/`awk`/`basename`/`dirname` for simple string manipulation, which measurably speeds up scripts with tight loops.

### 5.5 Error Handling & Traps

```bash
cleanup() {
  echo "Cleaning up temp files..."
  rm -f "$tmpfile"
}
trap cleanup EXIT               # ★ runs on any exit — normal, error, or signal
trap 'echo "Error on line $LINENO"' ERR
trap 'echo "Interrupted"; exit 130' INT TERM

tmpfile=$(mktemp)
```

`trap ... EXIT` is the bash equivalent of `finally` — it guarantees cleanup runs whether the script exits normally, hits `set -e`, or is killed with Ctrl-C (for signals, also trap `INT`/`TERM` explicitly since `EXIT` alone won't fire on a hard kill).

A more thorough error handler:

```bash
set -euo pipefail
trap 'err_handler $? $LINENO' ERR

err_handler() {
  local exit_code=$1 line_no=$2
  echo "[$(date '+%F %T')] ERROR: command failed with exit code $exit_code at line $line_no" >&2
  exit "$exit_code"
}
```

### 5.6 Subshells vs. Current Shell

```bash
(cd /tmp && rm -f *.log)   # runs in a subshell — cwd change doesn't affect parent shell
cd /tmp && rm -f *.log      # runs in current shell — cwd change persists
```

Subshells are also implicitly created by pipelines: `cat file | while read -r line; do count=$((count+1)); done` — the `count` variable modified inside the `while` loop **does not persist** after the pipe, because the loop runs in a subshell. Fix with process substitution:

```bash
count=0
while read -r line; do
  count=$((count+1))
done < <(cat file)      # process substitution — while loop stays in current shell
echo "$count"             # now correctly reflects the total
```

### 5.7 Process Substitution & Here-Documents

```bash
diff <(sort file1) <(sort file2)     # compare command outputs without temp files

cat << EOF > config.txt
host=localhost
port=8080
EOF

cat << 'EOF'                          # single-quoted heredoc: no variable expansion
literal $HOME stays literal
EOF

mysql -u root -p"$PASS" << SQL
SELECT * FROM users;
SQL
```

### 5.8 Debugging Scripts

```bash
bash -x script.sh              # ★ trace every command as it executes
bash -n script.sh               # syntax check only, don't run
set -x                            # enable tracing mid-script
set +x                             # disable tracing
PS4='+ ${BASH_SOURCE}:${LINENO}: '  # customize xtrace prefix to show file/line
```

Also invaluable: [`shellcheck`](https://www.shellcheck.net/) — a static analyzer that catches quoting bugs, unsafe expansions, and portability issues before you ever run the script.

```bash
shellcheck script.sh
```

### 5.9 Writing Idempotent, Safe Scripts

Idempotency means running a script twice produces the same result as running it once — critical for anything that touches infrastructure.

```bash
# Bad: fails on second run
mkdir /opt/myapp

# Good: idempotent
mkdir -p /opt/myapp

# Bad: duplicate cron entries on each run
echo "0 2 * * * /opt/backup.sh" >> /etc/crontab

# Good: check before adding
grep -qF "/opt/backup.sh" /etc/crontab || echo "0 2 * * * /opt/backup.sh" >> /etc/crontab

# Good: lock file to prevent concurrent runs
exec 200>/var/lock/myscript.lock
flock -n 200 || { echo "Already running"; exit 1; }
```

### 5.10 Logging Pattern for Production Scripts

```bash
LOG_FILE="/var/log/myscript.log"
log() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}
log "Starting backup job"
```

### 5.11 A Complete Reference Script

```bash
#!/usr/bin/env bash
#
# backup.sh - Rotate and archive logs, ship to remote storage.
set -euo pipefail
IFS=$'\n\t'

readonly SCRIPT_NAME=$(basename "$0")
readonly LOG_DIR="/var/log/myapp"
readonly DEST_HOST="backup.internal"
readonly LOCK_FILE="/var/lock/${SCRIPT_NAME}.lock"

usage() {
  cat << EOF
Usage: $SCRIPT_NAME [-d DIR] [-h HOST]
  -d DIR   Directory to back up (default: $LOG_DIR)
  -h HOST  Destination host    (default: $DEST_HOST)
EOF
  exit 1
}

log()  { echo "[$(date '+%F %T')] $*"; }
die()  { log "ERROR: $*" >&2; exit 1; }

cleanup() { rm -f "${tmpfile:-}"; }
trap cleanup EXIT

main() {
  exec 200>"$LOCK_FILE"
  flock -n 200 || die "Another instance is already running"

  local dir="$LOG_DIR" host="$DEST_HOST"
  while getopts ":d:h:" opt; do
    case $opt in
      d) dir="$OPTARG" ;;
      h) host="$OPTARG" ;;
      *) usage ;;
    esac
  done

  [[ -d "$dir" ]] || die "Directory not found: $dir"

  tmpfile=$(mktemp /tmp/backup-XXXXXX.tar.gz)
  log "Archiving $dir -> $tmpfile"
  tar -czf "$tmpfile" -C "$dir" .

  log "Shipping to $host"
  rsync -avz "$tmpfile" "${host}:/backups/" || die "rsync failed"

  log "Backup complete"
}

main "$@"
```

---

<a id="part-6-text-processing"></a>
## Part 6: Text Processing Mastery

Linux administration is, in large part, munging text: logs, config files, command output. `grep`, `sed`, and `awk` are the three tools worth mastering deeply.

### 6.1 grep

```bash
grep "error" file.log                    # basic search
grep -i "error" file.log                   # case-insensitive
grep -r "TODO" src/                          # ★ recursive
grep -n "error" file.log                      # show line numbers
grep -v "debug" file.log                       # ★ invert match (exclude lines)
grep -c "error" file.log                        # count matches
grep -E "error|warning" file.log                 # ★ extended regex (OR)
grep -A 3 -B 1 "Exception" file.log                # 3 lines after, 1 before match
grep -l "pattern" *.txt                              # just filenames with a match
grep -o -E "[0-9]{1,3}(\.[0-9]{1,3}){3}" file.log      # extract IPs (only matched text)
grep -P "(?<=user=)\w+" file.log                        # Perl regex (lookbehind)
```

### 6.2 sed — Stream Editor

```bash
sed 's/foo/bar/' file.txt              # replace first occurrence per line
sed 's/foo/bar/g' file.txt               # ★ replace all occurrences
sed -i 's/foo/bar/g' file.txt              # ★ edit file in place
sed -i.bak 's/foo/bar/g' file.txt            # in place, keep .bak backup
sed -n '5,10p' file.txt                        # print only lines 5-10
sed '/^#/d' file.txt                             # ★ delete comment lines
sed '/^$/d' file.txt                               # delete blank lines
sed -n '/START/,/END/p' file.txt                     # print block between markers
sed 's/^/  /' file.txt                                 # indent every line
sed -E 's/([0-9]+)-([0-9]+)/\2-\1/' file.txt              # capture groups, swap
```

### 6.3 awk — Pattern-Action Text Processing

`awk` treats input as records (default: lines) split into fields (default: whitespace-separated), referenced as `$1`, `$2`, ... and `$0` for the whole line.

```bash
awk '{print $1}' file.txt                    # first column
awk -F',' '{print $2}' file.csv                # ★ custom field separator (CSV)
awk '{print $NF}' file.txt                       # last field
awk '{sum += $3} END {print sum}' file.txt          # ★ sum a column
awk '$3 > 100 {print $1, $3}' file.txt                # filter rows by condition
awk 'NR==5' file.txt                                    # print only line 5
awk 'NR % 2 == 0' file.txt                                # every even line
awk '{print length($0)}' file.txt                           # line lengths
awk 'BEGIN{FS=":"} {print $1}' /etc/passwd                    # list usernames
awk '{count[$1]++} END {for (k in count) print k, count[k]}' access.log  # ★ group & count
```

Real-world example — top 10 IPs hitting a web server from an access log:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
```

Real-world example — sum bytes transferred per status code from an Apache/Nginx combined log:

```bash
awk '{bytes[$9] += $10} END {for (s in bytes) print s, bytes[s]}' access.log
```

### 6.4 Regular Expressions Cheat Sheet

| Pattern | Meaning |
|---|---|
| `.` | any character |
| `*` | zero or more of previous |
| `+` | one or more (ERE/PCRE) |
| `?` | zero or one (ERE/PCRE) |
| `^` `$` | start / end of line |
| `[abc]` | character class |
| `[^abc]` | negated class |
| `[0-9]` | digit range |
| `\d` `\w` `\s` | digit / word / whitespace (PCRE only, `grep -P`) |
| `(...)` | capture group |
| `\|` | alternation (needs `-E` or escaping in BRE) |
| `{n,m}` | repetition count |

### 6.5 Combining Tools — Real Pipelines

```bash
# Find the 5 largest files under /var, human readable
find /var -type f -exec du -h {} + | sort -rh | head -5

# Count unique visitors per hour from a log
awk '{print $4}' access.log | cut -d: -f2 | sort | uniq -c

# Find processes using more than 500MB RSS, sorted descending
ps aux | awk '$6 > 512000 {print $2, $4, $11}' | sort -k2 -rn

# Tail a log and highlight errors in real time
tail -f app.log | grep --line-buffered -i --color=always "error"

# Extract and dedupe all email addresses from a file
grep -Eo '[[:alnum:]._%+-]+@[[:alnum:].-]+\.[[:alpha:]]{2,}' file.txt | sort -u
```

---

<a id="part-7-process-management"></a>
## Part 7: Process, Job & Resource Management

### 7.1 Viewing Processes

```bash
ps aux                       # ★ all processes, BSD style
ps aux --sort=-%mem | head    # top memory consumers
ps -ef --forest                 # ★ tree view showing parent/child relationships
top                                # ★ live view (press 'q' to quit, 'M' sort by mem, 'P' by cpu)
htop                                 # nicer interactive version (install separately)
pgrep -f "nginx"                       # find PIDs matching a pattern
pstree -p                                # process tree with PIDs
```

### 7.2 Job Control

```bash
long_command &         # ★ run in background
jobs                      # list background jobs
fg %1                       # bring job 1 to foreground
bg %1                        # resume job 1 in background
disown %1                     # detach job from shell (survives shell exit)
nohup long_command &            # ★ survives terminal/SSH disconnect, output to nohup.out
Ctrl+Z                            # suspend foreground job
```

For long-running remote sessions, `screen` or `tmux` are essential — they let a process keep running after you disconnect, and let you reattach later:

```bash
tmux new -s work           # ★ start a named session
tmux attach -t work           # reattach after disconnect
tmux ls                         # list sessions
# inside tmux: Ctrl+b then d to detach, Ctrl+b then c for new window
```

### 7.3 Killing Processes

```bash
kill PID                # sends SIGTERM (15) — graceful, catchable
kill -9 PID                # ★ SIGKILL — immediate, uncatchable (last resort)
kill -HUP PID                 # SIGHUP — often used to reload config without restart
killall nginx                    # kill by process name
pkill -f "python.*worker"          # kill by pattern match
```

Prefer `SIGTERM` first, always. `SIGKILL` doesn't let the process clean up file handles, flush buffers, or release locks — this is a common cause of corrupted state in databases and half-written files.

### 7.4 Priority & Resource Limits

```bash
nice -n 10 long_command          # start with lower priority (higher niceness = less priority)
renice -n 5 -p PID                  # change priority of a running process
ionice -c3 -p PID                     # ★ set idle I/O priority, good for backup jobs
ulimit -a                               # show all limits for current shell
ulimit -n 65536                           # raise open-file-descriptor limit for this session
```

Persistent limits go in `/etc/security/limits.conf`:

```
appuser  soft  nofile  65536
appuser  hard  nofile  100000
```

### 7.5 Understanding Load Average

```bash
uptime
# load average: 0.52, 0.58, 0.61
```

These three numbers are the average number of processes waiting for CPU (or in uninterruptible I/O wait) over the last 1, 5, and 15 minutes. A load average above your core count sustained over 15 minutes indicates the system is consistently oversubscribed — investigate with `top`/`htop` and `iostat`.

### 7.6 /proc — The Process Filesystem

```bash
cat /proc/PID/status         # detailed process info (memory, state, threads)
cat /proc/PID/cmdline          # full command line that launched it
ls -la /proc/PID/fd               # ★ open file descriptors — great for "who has this file open"
cat /proc/loadavg                   # raw load average values
cat /proc/meminfo                     # detailed memory breakdown
cat /proc/cpuinfo                       # CPU details
lsof -p PID                                # ★ friendlier view of open files for a process
lsof -i :443                                 # ★ what's listening/connected on port 443
fuser -k 8080/tcp                              # kill whatever is using port 8080
```

---

<a id="part-8-users-permissions"></a>
## Part 8: Users, Permissions & Access Control

### 8.1 User & Group Basics

```bash
useradd -m -s /bin/bash alice        # ★ create user with home dir and shell
passwd alice                            # set password
usermod -aG sudo alice                    # ★ add to group (-a = append, don't overwrite)
userdel -r alice                            # delete user and home dir
groupadd developers                            # create group
id alice                                          # show uid/gid/groups
su - alice                                          # switch user, full login shell
sudo -u alice command                                 # run one command as alice
```

Key files:
- `/etc/passwd` — user accounts (username, uid, gid, home, shell)
- `/etc/shadow` — hashed passwords (root-readable only)
- `/etc/group` — group definitions
- `/etc/sudoers` — sudo rules (edit only with `visudo`, which validates syntax before saving)

### 8.2 Special Permission Bits

```bash
chmod u+s /usr/bin/passwd    # setuid — runs as file owner (root), not invoking user
chmod g+s /shared/dir           # setgid on dir — new files inherit the dir's group
chmod +t /tmp                     # sticky bit — users can only delete their own files in shared dir
```

Numeric form: setuid=4000, setgid=2000, sticky=1000, prepended to the normal 3 digits (e.g. `chmod 4755 file`).

**Security note:** setuid binaries are a classic privilege-escalation vector. Audit them periodically:

```bash
find / -perm -4000 -type f 2>/dev/null    # ★ find all setuid binaries on the system
find / -perm -2000 -type f 2>/dev/null      # setgid binaries
find / -perm -4000 -o -perm -2000 2>/dev/null | xargs ls -la   # combined, detailed
```

### 8.3 ACLs (Access Control Lists) — Beyond owner/group/other

Standard Unix permissions only support one owner and one group. ACLs let you grant fine-grained access to multiple users/groups on the same file:

```bash
getfacl file.txt                          # view ACLs
setfacl -m u:bob:rw file.txt                 # ★ give bob read/write, without changing owner
setfacl -m g:auditors:r file.txt               # give a group read access
setfacl -x u:bob file.txt                        # remove bob's entry
setfacl -R -m u:bob:rx /srv/shared/                 # recursive
setfacl -d -m u:bob:rw /srv/shared/                   # default ACL — inherited by new files
```

### 8.4 sudo Configuration

Edit with `visudo` only — it catches syntax errors before they lock you out:

```
# /etc/sudoers.d/deploy
deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart myapp, /usr/bin/systemctl status myapp
```

Best practices:
- Put custom rules in `/etc/sudoers.d/` as separate files, not the main `/etc/sudoers` — easier to manage and version.
- Scope `NOPASSWD` to specific commands, never `NOPASSWD: ALL` for automated/service accounts.
- Use `%groupname` to grant to a whole group instead of listing users individually.
- Log everything: sudo already logs to `/var/log/auth.log` (Debian) or `/var/log/secure` (RHEL); consider `Defaults logfile=/var/log/sudo.log` for a dedicated log.

### 8.5 PAM (Pluggable Authentication Modules)

PAM controls *how* authentication happens system-wide — password complexity, lockout policy, 2FA, login restrictions — via config files in `/etc/pam.d/`.

Example: enforce account lockout after failed login attempts (`/etc/pam.d/common-auth` on Debian, or the relevant service file on RHEL):

```
auth required pam_tally2.so deny=5 unlock_time=900 onerr=fail
```

Example: enforce password complexity via `pam_pwquality` (`/etc/security/pwquality.conf`):

```
minlen = 14
dcredit = -1
ucredit = -1
ocredit = -1
lcredit = -1
retry = 3
```

PAM misconfiguration can lock out **all** logins, including root — always test in a second session, and keep a root console/rescue path available before changing auth modules on a remote box.

---

<a id="part-9-systemd-services-logging"></a>
## Part 9: systemd, Services & Logging

### 9.1 Managing Services

```bash
systemctl status nginx           # ★ check state, recent logs, PID
systemctl start nginx               # start now
systemctl stop nginx                  # stop now
systemctl restart nginx                 # ★ stop then start
systemctl reload nginx                    # re-read config without dropping connections (if supported)
systemctl enable nginx                       # ★ start automatically at boot
systemctl disable nginx                        # don't start at boot
systemctl enable --now nginx                     # ★ enable AND start in one command
systemctl is-active nginx                          # just the state (active/inactive/failed)
systemctl list-units --type=service --state=running   # ★ all running services
systemctl list-unit-files --state=enabled              # what's set to start at boot
```

### 9.2 Writing a systemd Service Unit

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target postgresql.service
Wants=postgresql.service

[Service]
Type=simple
User=appuser
Group=appuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/venv/bin/python /opt/myapp/main.py
Restart=on-failure
RestartSec=5
Environment="ENV=production"
EnvironmentFile=-/etc/myapp/env
StandardOutput=journal
StandardError=journal

# Hardening (see Part 14 for the full set)
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true
ReadWritePaths=/opt/myapp/data

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload            # ★ required after editing any unit file
systemctl enable --now myapp
journalctl -u myapp -f               # ★ follow this service's logs live
```

`Type=` matters: `simple` (default, process doesn't fork), `forking` (traditional daemon that forks and exits parent), `oneshot` (runs once and exits, useful for setup tasks), `notify` (process signals systemd via sd_notify when ready).

### 9.3 systemd Timers (cron replacement)

```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Run backup daily

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true
RandomizedDelaySec=300

[Install]
WantedBy=timers.target
```

```bash
systemctl enable --now backup.timer
systemctl list-timers                # ★ see all scheduled timers and next run time
```

Timers are generally preferable to cron for anything tied to a systemd service: they get proper logging via journald, `Persistent=true` catches up on missed runs after downtime, and dependencies (`After=`) are explicit.

### 9.4 journalctl — The Structured Log Viewer

```bash
journalctl -u nginx                    # logs for one unit
journalctl -u nginx -f                    # ★ follow live
journalctl -u nginx --since "1 hour ago"    # ★ time-bounded
journalctl -u nginx --since today --until "2024-01-01 12:00"
journalctl -p err                              # ★ only error priority and above
journalctl -k                                    # kernel messages (like dmesg)
journalctl -b                                      # logs since last boot
journalctl -b -1                                     # logs from previous boot
journalctl --disk-usage                                # how much space the journal is using
journalctl --vacuum-time=7d                              # ★ prune logs older than 7 days
journalctl --vacuum-size=500M                              # cap total journal size
```

Making the journal persistent across reboots (often not default on minimal installs):

```bash
mkdir -p /var/log/journal
systemd-tmpfiles --create --prefix /var/log/journal
systemctl restart systemd-journald
```

Set retention limits in `/etc/systemd/journald.conf`:

```ini
[Journal]
Storage=persistent
SystemMaxUse=1G
MaxRetentionSec=30day
```

### 9.5 Log Rotation with logrotate

For applications that write plain log files (not journald), `logrotate` prevents unbounded disk growth:

```
# /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 appuser appuser
    sharedscripts
    postrotate
        systemctl reload myapp >/dev/null 2>&1 || true
    endscript
}
```

```bash
logrotate -d /etc/logrotate.d/myapp     # dry run, shows what would happen
logrotate -f /etc/logrotate.d/myapp       # force rotation now (for testing)
```

---

<a id="part-10-package-management"></a>
## Part 10: Package Management

### 10.1 Debian/Ubuntu (APT)

```bash
apt update                        # ★ refresh package index
apt upgrade                          # ★ upgrade installed packages
apt full-upgrade                        # upgrade, allowing package removal if needed
apt install nginx                          # ★ install
apt remove nginx                              # remove, keep config files
apt purge nginx                                 # remove including config files
apt autoremove                                    # ★ clean up orphaned dependencies
apt search "web server"                             # search package names/descriptions
apt show nginx                                        # package details
apt list --installed                                    # everything installed
apt-cache policy nginx                                     # available versions & sources
dpkg -L nginx                                                 # list files installed by a package
dpkg -S /usr/sbin/nginx                                          # which package owns this file
```

Hold a package at its current version (prevent accidental upgrades):

```bash
apt-mark hold nginx
apt-mark unhold nginx
```

### 10.2 RHEL/Fedora/Rocky (DNF/YUM)

```bash
dnf update                     # ★ update everything (yum on older systems)
dnf install nginx                 # ★ install
dnf remove nginx                     # remove
dnf search web server                  # search
dnf info nginx                            # details
dnf list installed                          # everything installed
dnf provides */nginx                          # which package provides this file
rpm -ql nginx                                    # list files in package
rpm -qf /usr/sbin/nginx                             # which package owns this file
dnf history                                            # ★ transaction history (for rollback)
dnf history undo LAST                                     # undo last transaction
dnf versionlock add nginx                                    # pin a package version
```

### 10.3 Arch (Pacman)

```bash
pacman -Syu           # ★ sync repos and upgrade everything
pacman -S nginx          # ★ install
pacman -R nginx             # remove
pacman -Rns nginx              # remove with dependencies and config
pacman -Ss nginx                  # search
pacman -Qi nginx                     # package info
pacman -Qo /usr/bin/nginx               # which package owns this file
```

### 10.4 Building From Source (when packages aren't enough)

```bash
./configure --prefix=/usr/local
make -j"$(nproc)"          # ★ parallel build using all cores
sudo make install
```

Prefer packages over source builds wherever possible — package managers handle dependency resolution, security updates, and clean uninstalls. Reserve source builds for software unavailable in repos or requiring custom compile flags.

### 10.5 Snap, Flatpak & AppImage (brief note)

These are containerized/universal package formats increasingly common on desktop and some server contexts. `snap` (Ubuntu-backed) and `flatpak` (community-driven) both sandbox applications and manage their own runtime dependencies. Prefer native distro packages on servers where possible — snaps/flatpaks add overhead and their own update cadence, complicating hardening and auditing.

---

<a id="part-11-storage-filesystems"></a>
## Part 11: Storage, Disks & Filesystems

### 11.1 Viewing Disks and Partitions

```bash
lsblk -f                      # ★ block devices, filesystem types, mount points
fdisk -l                         # list partition tables (MBR)
parted -l                           # list partition tables (works with GPT too)
blkid                                  # ★ UUIDs and filesystem types per device
df -hT                                    # ★ mounted filesystems, type, usage
mount | column -t                            # nicely formatted mount table
```

### 11.2 Partitioning & Filesystems

```bash
fdisk /dev/sdb              # interactive MBR partitioning (n=new, w=write)
parted /dev/sdb --script mklabel gpt mkpart primary ext4 0% 100%   # scripted GPT partition

mkfs.ext4 /dev/sdb1              # ★ format as ext4
mkfs.xfs /dev/sdb1                  # format as XFS (default on RHEL, good for large files)
mkswap /dev/sdb2                       # create swap partition
swapon /dev/sdb2                          # activate it

mount /dev/sdb1 /mnt/data                   # ★ mount manually
umount /mnt/data                               # unmount

tune2fs -l /dev/sdb1                              # view ext4 filesystem details
e2fsck -f /dev/sdb1                                  # ★ check/repair ext filesystem (unmount first!)
xfs_repair /dev/sdb1                                    # check/repair XFS
```

### 11.3 /etc/fstab — Persistent Mounts

```
UUID=1234-5678  /mnt/data  ext4  defaults,noatime  0  2
```

Always mount by UUID (`blkid` to find it), not device name (`/dev/sdb1`) — device names can shift on reboot, especially with multiple disks. Test before rebooting:

```bash
mount -a          # ★ mount everything in fstab, will error immediately if syntax is wrong
```

A broken `/etc/fstab` entry with a hard failure can drop a server into an emergency shell on next boot — always test with `mount -a` and consider `nofail` for non-critical mounts:

```
UUID=1234-5678  /mnt/backup  ext4  defaults,noatime,nofail  0  2
```

### 11.4 LVM (Logical Volume Management)

LVM adds a flexible layer between raw disks and filesystems, allowing you to resize, snapshot, and span volumes across physical disks without unmounting.

```bash
pvcreate /dev/sdb /dev/sdc          # mark disks as physical volumes
vgcreate vg_data /dev/sdb /dev/sdc     # ★ create a volume group spanning both disks
lvcreate -L 50G -n lv_app vg_data         # ★ create a 50GB logical volume
mkfs.ext4 /dev/vg_data/lv_app                # format it

# Grow it live, without downtime:
lvextend -L +20G /dev/vg_data/lv_app          # ★ add 20GB to the logical volume
resize2fs /dev/vg_data/lv_app                    # grow ext4 to fill the new space
# (use `xfs_growfs /mount/point` instead of resize2fs for XFS)

lvs; vgs; pvs                                        # ★ list volumes at each layer
lvcreate -L 10G -s -n lv_app_snap /dev/vg_data/lv_app   # snapshot before a risky upgrade
```

### 11.5 RAID (mdadm)

```bash
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc   # RAID 1 (mirror)
mdadm --detail /dev/md0                    # ★ status and health
cat /proc/mdstat                              # ★ quick status, shows rebuild progress
mdadm --detail --scan >> /etc/mdadm/mdadm.conf   # persist config across reboots
mdadm --manage /dev/md0 --fail /dev/sdc            # mark a disk failed (for replacement)
mdadm --manage /dev/md0 --remove /dev/sdc             # remove failed disk
mdadm --manage /dev/md0 --add /dev/sdd                   # add replacement disk, triggers rebuild
```

RAID levels quick reference: **RAID 0** (striping, no redundancy, speed only — never use for anything you can't lose), **RAID 1** (mirroring, 50% capacity, survives 1 disk failure), **RAID 5** (striping + parity, survives 1 disk failure, needs 3+ disks), **RAID 10** (mirrored stripes, best performance+redundancy balance, needs 4+ disks). RAID is not a backup — it protects against disk failure, not accidental deletion, ransomware, or filesystem corruption.

### 11.6 Disk Health & Performance

```bash
smartctl -a /dev/sda            # ★ SMART health data — check before a disk fails silently
smartctl -t short /dev/sda         # run a short self-test
iostat -x 2                           # ★ per-disk I/O stats every 2 seconds (util%, await)
iotop                                    # ★ live per-process disk I/O (like top, for disk)
badblocks -v /dev/sdb                       # scan for bad sectors (read-only, safe)
hdparm -Tt /dev/sda                            # quick read-speed benchmark
```

### 11.7 Finding What's Eating Disk Space

```bash
du -h --max-depth=1 / 2>/dev/null | sort -rh | head -20     # ★ biggest top-level dirs
find / -xdev -type f -size +500M 2>/dev/null                    # files over 500MB
ncdu /                                                              # ★ interactive disk usage explorer (install separately)
```

---

<a id="part-12-networking"></a>
## Part 12: Networking Deep Dive

### 12.1 Modern Interface & Route Management (iproute2)

The `ip` command family has replaced the older `ifconfig`/`route`/`netstat` tools on virtually every modern distro.

```bash
ip a                              # ★ show all interfaces and addresses
ip link set eth0 up                  # bring interface up
ip addr add 192.168.1.10/24 dev eth0    # assign IP (temporary, not persistent)
ip route                                  # ★ show routing table
ip route add default via 192.168.1.1        # set default gateway
ip route get 8.8.8.8                          # ★ show which route/interface a destination would use
ip neigh                                         # ARP table
ip -s link show eth0                                # interface stats (errors, drops, packets)
```

Persistent network config lives in distro-specific places: `/etc/netplan/*.yaml` (Ubuntu, using `netplan apply`), `/etc/sysconfig/network-scripts/` (older RHEL), or `nmcli`/`nmtui` for NetworkManager-based systems.

### 12.2 Sockets & Connections (ss)

```bash
ss -tulnp             # ★ all listening TCP/UDP sockets with process names
ss -tan                  # all TCP connections, numeric
ss -tan state established     # only established connections
ss -tan state time-wait          # connections stuck in TIME_WAIT (can indicate churn issues)
ss -o state established '( dport = :443 or sport = :443 )'   # filter by port
```

### 12.3 DNS Tools

```bash
dig example.com                    # ★ full DNS query
dig +short example.com                # just the answer
dig example.com MX                       # mail records
dig -x 8.8.8.8                              # reverse lookup
dig @1.1.1.1 example.com                       # query a specific resolver
host example.com
resolvectl status                                  # systemd-resolved status (modern Ubuntu)
cat /etc/resolv.conf                                  # active resolvers (may be managed dynamically)
```

### 12.4 Packet Capture & Inspection

```bash
tcpdump -i eth0                             # ★ capture all traffic on an interface
tcpdump -i eth0 port 443                       # filter by port
tcpdump -i eth0 host 192.168.1.5                  # filter by host
tcpdump -i eth0 -w capture.pcap                      # ★ write to file for later analysis (Wireshark)
tcpdump -r capture.pcap -n                              # read back a capture, no DNS resolution
tcpdump -A -i eth0 port 80                                 # show ASCII payload (plaintext HTTP)
nc -zv host 22                                                # ★ quick TCP port check ("is it open")
nc -l -p 8080                                                    # listen on a port (testing/debugging)
```

### 12.5 Firewalls: iptables, nftables, ufw, firewalld

Linux firewalling has evolved: `iptables` (legacy but still everywhere) → `nftables` (modern kernel-level replacement) → distro-friendly frontends `ufw` (Debian/Ubuntu) and `firewalld` (RHEL/Fedora).

**ufw (simplest, good default for single servers):**

```bash
ufw default deny incoming              # ★ deny everything by default
ufw default allow outgoing
ufw allow 22/tcp                          # ★ allow SSH
ufw allow 443/tcp                            # allow HTTPS
ufw allow from 10.0.0.0/8 to any port 5432      # ★ restrict DB port to internal network only
ufw limit 22/tcp                                   # ★ rate-limit SSH to slow brute-force attempts
ufw enable
ufw status verbose
ufw delete allow 443/tcp
```

**firewalld (RHEL/Fedora, zone-based):**

```bash
firewall-cmd --get-active-zones
firewall-cmd --zone=public --add-service=https --permanent    # ★ permanent = persists across reboot
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --reload                                              # ★ required to apply --permanent changes
firewall-cmd --list-all
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="10.0.0.0/8" port port="5432" protocol="tcp" accept' --permanent
```

**nftables (modern raw syntax, direct kernel filtering):**

```bash
nft list ruleset                    # ★ show current rules
nft add table inet filter
nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }
nft add rule inet filter input ct state established,related accept
nft add rule inet filter input iif lo accept
nft add rule inet filter input tcp dport 22 accept
nft add rule inet filter input tcp dport 443 accept
```

We revisit firewalling with a security-first mindset — default-deny posture, logging dropped packets, rate-limiting — in Part 14.

### 12.6 Bandwidth & Connection Monitoring

```bash
nload                    # ★ live bandwidth graph per interface
iftop                       # ★ live per-connection bandwidth usage
vnstat -l                      # live traffic totals
vnstat -d                        # daily traffic history (needs to run as a service to accumulate)
ss -s                               # socket statistics summary (counts by state)
```

### 12.7 SSH Deep Dive (client side)

```bash
ssh -i ~/.ssh/id_ed25519 user@host          # ★ specify a key explicitly
ssh -p 2222 user@host                          # non-default port
ssh -L 8080:localhost:80 user@host                # ★ local port forward (tunnel remote service to your machine)
ssh -R 9000:localhost:3000 user@host                 # remote port forward (expose your local service to remote)
ssh -D 1080 user@host                                   # SOCKS proxy through the SSH connection
ssh -J bastion user@internal-host                          # ★ jump host / bastion hop
scp -r dir/ user@host:/remote/path/
rsync -avz -e "ssh -p 2222" src/ user@host:/dst/
```

`~/.ssh/config` for readable, reusable connection shortcuts:

```
Host prod
    HostName 203.0.113.10
    User deploy
    Port 2222
    IdentityFile ~/.ssh/prod_ed25519
    ForwardAgent no

Host internal-*
    ProxyJump bastion
    User admin
```

Then simply: `ssh prod`. We cover **server-side** SSH hardening (disabling password auth, key types, `Match` blocks, fail2ban integration) in Part 14 — that's where the real security value is.

---

<a id="part-13-server-management"></a>
## Part 13: Server Management

### 13.1 Web Server Basics: Nginx

```nginx
# /etc/nginx/sites-available/example.com
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;     # ★ force HTTPS
}

server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    root /var/www/example.com;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8000/;         # ★ reverse proxy to app server
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    access_log /var/log/nginx/example.access.log;
    error_log  /var/log/nginx/example.error.log;
}
```

```bash
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/    # Debian/Ubuntu pattern
nginx -t                     # ★ test config syntax before reloading — always do this
systemctl reload nginx          # ★ apply changes without dropping connections
```

### 13.2 Reverse Proxy & Load Balancing Concepts

A reverse proxy sits in front of application servers, terminating TLS, handling static files, and forwarding dynamic requests. Load balancing distributes requests across multiple backend instances:

```nginx
upstream app_backend {
    least_conn;                          # ★ balancing method: least_conn, ip_hash, or default round-robin
    server 10.0.0.11:8000 max_fails=3 fail_timeout=30s;
    server 10.0.0.12:8000 max_fails=3 fail_timeout=30s;
    server 10.0.0.13:8000 backup;        # only used if others are down
}

server {
    location / {
        proxy_pass http://app_backend;
    }
}
```

`ip_hash` pins a client to the same backend (useful for sticky sessions without shared session storage); `least_conn` sends new requests to whichever backend has the fewest active connections — usually better than round-robin under uneven request costs.

### 13.3 TLS Certificates with Let's Encrypt

```bash
apt install certbot python3-certbot-nginx
certbot --nginx -d example.com -d www.example.com     # ★ obtains cert AND edits nginx config
certbot renew --dry-run                                    # ★ test renewal before trusting the cron job
certbot certificates                                          # list installed certs and expiry
```

Certbot installs a systemd timer or cron job automatically for renewal — verify it exists:

```bash
systemctl list-timers | grep certbot
```

### 13.4 Database Server Basics (PostgreSQL example)

```bash
sudo -u postgres psql                          # ★ connect as the postgres superuser
sudo -u postgres createuser --interactive         # create a new DB role
sudo -u postgres createdb myapp
```

`postgresql.conf` key production tunables:

```
listen_addresses = 'localhost'      # ★ don't expose to the network unless required
max_connections = 100
shared_buffers = 256MB               # ~25% of RAM on a dedicated DB server
```

`pg_hba.conf` controls *who* can connect and *how* they authenticate — this is the actual access-control layer:

```
# TYPE  DATABASE  USER  ADDRESS        METHOD
host    myapp     appuser  10.0.0.0/24   scram-sha-256    # ★ restrict by network + strong auth
local   all       postgres              peer
```

Never bind a database directly to a public interface (`listen_addresses = '*'`) without a firewall rule restricting the port to trusted networks (see Part 12.5 and Part 14).

### 13.5 Load Testing & Capacity Planning

```bash
ab -n 1000 -c 50 https://example.com/          # Apache Bench: 1000 requests, 50 concurrent
wrk -t4 -c100 -d30s https://example.com/          # ★ wrk: more modern, higher throughput testing
```

Watch `top`/`iostat`/`ss -s` on the server *while* the test runs — capacity planning is about finding the bottleneck (CPU-bound? I/O-bound? connection-limited?), not just getting a number.

### 13.6 Deployment Patterns

Zero-downtime deploys generally follow one of:

- **Blue-Green**: run two identical environments, switch traffic (via load balancer/DNS) once the new one is verified healthy.
- **Rolling**: update backend instances one at a time behind a load balancer, health-checking before moving to the next.
- **Canary**: route a small percentage of traffic to the new version first, monitor error rates, then ramp up.

A minimal rolling-deploy script for a systemd-managed app behind Nginx upstream:

```bash
#!/usr/bin/env bash
set -euo pipefail
for host in app1 app2 app3; do
  echo "Deploying to $host..."
  ssh "$host" "cd /opt/myapp && git pull && systemctl restart myapp"
  # wait for health check before moving to next node
  until curl -sf "http://$host:8000/health" > /dev/null; do
    echo "Waiting for $host to become healthy..."
    sleep 2
  done
  echo "$host is healthy."
done
```

---

<a id="part-14-security-hardening"></a>
## Part 14: Security Hardening

This is the section most guides skim. It shouldn't be skimmed — most breaches come from a handful of well-known, entirely preventable misconfigurations.

### 14.1 SSH Hardening

Edit `/etc/ssh/sshd_config`, then `sshd -t` to validate syntax, then `systemctl reload sshd`. **Keep an existing session open while testing** — a syntax error or overly-strict rule can lock you out.

```
# /etc/ssh/sshd_config

Port 2222                                  # obscurity isn't security, but it cuts automated scan noise dramatically
Protocol 2
PermitRootLogin no                          # ★ never allow direct root SSH login
PasswordAuthentication no                      # ★ keys only — the single biggest hardening win
PubkeyAuthentication yes
AuthenticationMethods publickey                   # explicit, don't rely on defaults
PermitEmptyPasswords no
MaxAuthTries 3                                       # ★ limit brute-force attempts per connection
LoginGraceTime 20
ClientAliveInterval 300
ClientAliveCountMax 2
X11Forwarding no
AllowTcpForwarding no                                    # disable unless you specifically need tunneling
AllowUsers deploy admin                                     # ★ explicit allowlist of who may connect
Banner /etc/issue.net                                          # legal warning banner (compliance requirement in many orgs)

# Restrict algorithms to modern, strong ones only
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com

# Per-group restriction example
Match Group sftpusers
    ChrootDirectory /srv/sftp/%u
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```

Generate a modern key on the client side and copy it over:

```bash
ssh-keygen -t ed25519 -a 100 -C "you@host"    # ★ Ed25519: modern, fast, small, strong
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@host
```

Avoid RSA keys under 4096 bits and never use DSA — it's deprecated and weak. Ed25519 is preferred wherever the target system supports it (any OpenSSH from the last decade).

### 14.2 Firewall Posture: Default Deny

The single most important firewall principle: **default deny inbound, explicit allow only what's needed.**

```bash
# ufw example — clean baseline for a public web server
ufw default deny incoming
ufw default allow outgoing
ufw allow 2222/tcp comment 'SSH (custom port)'
ufw allow 80/tcp comment 'HTTP (redirects to HTTPS)'
ufw allow 443/tcp comment 'HTTPS'
ufw enable
```

Log dropped packets so you can investigate scanning/attack patterns later:

```bash
ufw logging on
```

For nftables, an explicit default-drop policy with logging:

```
table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        ct state established,related accept
        iif lo accept
        tcp dport { 2222, 80, 443 } accept
        counter log prefix "nft-dropped: " drop
    }
}
```

### 14.3 fail2ban — Automated Brute-Force Blocking

```bash
apt install fail2ban
```

```ini
# /etc/fail2ban/jail.local  (never edit jail.conf directly — it gets overwritten on update)
[DEFAULT]
bantime  = 3600
findtime = 600
maxretry = 4
backend  = systemd

[sshd]
enabled = true
port    = 2222

[nginx-http-auth]
enabled = true
```

```bash
systemctl enable --now fail2ban
fail2ban-client status                    # ★ list active jails
fail2ban-client status sshd                  # see current bans for a jail
fail2ban-client set sshd unbanip 1.2.3.4        # manually unban an IP
```

### 14.4 Kernel Hardening via sysctl

```bash
# /etc/sysctl.d/99-hardening.conf

# Network hardening
net.ipv4.conf.all.accept_redirects = 0        # ★ ignore ICMP redirects (spoofing/MITM vector)
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0        # ★ disable source-routed packets
net.ipv4.conf.all.log_martians = 1                  # log packets with impossible addresses
net.ipv4.icmp_echo_ignore_broadcasts = 1               # ★ mitigate smurf attacks
net.ipv4.tcp_syncookies = 1                               # ★ SYN flood protection
net.ipv4.conf.all.rp_filter = 1                              # reverse path filtering — anti-spoofing
net.ipv6.conf.all.accept_ra = 0                                 # disable IPv6 router advertisements if unused

# Kernel hardening
kernel.randomize_va_space = 2         # ★ full ASLR
kernel.kptr_restrict = 2                 # hide kernel pointers from unprivileged users
kernel.dmesg_restrict = 1                   # restrict dmesg to privileged users
fs.suid_dumpable = 0                           # ★ don't core-dump setuid processes (leaks memory contents)
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
```

```bash
sysctl -p /etc/sysctl.d/99-hardening.conf     # ★ apply immediately
sysctl -a | grep randomize_va_space               # verify a specific setting
```

### 14.5 Mandatory Access Control: SELinux & AppArmor

Standard Unix permissions are discretionary (the owner decides). Mandatory Access Control adds a system-wide policy that even root can't casually override — a compromised process is still confined to what its *policy* allows, not just its uid.

**SELinux (RHEL/Fedora/CentOS):**

```bash
getenforce                          # ★ current mode: Enforcing / Permissive / Disabled
sestatus                               # detailed status
setenforce 1                              # switch to Enforcing (runtime only, not persistent)
# persistent: edit /etc/selinux/config, set SELINUX=enforcing

ls -Z /var/www/html                          # ★ view SELinux context on files
semanage fcontext -a -t httpd_sys_content_t "/srv/web(/.*)?"    # define a context rule
restorecon -Rv /srv/web                                            # ★ apply context to matching files

ausearch -m avc -ts recent                                            # ★ find recent SELinux denials
sealert -a /var/log/audit/audit.log                                      # human-readable denial analysis (needs setroubleshoot)
getsebool -a | grep httpd                                                    # list booleans (toggles) for a service
setsebool -P httpd_can_network_connect on                                       # ★ persistent boolean change
```

Never just `setenforce 0` (disable) to "fix" a denial in production — that's disabling the entire protection layer. Instead, identify the specific denial with `ausearch`/`audit2allow` and create a targeted policy exception.

**AppArmor (Debian/Ubuntu):**

```bash
aa-status                        # ★ show loaded profiles and their mode
aa-enforce /etc/apparmor.d/usr.sbin.nginx     # ★ set a profile to enforcing
aa-complain /etc/apparmor.d/usr.sbin.nginx       # set to complain-only (logs but doesn't block, for testing)
aa-logprof                                          # interactively review denials and update the profile
journalctl -k | grep -i apparmor                       # view AppArmor denials in the kernel log
```

### 14.6 Auditd — System Call Auditing

`auditd` logs security-relevant events at the kernel level — file access, privilege escalation, config changes — independent of application logs. Essential for compliance (PCI-DSS, HIPAA, SOC2) and forensics.

```bash
systemctl enable --now auditd
auditctl -w /etc/passwd -p wa -k passwd_changes        # ★ watch a file for writes/attribute changes
auditctl -w /etc/shadow -p wa -k shadow_changes
auditctl -a always,exit -F arch=b64 -S execve -k exec_commands   # log all command executions
auditctl -l                                                          # list active rules

ausearch -k passwd_changes                                              # ★ search logs by rule key
aureport --summary                                                          # summary report of all audit events
```

Persist rules in `/etc/audit/rules.d/audit.rules` so they survive reboots.

### 14.7 Minimizing Attack Surface

```bash
systemctl list-units --type=service --state=running    # ★ what's actually running — disable anything unneeded
ss -tulnp                                                  # ★ what's actually listening — every open port is attack surface
apt list --installed | wc -l                                 # audit installed package count; remove unused software
rpm -qa --last | head -20                                       # recently installed packages (RHEL)
```

Disable unused services rather than just stopping them (a reboot re-enables a merely-stopped service):

```bash
systemctl disable --now avahi-daemon cups bluetooth    # example: services rarely needed on a headless server
```

### 14.8 File Integrity Monitoring

```bash
apt install aide
aideinit                                    # ★ build the initial database (baseline of "known good" state)
mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
aide --check                                    # ★ compare current state to baseline, run via cron
```

Pair this with a cron job that emails/alerts on unexpected changes to critical paths (`/etc`, `/bin`, `/sbin`, `/usr/bin`) — this is one of the highest-signal ways to detect a compromised system.

### 14.9 Automatic Security Updates

```bash
# Debian/Ubuntu
apt install unattended-upgrades
dpkg-reconfigure --priority=low unattended-upgrades
# Then verify /etc/apt/apt.conf.d/50unattended-upgrades only allows security-origin updates
```

```bash
# RHEL/Fedora
dnf install dnf-automatic
systemctl enable --now dnf-automatic.timer
# Edit /etc/dnf/automatic.conf: apply_updates = yes
```

Automatic *security* updates are almost always worth the small risk of an unattended change; automatic *full* upgrades on a production server are riskier and better handled via scheduled maintenance windows with testing.

### 14.10 Hardening Checklist Summary

| Area | Action |
|---|---|
| SSH | Keys only, no root login, non-default port, `AllowUsers`, modern ciphers |
| Firewall | Default-deny inbound, explicit allow list, logging enabled |
| Auth | fail2ban on all exposed auth services, PAM lockout policy, strong password policy |
| Kernel | sysctl hardening (`accept_redirects=0`, `syncookies=1`, ASLR, etc.) |
| MAC | SELinux/AppArmor enforcing, not permissive/disabled |
| Auditing | auditd watching sensitive files, journald persistent and retained |
| Updates | Automatic security patching enabled |
| Surface | Unused services disabled, unused packages removed, minimal open ports |
| Integrity | AIDE or similar baseline + periodic check |
| Secrets | No plaintext credentials in scripts/repos; use env files with restrictive perms or a secrets manager |
| Backups | Tested, offsite/immutable, see Part 17 |

---

<a id="part-15-monitoring-performance"></a>
## Part 15: Monitoring, Performance & Tuning

### 15.1 CPU

```bash
top -o %CPU                     # ★ sort by CPU usage
mpstat -P ALL 2                    # per-core utilization every 2 seconds
vmstat 2                              # ★ CPU, memory, I/O, and context-switch snapshot every 2s
sar -u 2 5                               # historical CPU usage (needs sysstat installed and running)
perf top                                    # ★ live symbol-level CPU profiling (what functions are hot)
perf stat -p PID                               # hardware counters (cache misses, branch mispredicts) for a process
```

Reading `vmstat` output: high `us` (user) means application CPU-bound; high `sy` (system) means heavy syscalls/kernel work; high `wa` (I/O wait) means processes are blocked on disk, not actually CPU-limited — check `iostat` next.

### 15.2 Memory

```bash
free -h                              # ★ total/used/free/buffers/cache
cat /proc/meminfo                       # detailed breakdown
vmstat -s                                  # memory stats summary
smem -tk                                      # ★ better than `free` — accounts for shared memory correctly per-process
slabtop                                          # kernel slab allocator usage (kernel-level memory)
```

A key gotcha: Linux aggressively uses free RAM for disk cache (the `buff/cache` column in `free -h`). This is *available*, not wasted — the kernel reclaims it instantly under pressure. Don't panic at low "free" memory; look at the `available` column instead.

### 15.3 Disk I/O

```bash
iostat -x 2                    # ★ %util close to 100 = disk is the bottleneck
iotop -oa                          # ★ accumulated I/O per process since iotop started
blktrace -d /dev/sda -o trace       # low-level block layer tracing (advanced debugging)
```

### 15.4 Network

```bash
nload eth0                    # ★ live bandwidth
ss -s                            # connection state summary
tcpdump -i eth0 -c 100 -nn         # quick capture for troubleshooting a connectivity issue
mtr example.com                       # ★ combines ping + traceroute, shows per-hop packet loss/latency live
```

### 15.5 Putting It Together: A Systematic Approach to "The Server Is Slow"

1. `uptime` — is load average actually high? Relative to core count?
2. `top`/`htop` — which process, and is it CPU, memory, or blocked in I/O wait (`D` state)?
3. `vmstat 2` — confirm `us`/`sy`/`wa` breakdown across a few samples, not one snapshot.
4. If CPU-bound: `perf top` to find the hot function/process.
5. If I/O-bound: `iostat -x 2` and `iotop` to find the disk and the process.
6. If memory-bound: `free -h`, check for swapping (`si`/`so` columns in `vmstat`) — swapping under load is a major performance killer.
7. If network-bound: `ss -s`, `nload`, check for saturated bandwidth or excessive `TIME_WAIT` connections indicating connection churn.
8. Check application-level logs and metrics — infrastructure tools tell you *where*, application logs tell you *why*.

### 15.6 Long-Term Monitoring Stacks (brief overview)

For anything beyond single-server ad hoc debugging, invest in persistent metrics + alerting:

- **Prometheus + Grafana** — pull-based metrics collection with `node_exporter` for host-level stats, visualized in Grafana dashboards, alerting via Alertmanager.
- **Netdata** — near-zero-config, gives immediate per-second dashboards out of the box, good for a single-box quick win.
- **ELK/Loki stack** — centralized log aggregation and search, essential once you have more than one or two servers to correlate.

The command-line tools above remain essential even with a monitoring stack in place — dashboards tell you *that* something is wrong; the shell is still how you dig into *why*, live, on the box.

---

<a id="part-16-automation"></a>
## Part 16: Automation — Cron, Timers, and Config Management

### 16.1 Cron Basics

```bash
crontab -e              # ★ edit current user's crontab
crontab -l                 # list current user's crontab
crontab -u alice -e           # edit another user's crontab (as root)
```

Cron syntax: `minute hour day-of-month month day-of-week command`

```cron
# ★ Run every day at 2:30 AM
30 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1

# Every 15 minutes
*/15 * * * * /opt/scripts/healthcheck.sh

# Every Monday at 9am
0 9 * * 1 /opt/scripts/weekly_report.sh

# First day of every month at midnight
0 0 1 * * /opt/scripts/monthly_cleanup.sh
```

Always redirect output (`>> logfile 2>&1`) — cron jobs fail silently otherwise, and you'll only find out when something downstream breaks. System-wide cron jobs also live in `/etc/cron.d/`, `/etc/cron.daily/`, `/etc/cron.hourly/`, etc., run as whatever user is specified in the file (system crontabs have an extra user field that per-user crontabs don't).

### 16.2 systemd Timers vs. Cron

Covered in depth in 9.3 — prefer timers for anything already systemd-managed (get `journalctl` logging, dependency ordering, and `Persistent=` catch-up for free). Cron remains simpler for quick one-off scheduled scripts on systems you don't want to manage as full services.

### 16.3 Configuration Management: Why and When

Once you're managing more than a handful of servers, manually SSHing in to make changes stops scaling and starts causing drift (server A has a setting server B doesn't, and nobody remembers why). Configuration management tools describe desired state declaratively and enforce it.

**Ansible** is the most approachable starting point — agentless (just needs SSH), uses YAML playbooks:

```yaml
# playbook.yml
- hosts: webservers
  become: true
  tasks:
    - name: Ensure nginx is installed
      apt:
        name: nginx
        state: present
        update_cache: true

    - name: Ensure nginx is running and enabled
      systemd:
        name: nginx
        state: started
        enabled: true

    - name: Deploy site config
      template:
        src: templates/site.conf.j2
        dest: /etc/nginx/sites-available/mysite.conf
      notify: reload nginx

  handlers:
    - name: reload nginx
      systemd:
        name: nginx
        state: reloaded
```

```bash
ansible-playbook -i inventory.ini playbook.yml --check    # ★ dry run, shows what WOULD change
ansible-playbook -i inventory.ini playbook.yml               # apply for real
ansible all -i inventory.ini -m ping                            # connectivity check across all hosts
ansible webservers -i inventory.ini -a "uptime"                    # ad hoc command across a group
```

Other tools in this space worth knowing about: **Puppet** and **Chef** (agent-based, older, still common in enterprise), **Terraform** (infrastructure provisioning rather than in-host config, but frequently paired with Ansible), and **cloud-init** (declarative first-boot configuration for cloud VM images — the standard way to bootstrap a fresh cloud instance with users, packages, and SSH keys already in place).

### 16.4 A Practical Automation Pattern: Idempotent Bootstrap Script

For teams not ready for full config management, a well-structured idempotent shell script gets most of the benefit:

```bash
#!/usr/bin/env bash
set -euo pipefail

log() { echo "[bootstrap] $*"; }

ensure_package() {
  dpkg -s "$1" &>/dev/null || { log "Installing $1"; apt-get install -y "$1"; }
}

ensure_user() {
  id "$1" &>/dev/null || { log "Creating user $1"; useradd -m -s /bin/bash "$1"; }
}

ensure_line_in_file() {
  local line="$1" file="$2"
  grep -qF -- "$line" "$file" || echo "$line" >> "$file"
}

log "Updating package index"
apt-get update -qq

for pkg in nginx fail2ban ufw unattended-upgrades; do
  ensure_package "$pkg"
done

ensure_user deploy
ensure_line_in_file "deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart myapp" /etc/sudoers.d/deploy

log "Bootstrap complete"
```

Every function here checks state before acting — running it twice is safe, which is the entire point of automation you can trust.

---

<a id="part-17-backup-recovery"></a>
## Part 17: Backup & Disaster Recovery

### 17.1 The 3-2-1 Rule

Keep **3** copies of data, on **2** different media types, with **1** copy offsite. Anything less isn't really a backup strategy — RAID, snapshots, and replication are all *availability* tools, not substitutes for a true independent backup that survives a ransomware event, an admin mistake, or a whole-datacenter failure.

### 17.2 File-Level Backups with rsync

```bash
rsync -avz --delete /srv/data/ backup-host:/backups/srv-data/     # ★ mirror, deletes files removed at source
rsync -avz --link-dest=/backups/latest /srv/data/ /backups/2026-09-05/   # ★ incremental via hardlinks (rsnapshot-style)
ln -sfn /backups/2026-09-05 /backups/latest                                # update "latest" pointer
```

The `--link-dest` pattern gives you space-efficient incremental snapshots: unchanged files are hardlinked (zero extra space), changed files are copied fresh — every snapshot directory looks like a complete backup, but only changed data actually consumes new disk space.

### 17.3 Database Backups

**PostgreSQL:**

```bash
pg_dump -U postgres -Fc mydb > mydb_$(date +%F).dump    # ★ custom format, supports selective restore
pg_dumpall -U postgres > all_databases_$(date +%F).sql      # everything including roles
pg_restore -U postgres -d mydb mydb_2026-09-05.dump             # ★ restore
```

**MySQL/MariaDB:**

```bash
mysqldump -u root -p --single-transaction --all-databases > backup_$(date +%F).sql   # ★ consistent snapshot without locking
mysql -u root -p mydb < backup_2026-09-05.sql       # restore
```

For large, always-on databases, prefer native replication/point-in-time-recovery tools (`pg_basebackup` + WAL archiving for Postgres, binary log replication for MySQL) over dump-based backups alone — dumps get slow and lock-heavy at scale, and don't give you point-in-time recovery.

### 17.4 Full System / Image Backups

```bash
tar --numeric-owner -czvf system-backup.tar.gz --exclude=/proc --exclude=/sys --exclude=/tmp --exclude=/mnt --exclude=/dev / 
```

For actual disaster recovery of a physical/VM disk, block-level tools are more reliable than tar for full-system restores:

```bash
dd if=/dev/sda of=/backups/sda.img bs=4M status=progress    # raw block-level image (large, but exact)
```

Prefer LVM or filesystem snapshots (Part 11.4) combined with `rsync`/`borg`/`restic` for something more space-efficient and restorable at the file level.

### 17.5 Modern Deduplicating Backup Tools

**restic** and **borgbackup** are strongly preferred over raw tar/rsync scripts for anything beyond a single simple mirror — they provide encryption, deduplication, compression, and easy pruning out of the box:

```bash
# restic
restic init --repo /backups/restic-repo                              # ★ initialize an encrypted repo
restic -r /backups/restic-repo backup /srv/data /etc                    # ★ take a snapshot
restic -r /backups/restic-repo snapshots                                   # list snapshots
restic -r /backups/restic-repo restore latest --target /restore/            # ★ restore latest snapshot
restic -r /backups/restic-repo forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune   # ★ retention policy
restic -r /backups/restic-repo check                                          # ★ verify repo integrity — run periodically
```

Ship the repo to offsite/cloud storage (S3, B2, SFTP — restic supports all natively) to satisfy the "1 offsite copy" rule.

### 17.6 Testing Restores — The Step Everyone Skips

A backup you haven't test-restored is a hypothesis, not a backup. Schedule periodic restore drills:

```bash
# Automate a monthly restore-and-verify into a scratch environment
restic -r /backups/restic-repo restore latest --target /tmp/restore-test/
diff -rq /tmp/restore-test/srv/data /srv/data   # sanity check restored content matches expectations
rm -rf /tmp/restore-test
```

Also verify: does the backup *actually* contain what you think? (`restic ls latest`), and does the restore process work from a completely fresh machine with no prior state — the assumption "I'll just restore it" often quietly depends on tools/keys/access that only exist on the box that's currently on fire.

### 17.7 Immutable / Ransomware-Resistant Backups

For ransomware resilience specifically, backups need to be **unwritable from the systems being backed up** — a compromised production server should not be able to delete or encrypt its own backups.

- Use **object lock / WORM storage** (S3 Object Lock, Backblaze B2 immutability) for a period matching your detection window.
- Use **pull-based** backup architecture where the backup server initiates the connection to fetch data, rather than production servers pushing with credentials that could be stolen.
- Keep at least one backup copy with credentials that are not accessible from production systems at all (a genuinely separate trust boundary).

---

<a id="part-18-containers"></a>
## Part 18: Containers Primer

A brief grounding, since containers are now a near-universal deployment unit alongside traditional server management.

### 18.1 Core Concept

Containers use kernel namespaces (isolation: PID, network, mount, etc.) and cgroups (resource limiting: CPU, memory) to give a process an isolated view of the system without the overhead of a full virtual machine. A container shares the host kernel — it is not a VM.

### 18.2 Docker Essentials

```bash
docker ps                            # ★ running containers
docker ps -a                            # all containers, including stopped
docker images                              # local images
docker run -d --name web -p 8080:80 nginx     # ★ run detached, map host:container port
docker exec -it web bash                         # ★ shell into a running container
docker logs -f web                                  # ★ follow logs
docker stop web && docker rm web                       # stop and remove
docker build -t myapp:latest .                            # ★ build an image from a Dockerfile
docker system prune -a                                       # ★ reclaim disk from unused images/containers/networks
```

Minimal production-sensible `docker-compose.yml`:

```yaml
services:
  web:
    image: myapp:latest
    restart: unless-stopped
    ports:
      - "127.0.0.1:8000:8000"     # ★ bind to localhost, let Nginx reverse-proxy handle TLS/public exposure
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/myapp
    depends_on:
      - db
    deploy:
      resources:
        limits:
          memory: 512M
  db:
    image: postgres:16
    restart: unless-stopped
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
volumes:
  pgdata:
```

### 18.3 Container Security Basics

- Run as a **non-root user inside the container** (`USER appuser` in the Dockerfile) — a container escape from a root process is far more dangerous.
- Use minimal base images (`alpine`, `distroless`) to reduce attack surface and image size.
- Never bake secrets into images — use environment variables sourced from a secrets manager, or Docker/Compose secrets, and never commit `.env` files with real credentials.
- Set explicit resource limits (`--memory`, `--cpus`) so one runaway container can't starve the host.
- Scan images for known vulnerabilities: `docker scan myapp:latest` or `trivy image myapp:latest`.
- Keep the Docker daemon socket (`/var/run/docker.sock`) off any container that doesn't explicitly need it — mounting it into a container is equivalent to giving that container root on the host.

### 18.4 When *Not* to Containerize

Containers add real value for reproducible builds, horizontal scaling, and dependency isolation — but a single-purpose database server, or a workload with strict low-latency I/O requirements, is often simpler and more transparent to run and harden as a traditional systemd-managed service directly on the host. Everything in Parts 2–17 of this guide applies just as much inside a container's base OS as it does on bare metal.

---

<a id="part-19-troubleshooting"></a>
## Part 19: Troubleshooting Methodology

Tools are only useful with a method. A general framework:

### 19.1 The Loop

1. **Define the symptom precisely.** "It's slow" is not a symptom. "GET /api/orders takes 8s instead of 200ms since 14:32 UTC" is.
2. **Check what changed.** Deploys, config changes, traffic patterns, dependency updates — `journalctl --since` around the incident window, `git log` on recent deploys.
3. **Reproduce, if possible.** A problem you can trigger on demand is a problem you can actually debug.
4. **Narrow the layer.** Is it network, application, database, disk, or DNS? Each has its own toolset (Parts 12, 15).
5. **Form a hypothesis, test it, don't guess-and-restart.** Restarting a service can mask a symptom without fixing a root cause, and destroys the evidence needed to diagnose it.
6. **Fix, verify, then document.** A postmortem that explains root cause and prevention is worth more long-term than the fix itself.

### 19.2 Service Won't Start

```bash
systemctl status myapp -l              # ★ read the actual error, don't skip this
journalctl -u myapp -n 100 --no-pager     # last 100 log lines
journalctl -u myapp -p err --since today     # errors only, today
systemctl show myapp -p ExecStart               # confirm the actual command being run
sudo -u appuser /opt/myapp/bin/start.sh              # ★ try running it manually as the service user — often reveals permission issues invisible to root
```

### 19.3 "Connection Refused" vs "Connection Timed Out"

These mean different things and point to different layers:

- **Connection refused** — something answered and actively rejected the connection. The service isn't listening on that port/interface, or a firewall sent a `REJECT` (not `DROP`). Check `ss -tulnp` on the target first.
- **Connection timed out** — no response at all. Usually a firewall silently dropping packets (`DROP` policy), a routing issue, or a security group/network ACL in a cloud environment. Check `tcpdump` on both ends if possible, and verify routes with `ip route get`.

### 19.4 Disk Full but `du` Doesn't Show Why

Classic cause: a deleted file still held open by a running process. `du` only sees the directory tree; the kernel keeps disk space allocated as long as a file descriptor is open, even after `rm`.

```bash
df -h                                    # confirms disk is actually full
lsof +L1                                    # ★ lists open files with a link count of 0 — deleted-but-open files
lsof / | grep deleted                          # alternate way to spot the same thing
# once you find the PID, either restart that service (releases the fd) or, more surgically:
: > /proc/PID/fd/N                                # truncate the file in place without killing the process
```

### 19.5 High Load but Low CPU Usage

Almost always processes in uninterruptible sleep (`D` state) — usually blocked on disk or NFS I/O:

```bash
ps aux | awk '$8 ~ /D/ {print}'        # ★ find processes stuck in D state
iostat -x 2                               # confirm a disk is saturated (%util near 100)
```

### 19.6 Sudden Memory Exhaustion / OOM Killer

```bash
dmesg | grep -i "out of memory"        # ★ confirm OOM killer fired, and see what it killed
journalctl -k | grep -i oom
cat /proc/PID/oom_score                    # how likely a process is to be picked next
```

If a specific process is repeatedly OOM-killed, either it has a memory leak (check with `smem`/monitoring over time) or its `Restart=` behavior in the systemd unit and expected memory footprint need alignment — set explicit `MemoryMax=` in the unit file so the *application* fails predictably rather than taking the whole host down with it.

---

<a id="part-20-tips-tricks"></a>
## Part 20: Advanced Tips, Tricks & One-Liners

### 20.1 Shell Productivity

```bash
!!                        # ★ rerun last command
sudo !!                     # ★ rerun last command with sudo (classic "forgot sudo" fix)
!$                            # last argument of previous command
Ctrl+R                          # ★ reverse search command history
Ctrl+A / Ctrl+E                    # jump to start/end of line
Ctrl+U / Ctrl+K                       # cut from cursor to start/end of line
alt+.                                    # insert last argument of previous command (same as !$)
cd -                                        # ★ toggle between last two directories
mkdir -p a/{b,c,d}/{1,2,3}                     # ★ brace expansion — creates a full nested tree instantly
echo {1..10}                                      # 1 2 3 4 5 6 7 8 9 10
echo file{1,2,3}.txt                                 # file1.txt file2.txt file3.txt
```

### 20.2 History Tricks

```bash
history | grep ssh              # ★ search past commands
HISTTIMEFORMAT="%F %T "            # show timestamps in `history` output
HISTCONTROL=ignoredups:erasedups      # dedupe history
HISTSIZE=100000                          # keep a lot of history
shopt -s histappend                         # append to history file instead of overwriting on shell exit
```

### 20.3 Xargs — Turning Output Into Arguments

```bash
find . -name "*.tmp" | xargs rm                       # ★ delete all matches
find . -name "*.tmp" -print0 | xargs -0 rm                # ★ null-delimited — safe with spaces/special chars in filenames
cat urls.txt | xargs -n1 -P4 curl -O                          # ★ 4 parallel downloads
ls *.jpg | xargs -I{} convert {} {}.png                          # rename/transform per-file with a placeholder
```

### 20.4 Watch — Repeat a Command and See What Changes

```bash
watch -n 2 'df -h'                 # ★ re-run every 2 seconds
watch -d 'ss -tan | wc -l'             # -d highlights what changed between runs
```

### 20.5 Useful One-Liners

```bash
# Kill everything using more than 90% CPU (careful, use with real targeting)
ps aux | awk '$3 > 90 {print $2}' | xargs -r kill -9

# Find and remove empty directories
find . -type d -empty -delete

# Recursively change line endings CRLF -> LF
find . -name "*.sh" -exec sed -i 's/\r$//' {} +

# Generate a random strong password
openssl rand -base64 24

# Quickly serve a directory over HTTP for local file transfer
python3 -m http.server 8000

# Check which shell a user has, across all users
awk -F: '{print $1, $7}' /etc/passwd

# Test if a variable is a number
[[ "$var" =~ ^[0-9]+$ ]] && echo "numeric"

# Get your public IP
curl -s ifconfig.me

# Count HTTP status codes in an Nginx log
awk '{print $9}' access.log | sort | uniq -c | sort -rn

# Find the largest 10 files anywhere on the system
find / -xdev -type f -printf '%s %p\n' 2>/dev/null | sort -rn | head -10
```

### 20.6 Bash Traps for Interactive Safety

```bash
# Confirm before rm in interactive shells (put in .bashrc if desired)
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# A safer recursive delete function that requires explicit confirmation
safe_rm() {
  echo "About to delete: $*"
  read -rp "Confirm? [y/N] " ans
  [[ "$ans" == "y" ]] && rm -rf "$@"
}
```

### 20.7 Aliases and Functions Worth Having

```bash
alias ll='ls -alF'
alias grep='grep --color=auto'
alias df='df -h'
alias ports='ss -tulnp'
alias myip='curl -s ifconfig.me'

# Quickly extract almost any archive type
extract() {
  case "$1" in
    *.tar.gz|*.tgz) tar xzf "$1" ;;
    *.tar.bz2) tar xjf "$1" ;;
    *.tar.xz) tar xJf "$1" ;;
    *.zip) unzip "$1" ;;
    *.rar) unrar x "$1" ;;
    *) echo "Unknown archive type: $1" ;;
  esac
}

# cd and ls in one motion
cll() { cd "$1" && ls -la; }
```

---

<a id="part-21-appendix"></a>
## Part 21: Appendix — Cheat Sheets

### 21.1 Permission Numeric Reference

| Number | Permission | Number | Permission |
|---|---|---|---|
| 0 | `---` | 4 | `r--` |
| 1 | `--x` | 5 | `r-x` |
| 2 | `-w-` | 6 | `rw-` |
| 3 | `-wx` | 7 | `rwx` |

Common combos: `755` (owner full, others read+execute — typical for directories/executables), `644` (owner read+write, others read-only — typical for regular files), `600` (owner-only read+write — private keys, secrets), `700` (owner-only full access — private directories).

### 21.2 Signal Reference

| Signal | Number | Meaning |
|---|---|---|
| SIGHUP | 1 | Hangup — often used to trigger config reload |
| SIGINT | 2 | Interrupt (Ctrl+C) |
| SIGQUIT | 3 | Quit with core dump |
| SIGKILL | 9 | Force kill, uncatchable, use as last resort |
| SIGTERM | 15 | Graceful termination request (default for `kill`) |
| SIGSTOP | 19 | Pause process, uncatchable |
| SIGCONT | 18 | Resume a stopped process |

### 21.3 File Type Indicators (`ls -l` first character)

| Char | Type |
|---|---|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character device |
| `b` | Block device |
| `s` | Socket |
| `p` | Named pipe (FIFO) |

### 21.4 Useful Log Locations

| Log | Path |
|---|---|
| Auth (Debian/Ubuntu) | `/var/log/auth.log` |
| Auth (RHEL/Fedora) | `/var/log/secure` |
| Kernel | `/var/log/kern.log` or `journalctl -k` |
| System boot | `journalctl -b` |
| Package manager (Debian) | `/var/log/apt/history.log` |
| Package manager (RHEL) | `/var/log/dnf.log` |
| Nginx | `/var/log/nginx/access.log`, `/var/log/nginx/error.log` |
| Cron | `/var/log/cron.log` or `journalctl -u cron` |

### 21.5 Quick Distro Identification

```bash
cat /etc/os-release       # ★ works across virtually all modern distros
lsb_release -a               # if available
uname -r                        # kernel version
```

### 21.6 Common Port Reference

| Port | Service |
|---|---|
| 22 | SSH |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 3306 | MySQL/MariaDB |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 27017 | MongoDB |
| 9200 | Elasticsearch |

### 21.7 Recommended Reading & Tools to Explore Next

- `man <command>` and `<command> --help` — always the first stop; this guide is a map, not a replacement for the territory.
- `tldr <command>` — community-maintained, example-first alternative to man pages.
- ShellCheck — static analysis for bash scripts (mentioned in 5.8), integrate into CI for any repo with shell scripts.
- Ansible / Terraform — once managing more than a few servers by hand becomes painful.
- Prometheus + Grafana, or Netdata — for anything beyond ad-hoc single-server monitoring.
- CIS Benchmarks (Center for Internet Security) — detailed, distro-specific hardening checklists that go well beyond this guide's summary in Part 14.

---

*End of guide. This document intentionally favors depth on the areas that most affect production reliability and security — bash scripting discipline, hardening, and systematic troubleshooting — over exhaustive coverage of every tool that exists. Use `man` and the tools' own documentation to go deeper on any single topic.*
