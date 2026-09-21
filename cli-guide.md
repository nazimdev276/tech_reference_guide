# The Complete CLI Guide: Flags, Options & Use Cases

A deep-dive reference covering classic Unix commands and their modern replacements — every major flag explained, with real use cases for each.

---

## 1. Navigation

### `pwd` — print working directory
```bash
pwd          # /home/user/projects
pwd -P       # resolve symlinks to physical path
```

### `cd` — change directory
| Flag/Form | Meaning |
|---|---|
| `cd path` | go to `path` |
| `cd ..` | go up one level |
| `cd ../..` | go up two levels |
| `cd -` | go to previous directory (toggle) |
| `cd ~` or `cd` | go home |
| `cd ~user` | go to another user's home dir |

### `pushd` / `popd` / `dirs` — directory stack (bash/zsh)
```bash
pushd /var/log       # go there, remember current dir on a stack
popd                  # return to previous dir, popping the stack
dirs -v               # view the stack with indices
```
**Use case:** jumping between 3–4 working directories repeatedly without retyping full paths each time.

### Modern: `zoxide` (`z`)
| Command | Meaning |
|---|---|
| `z foo` | jump to highest-ranked dir matching "foo" |
| `zi foo` | interactive fuzzy picker (needs `fzf`) |
| `zoxide query foo` | show what `z foo` would resolve to, without moving |
| `zoxide add /path` | manually add a directory to the database |
| `zoxide remove /path` | remove a directory from tracking |

**Use case:** After weeks of normal `cd` usage, `z dotfiles` takes you to `~/code/personal/dotfiles` from anywhere, because zoxide ranks by frequency × recency ("frecency").

---

## 2. Listing Files — `ls`

| Flag | Meaning |
|---|---|
| `-l` | long format: permissions, links, owner, group, size, date, name |
| `-a` | show hidden files (dotfiles) including `.` and `..` |
| `-A` | like `-a` but excludes `.` and `..` |
| `-h` | human-readable sizes (used with `-l`) |
| `-t` | sort by modification time, newest first |
| `-r` | reverse sort order |
| `-S` | sort by file size, largest first |
| `-R` | recurse into subdirectories |
| `-d` | list directories themselves, not their contents |
| `-i` | show inode numbers |
| `-G` | colorized output (macOS/BSD; Linux uses `--color`) |
| `-1` | one entry per line |
| `-F` | append indicator (`/` dir, `*` executable, `@` symlink) |

**Common combos:**
```bash
ls -lah          # long, all files, human-readable — the everyday default
ls -lt            # most recently modified files first
ls -lS            # largest files first
ls -d */          # list only directories in current path
```

### Modern: `eza` (maintained fork of `exa`)
| Flag | Meaning |
|---|---|
| `-l` | long format |
| `-a` | show hidden files |
| `--icons` | show file-type icons (needs a Nerd Font) |
| `--git` | show per-file git status (M/N/D markers) |
| `--tree` | render as a tree |
| `--level=N` | limit tree depth to N |
| `-s modified` / `--sort=modified` | sort by modification time |
| `-r` / `--reverse` | reverse sort |
| `-h` / `--header` | show column headers in long view |
| `--group-directories-first` | dirs before files |
| `-@` | show extended attributes |
| `-U` | disable sorting (raw directory order, faster) |

**Use case:**
```bash
eza -la --icons --git --group-directories-first
# a full replacement for `ls -la` that's also git-aware and icon-annotated
eza --tree --level=2 -I "node_modules|.git"
# tree view two levels deep, ignoring noisy dirs
```

---

## 3. Viewing Files

### `cat` — concatenate/print
| Flag | Meaning |
|---|---|
| `-n` | number all lines |
| `-b` | number only non-blank lines |
| `-s` | squeeze multiple blank lines into one |
| `-A` | show all (tabs as `^I`, line ends as `$`, non-printables) |
| `-E` | show `$` at end of each line |

### `less` / `more` — pagers
| Key/Flag | Meaning |
|---|---|
| `less file` | page through file |
| `/pattern` | search forward |
| `?pattern` | search backward |
| `n` / `N` | next/previous search match |
| `g` / `G` | go to top / bottom |
| `q` | quit |
| `less -N file` | show line numbers |
| `less -S file` | don't wrap long lines (scroll horizontally) |

### `head` / `tail`
| Flag | Meaning |
|---|---|
| `head -n 20 file` | first 20 lines |
| `tail -n 20 file` | last 20 lines |
| `tail -f file` | follow file as it grows (live log tailing) |
| `tail -F file` | like `-f`, but re-attaches if file is rotated/recreated |
| `head -c 100 file` | first 100 bytes |

**Use case:** `tail -f /var/log/syslog` is the classic way to watch a log file update in real time.

### Modern: `bat`
| Flag | Meaning |
|---|---|
| `-n` / `--number` | show line numbers (default on) |
| `-A` | show non-printable characters |
| `--diff` | highlight git changes inline |
| `-p` / `--plain` | disable line numbers & decorations (script-friendly) |
| `--theme=NAME` | choose a color theme |
| `-l LANG` | force syntax highlighting language |
| `--paging=never` | disable the internal pager, print directly |
| `-r 10:20` | show only lines 10–20 |

**Use case:**
```bash
bat -r 50:80 server.log       # inspect a specific line range with highlighting
bat --diff main.py             # see uncommitted changes rendered inline
```

---

## 4. Searching File Contents

### `grep`
| Flag | Meaning |
|---|---|
| `-i` | case-insensitive |
| `-v` | invert match (show non-matching lines) |
| `-r` / `-R` | recursive (R follows symlinks) |
| `-n` | show line numbers |
| `-c` | count matching lines instead of printing them |
| `-l` | list only filenames with matches |
| `-L` | list only filenames with NO matches |
| `-w` | match whole words only |
| `-x` | match whole lines only |
| `-A N` | show N lines of context after each match |
| `-B N` | show N lines of context before |
| `-C N` | show N lines of context both before and after |
| `-E` | extended regex (equivalent to `egrep`) |
| `-F` | fixed string, no regex interpretation (equivalent to `fgrep`) |
| `-o` | print only the matched part, not the whole line |
| `--include="*.py"` | limit search to matching filenames |
| `--exclude-dir=node_modules` | skip a directory |

**Use case:**
```bash
grep -rn "TODO" --include="*.js" .          # find all TODOs in JS files with line numbers
grep -c "ERROR" app.log                      # count error lines
grep -oE "[0-9]{3}-[0-9]{4}" contacts.txt      # extract phone-number-like patterns
```

### Modern: `ripgrep` (`rg`)
| Flag | Meaning |
|---|---|
| `-i` | case-insensitive |
| `-S` / `--smart-case` | case-insensitive unless pattern has uppercase |
| `-v` | invert match |
| `-w` | whole word match |
| `-c` | count matches per file |
| `-l` | filenames only |
| `-n` | line numbers (on by default when output is a terminal) |
| `-A/-B/-C N` | context lines, same as grep |
| `-t TYPE` | restrict to a file type (`-t py`, `-t js`) |
| `-T TYPE` | exclude a file type |
| `--type-list` | show all supported type shortcuts |
| `-g GLOB` | include/exclude via glob, e.g. `-g "*.md"` |
| `-u` | reduce "smart filtering" by one level (see hidden files too) |
| `-uu` | ignore `.gitignore` rules entirely |
| `--hidden` | search hidden files/dirs |
| `--no-ignore` | don't respect `.gitignore`/`.ignore` |
| `-F` | fixed-string (literal) search |
| `-x` | match whole lines |
| `-P` | enable PCRE2 regex (lookahead/lookbehind support) |
| `--stats` | print summary of matches/files searched |
| `-j N` | number of threads to use |

**Use case:**
```bash
rg -t py "def old_function"       # search only Python files, gitignore-aware
rg --hidden --no-ignore "API_KEY"   # audit for leaked secrets, including ignored/hidden files
rg -C 2 "panic\(" src/                # see context around Rust panics
```

---

## 5. Finding Files

### `find`
| Flag | Meaning |
|---|---|
| `-name "pat"` | match filename (case-sensitive) |
| `-iname "pat"` | case-insensitive name match |
| `-type f` | files only |
| `-type d` | directories only |
| `-type l` | symlinks only |
| `-mtime -N` | modified less than N days ago |
| `-mtime +N` | modified more than N days ago |
| `-mmin -N` | modified less than N minutes ago |
| `-size +100M` | larger than 100 MB |
| `-size -1k` | smaller than 1 KB |
| `-empty` | empty files or directories |
| `-maxdepth N` | limit recursion depth |
| `-mindepth N` | skip the top N levels |
| `-perm 644` | match exact permission bits |
| `-user NAME` | owned by a specific user |
| `-exec CMD {} \;` | run CMD once per result |
| `-exec CMD {} +` | run CMD once with all results batched (faster) |
| `-delete` | delete matched files (careful!) |
| `-not` / `!` | negate a condition |

**Use case:**
```bash
find . -type f -name "*.tmp" -mtime +7 -delete     # clean up old temp files
find . -type d -empty -delete                        # remove empty directories
find /var/log -size +100M -exec ls -lh {} \;           # find and inspect large log files
```

### Modern: `fd`
| Flag | Meaning |
|---|---|
| `-e ext` | filter by extension, e.g. `-e py` |
| `-t f` / `-t d` | file/directory type |
| `-H` | include hidden files |
| `-I` | disable `.gitignore` filtering |
| `-d N` | max search depth |
| `-a` | show absolute paths |
| `-x CMD` | execute CMD per result (like `find -exec {} \;`) |
| `-X CMD` | execute CMD once with all results batched |
| `-g GLOB` | glob pattern instead of regex |
| `-c always` | force colored output (e.g. when piping) |
| `-S +100M` | size filter, same syntax idea as find |
| `--changed-within 1d` | modified within the last day |

**Use case:**
```bash
fd -e log --changed-within 7d          # recent log files, no regex needed
fd -H -t f "\.env$" -x cat {}             # find and print all .env files, including hidden dirs
fd -t d node_modules -X rm -rf            # delete every node_modules dir in a monorepo
```

---

## 6. Text Processing Pipeline: `sort`, `uniq`, `cut`, `tr`, `awk`, `sed`

### `sort`
| Flag | Meaning |
|---|---|
| `-n` | numeric sort (not lexical) |
| `-r` | reverse order |
| `-k N` | sort by field N |
| `-t ","` | set field delimiter |
| `-u` | unique output (like piping to `uniq`) |
| `-h` | human-numeric sort (understands "10K", "2M") |

### `uniq`
| Flag | Meaning |
|---|---|
| `-c` | prefix lines with count of occurrences |
| `-d` | show only duplicated lines |
| `-u` | show only unique (non-duplicated) lines |
| `-i` | case-insensitive comparison |

**Use case:**
```bash
sort access.log | uniq -c | sort -rn | head -10    # top 10 most frequent lines
```

### `cut`
| Flag | Meaning |
|---|---|
| `-d ","` | set delimiter |
| `-f 1,3` | select fields 1 and 3 |
| `-c 1-10` | select characters 1 through 10 |

```bash
cut -d ":" -f 1 /etc/passwd     # list all usernames
```

### `tr` — translate characters
```bash
tr 'a-z' 'A-Z' < file.txt        # uppercase everything
tr -d '\n' < file.txt              # strip all newlines
tr -s ' ' < file.txt                # squeeze repeated spaces into one
```

### `awk` — field-based text processing
```bash
awk '{print $1}' file.txt              # print first column
awk -F: '{print $1}' /etc/passwd        # custom field separator
awk '$3 > 100 {print $0}' data.txt        # print rows where column 3 > 100
awk '{sum += $2} END {print sum}' file      # sum a column
```

### `sed`
| Flag | Meaning |
|---|---|
| `s/old/new/` | replace first match per line |
| `s/old/new/g` | replace all matches per line |
| `s/old/new/gi` | global, case-insensitive |
| `-i` | edit file in place (GNU); `-i ''` needed on macOS/BSD |
| `-n` | suppress automatic printing (used with `p`) |
| `-e` | chain multiple expressions |
| `/pattern/d` | delete matching lines |
| `Nd` | delete line N |
| `N,Md` | delete line range N to M |

**Use case:**
```bash
sed -i 's/localhost/127.0.0.1/g' config.yml     # in-place replace across a file
sed -n '10,20p' file.txt                          # print only lines 10–20
sed '/^#/d' config.conf                           # strip comment lines
```

### Modern: `sd`
```bash
sd 'old' 'new' file.txt                 # no regex-escaping quirks, straightforward literal replace
sd '(\w+)@(\w+)' '$2@$1' file.txt         # capture-group swap without sed's \1 syntax
sd -p 'foo' 'bar' file.txt                 # preview changes without writing
```
**Use case:** `sd` treats find/replace strings literally by default and uses `$1` for capture groups — far fewer surprises than `sed`'s BRE/ERE inconsistencies across platforms.

---

## 7. Process Management

### `ps`
| Flag | Meaning |
|---|---|
| `aux` | all processes, user-oriented format, no TTY restriction (BSD style) |
| `-ef` | all processes, full format (System V style) |
| `-p PID` | show a specific PID |
| `--sort=-%cpu` | sort by CPU usage descending |
| `-o pid,comm,%mem` | custom output columns |

```bash
ps aux --sort=-%mem | head -10     # top 10 memory-hungry processes
```

### `top` — interactive live view
| Key | Meaning |
|---|---|
| `M` | sort by memory |
| `P` | sort by CPU |
| `k` | kill a process (prompts for PID) |
| `1` | toggle per-core CPU display |
| `q` | quit |

### `kill` / `killall` / `pkill`
| Command | Meaning |
|---|---|
| `kill PID` | send SIGTERM (graceful stop) |
| `kill -9 PID` | send SIGKILL (force stop) |
| `kill -l` | list all available signals |
| `killall name` | kill all processes matching a name |
| `pkill -f pattern` | kill by matching full command line |

### `jobs` / `bg` / `fg` / `&` / `Ctrl+Z`
```bash
long_task &        # run in background
jobs                 # list background jobs
fg %1                 # bring job 1 to foreground
bg %1                  # resume job 1 in background
# Ctrl+Z suspends the current foreground job
```

### Modern: `htop`
| Key | Meaning |
|---|---|
| `F6` | choose sort column |
| `F9` | kill selected process |
| `F5` | tree view |
| `/` | search/filter processes |
| `Space` | tag a process (for batch signaling) |

### Modern: `btop`
- Mouse-clickable panels for CPU (per-core), memory, disks, network, and process tree.
- `Esc` opens the options menu; themes and layout presets are configurable live.

### Modern: `procs`
```bash
procs                     # colorized ps replacement
procs --tree                # process tree
procs --sortd cpu             # sort descending by CPU
procs firefox                  # filter by name automatically (no -ef needed)
```

---

## 8. Disk Usage

### `du`
| Flag | Meaning |
|---|---|
| `-s` | summarize (total only, not every subfile) |
| `-h` | human-readable sizes |
| `-a` | include files, not just directories |
| `-c` | show a grand total |
| `--max-depth=N` | limit summary depth |

```bash
du -sh */                        # size of each top-level dir, human-readable
du -ah . | sort -rh | head -20     # 20 largest files/dirs anywhere below here
```

### `df`
| Flag | Meaning |
|---|---|
| `-h` | human-readable |
| `-T` | show filesystem type |
| `-i` | show inode usage instead of block usage |

```bash
df -h        # free space per mounted filesystem
```

### Modern: `dust`
```bash
dust                    # visual bar-chart tree of biggest space users, current dir
dust -d 2                 # limit depth to 2
dust -n 20                  # show top 20 entries
dust -X node_modules          # exclude a path/pattern
```

### Modern: `duf`
```bash
duf                      # colorized table of all disks/mounts
duf --only local            # skip network/temp filesystems
```

---

## 9. Permissions & Ownership

### `chmod`
| Form | Meaning |
|---|---|
| `chmod 755 file` | rwxr-xr-x (owner full, others read+execute) |
| `chmod 644 file` | rw-r--r-- (owner read/write, others read-only) |
| `chmod +x file` | add execute permission for all |
| `chmod -w file` | remove write permission for all |
| `chmod u+x,g-w file` | symbolic: add exec for user, remove write for group |
| `chmod -R 755 dir/` | recursive |

### `chown`
```bash
chown user file             # change owner
chown user:group file         # change owner and group
chown -R user:group dir/        # recursive
```

### `sudo`
| Flag | Meaning |
|---|---|
| `sudo -i` | interactive root shell |
| `sudo -u user cmd` | run cmd as another user |
| `sudo -l` | list what the current user is allowed to run |
| `sudo !!` | re-run the previous command with sudo |

---

## 10. Networking

### `curl`
| Flag | Meaning |
|---|---|
| `-X METHOD` | HTTP method (GET/POST/PUT/DELETE) |
| `-d DATA` | send data in the request body |
| `-H "Header: val"` | add a header |
| `-I` | fetch headers only (HEAD request) |
| `-o file` | write output to a file |
| `-O` | save with the remote filename |
| `-L` | follow redirects |
| `-s` | silent (no progress meter) |
| `-v` | verbose (show request/response details) |
| `-u user:pass` | basic auth |
| `-k` | skip TLS certificate verification (insecure) |
| `--data-urlencode` | URL-encode form data automatically |

```bash
curl -sL https://example.com/install.sh | bash        # common (risky!) install pattern
curl -X POST -H "Content-Type: application/json" -d '{"a":1}' https://api.example.com
curl -I https://example.com                             # check response headers/status only
```

### `wget`
```bash
wget https://example.com/file.zip
wget -c https://example.com/large.iso     # resume a partial download
wget -r -np https://example.com/docs/       # recursive mirror, no parent dir traversal
```

### `ssh` / `scp`
| Flag | Meaning |
|---|---|
| `ssh -p PORT host` | connect on a non-default port |
| `ssh -i key.pem user@host` | use a specific private key |
| `ssh -L 8080:localhost:80 host` | local port forwarding |
| `scp file user@host:/path` | copy file to remote |
| `scp -r dir user@host:/path` | copy directory recursively |
| `scp user@host:/path/file .` | copy from remote to local |

### `netstat` / Modern: `ss`
```bash
ss -tulpn        # listening TCP/UDP ports with process info (modern replacement for netstat -tulpn)
ss -s              # summary statistics
```

### Modern: `httpie` (`http`) / `xh`
| Usage | Meaning |
|---|---|
| `http GET api.com/users` | simple GET, auto-formatted colorized JSON |
| `http POST api.com/users name=Ada age:=30` | `=` for strings, `:=` for raw JSON types |
| `http --auth user:pass GET api.com` | basic auth |
| `http --download api.com/file.zip` | download with progress bar |
| `xh` | drop-in faster Rust reimplementation, same syntax |

**Use case:** `http POST api.com/login user=admin pass=1234` is far more readable than the equivalent `curl -X POST -H ... -d '{"user":"admin","pass":"1234"}'`.

---

## 11. Archives & Compression

### `tar`
| Flag | Meaning |
|---|---|
| `-c` | create archive |
| `-x` | extract archive |
| `-v` | verbose (list files as processed) |
| `-f FILE` | specify archive filename |
| `-z` | gzip compression |
| `-j` | bzip2 compression |
| `-J` | xz compression |
| `-t` | list contents without extracting |

```bash
tar -czvf archive.tar.gz folder/     # create gzipped archive
tar -xzvf archive.tar.gz              # extract gzipped archive
tar -tvf archive.tar.gz                # list contents without extracting
```

### `zip` / `unzip`
```bash
zip -r archive.zip folder/
unzip archive.zip
unzip -l archive.zip        # list contents only
```

---

## 12. JSON Processing — `jq`

| Flag/Filter | Meaning |
|---|---|
| `.` | pretty-print input |
| `.key` | access a field |
| `.[]` | iterate over array elements |
| `.[0]` | first array element |
| `select(.age > 30)` | filter objects by condition |
| `map(.name)` | transform each element |
| `length` | count elements |
| `keys` | list object keys |
| `-r` | raw output (no quotes around strings) |
| `-c` | compact output (single line) |
| `-s` | slurp: read all inputs into one array |
| `@csv` | format array as CSV row |

```bash
curl -s api.com/users | jq '.[] | select(.active) | .email'    # emails of active users
jq -r '.items[].id' data.json                                     # plain list of IDs
jq -c '.' data.json                                                  # minify JSON
```

---

## 13. Terminal Multiplexing

### `screen`
```bash
screen -S name       # start named session
screen -ls             # list sessions
screen -r name           # reattach
# Ctrl+A D to detach, Ctrl+A K to kill window
```

### Modern: `tmux`
| Command/Key | Meaning |
|---|---|
| `tmux new -s name` | new named session |
| `tmux ls` | list sessions |
| `tmux attach -t name` | reattach |
| `tmux kill-session -t name` | kill a session |
| `Ctrl+B %` | split pane vertically |
| `Ctrl+B "` | split pane horizontally |
| `Ctrl+B arrow` | move between panes |
| `Ctrl+B c` | new window |
| `Ctrl+B n` / `p` | next/previous window |
| `Ctrl+B d` | detach |
| `Ctrl+B [` | enter scroll/copy mode |

**Use case:** start a long-running remote build inside `tmux`, detach, close your laptop, reattach hours later from anywhere — the session survives the SSH disconnect.

---

## 14. Quick Reference Table

| Task | Classic | Modern |
|---|---|---|
| List files | `ls` | `eza` |
| View file | `cat` | `bat` |
| Search text | `grep` | `ripgrep (rg)` |
| Find files | `find` | `fd` |
| Edit stream | `sed` | `sd` |
| Process viewer | `top` / `ps` | `htop` / `btop` / `procs` |
| Disk usage | `du` | `dust` |
| Disk free | `df` | `duf` |
| HTTP client | `curl` | `httpie` / `xh` |
| Diff pager | `diff` | `delta` |
| JSON parsing | manual | `jq` |
| Multiplexer | `screen` | `tmux` |
| Change dir | `cd` | `zoxide` |
| Ports/sockets | `netstat` | `ss` |

---

## 15. Installing the Modern Tools

```bash
# macOS (Homebrew)
brew install eza bat ripgrep fd sd htop btop dust duf httpie jq tmux zoxide git-delta procs

# Ubuntu/Debian (package names sometimes differ)
sudo apt install ripgrep fd-find bat htop jq tmux
# some (eza, dust, sd, zoxide, delta, procs, btop) may need cargo, snap, or a manual release download

# Via cargo (Rust), works cross-platform for most of these
cargo install eza bat ripgrep fd-find sd du-dust zoxide git-delta procs
```

## Closing Notes
- Classic tools (`ls`, `cat`, `grep`, `find`, `sed`, `awk`, `ps`, `du`, `curl`, `tar`, `screen`) are POSIX-standard or near-universal — worth knowing by heart since they exist on any server with zero setup.
- Modern tools trade that universality for speed, sane defaults (gitignore-awareness, color, readable flags), and better output — install them on machines you personally control, but don't rely on them being present on a random remote box.
- Flags shown here follow GNU conventions (Linux); BSD/macOS versions of `sed`, `grep`, etc. sometimes differ slightly (e.g. `sed -i ''` needs an explicit empty string argument on macOS).
