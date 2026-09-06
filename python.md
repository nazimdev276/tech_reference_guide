# The Comprehensive Python Standard Library & Automation Guide

*A deep-dive reference to the 20 most useful built-in Python libraries, plus practical automation patterns and command-line workflows using `uv`.*

> This guide assumes you already know core Python syntax (variables, `if`/`elif`, loops, functions, classes). It focuses entirely on the **standard library** — the tools that ship with Python and let you do real work without installing anything — followed by dedicated sections on **task automation** and the **command line with `uv`**.

---

## Table of Contents

1. [os](#1-os)
2. [sys](#2-sys)
3. [pathlib](#3-pathlib)
4. [shutil](#4-shutil)
5. [subprocess](#5-subprocess)
6. [datetime](#6-datetime)
7. [time](#7-time)
8. [collections](#8-collections)
9. [itertools](#9-itertools)
10. [functools](#10-functools)
11. [re (regular expressions)](#11-re-regular-expressions)
12. [json](#12-json)
13. [csv](#13-csv)
14. [argparse](#14-argparse)
15. [logging](#15-logging)
16. [threading, multiprocessing & concurrent.futures](#16-threading-multiprocessing--concurrentfutures)
17. [sqlite3](#17-sqlite3)
18. [urllib & http](#18-urllib--http)
19. [random & secrets](#19-random--secrets)
20. [math & statistics](#20-math--statistics)
21. [Bonus modules worth knowing](#21-bonus-modules-worth-knowing)
22. [Part II — Automating Tasks with Python](#part-ii--automating-tasks-with-python)
23. [Part III — Command Line Tips & Managing Python with uv](#part-iii--command-line-tips--managing-python-with-uv)

---

## 1. `os`

The `os` module is your interface to the operating system: environment variables, processes, file paths (the older, lower-level way), directory listing, permissions, and more.

### Key concepts & functions

**Environment variables**
```python
import os

os.environ["MY_VAR"]                  # raises KeyError if missing
os.environ.get("MY_VAR", "default")   # safe access
os.environ["MY_VAR"] = "value"        # set for this process (and children)
os.getenv("MY_VAR", "default")        # same as os.environ.get
```

**Working directory & paths**
```python
os.getcwd()                 # current working directory
os.chdir("/tmp")            # change working directory
os.path.join("a", "b.txt")  # 'a/b.txt' (OS-correct separators)
os.path.abspath("file.txt")
os.path.basename("/a/b/c.txt")   # 'c.txt'
os.path.dirname("/a/b/c.txt")    # '/a/b'
os.path.splitext("file.tar.gz")  # ('file.tar', '.gz')
os.path.exists(path)
os.path.isfile(path) / os.path.isdir(path)
os.path.getsize(path)       # bytes
os.path.getmtime(path)      # last modified, epoch seconds
```
> Note: `pathlib` (section 3) is the modern, object-oriented replacement for most of `os.path`. Use `os` for environment/process work, `pathlib` for path manipulation.

**Directory operations**
```python
os.listdir(path=".")            # list names (not full paths) in a dir
os.mkdir("new_dir")             # single directory; errors if parents missing
os.makedirs("a/b/c", exist_ok=True)  # create nested dirs, no error if exists
os.rmdir("empty_dir")           # remove EMPTY directory
os.removedirs("a/b/c")          # remove nested empty dirs upward
os.rename(src, dst)
os.remove(path)                 # delete a file
os.walk(top)                    # generator: (dirpath, dirnames, filenames) recursively
```

`os.walk` is the classic tool for recursively processing a directory tree:
```python
for dirpath, dirnames, filenames in os.walk("project"):
    for f in filenames:
        if f.endswith(".py"):
            print(os.path.join(dirpath, f))
```

**Process & system info**
```python
os.getpid()          # current process ID
os.cpu_count()       # number of logical CPUs
os.name              # 'posix' or 'nt'
os.uname()           # (POSIX only) system info
os.system("ls -la")  # run a shell command (prefer subprocess, see §5)
```

**Permissions**
```python
os.chmod(path, 0o755)
os.access(path, os.R_OK)   # check readability without opening
```

### Common keyword/parameter gotchas
- `os.makedirs(path, exist_ok=True)` — always use `exist_ok=True` in scripts to avoid crashing on re-runs.
- `os.listdir()` returns **names only**, not full paths — a frequent source of bugs; join with `os.path.join(base, name)`.
- On Windows vs POSIX, path separators differ — always build paths with `os.path.join`/`pathlib`, never hardcode `/` or `\`.

---

## 2. `sys`

`sys` exposes interpreter-level state: command-line arguments, the module search path, standard streams, and exit codes. It's essential for writing scripts and CLI tools.

### Key attributes & functions

```python
import sys

sys.argv            # list of command-line args; sys.argv[0] is the script name
sys.exit(0)          # exit the program with a status code (0 = success)
sys.path             # list of directories Python searches for modules
sys.platform         # 'linux', 'darwin', 'win32'
sys.version          # Python version string
sys.version_info     # named tuple, e.g. sys.version_info >= (3, 10)
sys.modules          # dict of currently loaded modules
sys.maxsize          # largest Py_ssize_t value
sys.stdin / sys.stdout / sys.stderr  # standard stream file objects
```

**Reading arguments**
```python
# script.py foo.txt --verbose
print(sys.argv)  # ['script.py', 'foo.txt', '--verbose']
```
For anything beyond trivial arg handling, use `argparse` (section 14) instead of parsing `sys.argv` manually.

**Writing to stderr / controlling exit codes** (important for CLI tools & automation scripts that other tools depend on):
```python
def main():
    if error_condition:
        print("Something went wrong", file=sys.stderr)
        sys.exit(1)   # non-zero = failure, useful in shell scripts / CI
    sys.exit(0)
```

**Piping / reading stdin** (useful for Unix-style pipelines):
```python
for line in sys.stdin:
    process(line.rstrip("\n"))
```

**Recursion limit** (rarely needed, but good to know):
```python
sys.setrecursionlimit(3000)
sys.getrecursionlimit()
```

**Manipulating the import path** (useful in scripts that import from a sibling directory):
```python
sys.path.append("/path/to/my/modules")
```

### Notes
- `sys.exit()` raises `SystemExit` — it can be caught with `try/except SystemExit`, but generally shouldn't be.
- Prefer `print(..., file=sys.stderr)` for error/log/diagnostic output so users can redirect stdout (real output) separately from stderr (diagnostics) — a core Unix convention automation scripts should respect.

---

## 3. `pathlib`

`pathlib` (Python 3.4+) is the modern, object-oriented way to work with filesystem paths. It replaces most `os.path` string-fiddling with a `Path` object that has useful methods.

### Creating paths
```python
from pathlib import Path

p = Path("data") / "raw" / "file.csv"   # '/' operator joins paths!
p = Path.cwd()          # current working directory as a Path
p = Path.home()          # user home directory
p = Path(__file__)       # path to current script
```

### Inspecting paths
```python
p.name          # 'file.csv'
p.stem          # 'file'
p.suffix        # '.csv'
p.suffixes      # ['.tar', '.gz'] for 'file.tar.gz'
p.parent        # parent directory (Path)
p.parents[1]    # grandparent
p.parts         # tuple of path components
p.is_absolute()
p.resolve()     # absolute, normalized path (resolves symlinks and '..')
```

### Filesystem queries
```python
p.exists()
p.is_file()
p.is_dir()
p.is_symlink()
p.stat()             # os.stat_result: st_size, st_mtime, etc.
p.stat().st_size
```

### Reading & writing files (shortcuts — no manual `open()`/`close()`)
```python
text = p.read_text(encoding="utf-8")
p.write_text("hello", encoding="utf-8")
data = p.read_bytes()
p.write_bytes(b"...")
```

### Creating / removing
```python
p.mkdir(parents=True, exist_ok=True)   # like os.makedirs
p.touch(exist_ok=True)                 # create empty file / update mtime
p.unlink(missing_ok=True)              # delete file (3.8+ for missing_ok)
p.rmdir()                              # remove empty directory
```

### Globbing (pattern matching within a Path)
```python
list(p.glob("*.txt"))          # non-recursive
list(p.rglob("*.py"))          # recursive (** implicitly)
list(Path(".").glob("**/*.md"))
```

### Renaming / moving
```python
p.rename("new_name.csv")
p.replace("new_name.csv")   # like rename but overwrites on POSIX & Windows
```

### Useful patterns
```python
# Iterate all Python files recursively, skipping virtual envs
for path in Path(".").rglob("*.py"):
    if ".venv" in path.parts:
        continue
    print(path)

# Ensure a directory exists before writing into it
out_dir = Path("output")
out_dir.mkdir(parents=True, exist_ok=True)
(out_dir / "result.txt").write_text("done")
```

### Why prefer `pathlib` over `os.path`
- Chainable, readable object syntax (`path.parent.parent / "x"`) instead of nested `os.path.join`/`os.path.dirname` calls.
- Cross-platform by design.
- Built-in `read_text`/`write_text` remove a huge amount of `open()`/`with` boilerplate for simple cases.

---

## 4. `shutil`

`shutil` ("shell utilities") handles high-level file operations that `os` doesn't: copying, moving, deleting entire trees, archiving, and disk usage — the things you'd normally reach for `cp -r`, `mv`, `rm -rf`, or `tar` for.

### Copying
```python
import shutil

shutil.copy(src, dst)            # copy file (not metadata like permissions fully)
shutil.copy2(src, dst)           # copy file + preserve metadata (mtime, permissions)
shutil.copyfile(src, dst)        # copy file contents only (dst must be a file path)
shutil.copytree(src, dst, dirs_exist_ok=True)   # recursively copy a directory tree
```

### Moving & removing
```python
shutil.move(src, dst)            # move file or directory (works across filesystems)
shutil.rmtree(path)              # recursively delete a directory tree (like rm -rf)
shutil.rmtree(path, ignore_errors=True)
```

### Archiving
```python
shutil.make_archive("backup", "zip", root_dir="my_folder")   # creates backup.zip
shutil.make_archive("backup", "gztar", root_dir="my_folder")  # .tar.gz
shutil.unpack_archive("backup.zip", extract_dir="restored")
shutil.get_archive_formats()     # list supported formats
```

### Disk usage & "which"
```python
total, used, free = shutil.disk_usage("/")
shutil.which("git")              # find path to executable, like Unix `which`
```

### Terminal size (handy for CLI tools)
```python
cols, rows = shutil.get_terminal_size()
```

### Practical pattern: safe copy-then-delete "move"
```python
def archive_and_clean(src_dir, archive_name):
    shutil.make_archive(archive_name, "zip", src_dir)
    shutil.rmtree(src_dir)
```

### Gotchas
- `copytree` fails if the destination already exists **unless** you pass `dirs_exist_ok=True` (Python 3.8+).
- `rmtree` permanently deletes — no undo. Always double-check the path (ideally resolve it with `pathlib` first) before calling it in an automation script.
- `move` behaves like `mv`: if `dst` is an existing directory, the source is moved *inside* it, not renamed to it.

---

## 5. `subprocess`

`subprocess` lets Python launch and control external programs — the correct, modern replacement for `os.system`. Essential for automation scripts that shell out to other tools (git, ffmpeg, curl, system utilities, etc.).

### The one function you need most: `subprocess.run`
```python
import subprocess

result = subprocess.run(
    ["ls", "-la"],       # list of args (preferred: avoids shell injection issues)
    capture_output=True, # capture stdout/stderr
    text=True,           # decode bytes to str automatically
    check=True,          # raise CalledProcessError on non-zero exit
    cwd="/tmp",           # working directory for the subprocess
    timeout=30,           # seconds before raising TimeoutExpired
    env={**os.environ, "MY_VAR": "1"},  # custom environment
)

print(result.stdout)
print(result.returncode)
```

### Key parameters of `run()`
| Parameter | Purpose |
|---|---|
| `args` | Command as a list (safe) or string (only with `shell=True`) |
| `capture_output=True` | Capture `stdout`/`stderr` into the result object |
| `text=True` (or `universal_newlines=True`) | Return strings instead of bytes |
| `check=True` | Raise `subprocess.CalledProcessError` if exit code != 0 |
| `shell=True` | Run through the system shell — enables pipes/globs, but **avoid with untrusted input** (shell injection risk) |
| `cwd` | Directory to run the command in |
| `env` | Dict of environment variables for the subprocess |
| `timeout` | Kill the process if it runs longer than N seconds |
| `input="text"` | Send data to the process's stdin |

### Common patterns
```python
# Simple fire-and-check
subprocess.run(["git", "pull"], check=True)

# Capture output for further processing
out = subprocess.run(["git", "rev-parse", "HEAD"], capture_output=True, text=True).stdout.strip()

# Piping input to a process
subprocess.run(["sort"], input="banana\napple\n", text=True, capture_output=True).stdout

# Handling failures gracefully
try:
    subprocess.run(["cmd", "that", "might", "fail"], check=True)
except subprocess.CalledProcessError as e:
    print("Failed with code", e.returncode, e.stderr)

# Streaming a long-running command's output in real time
with subprocess.Popen(["ping", "-c", "4", "example.com"], stdout=subprocess.PIPE, text=True) as proc:
    for line in proc.stdout:
        print("LIVE:", line.strip())
```

### `Popen` (lower-level, for streaming / concurrent processes)
`run()` is a convenience wrapper around `Popen`. Use `Popen` directly when you need to interact with a process while it's running (read output line-by-line, write to stdin interactively, or manage several processes concurrently).

### Best practices for automation scripts
1. **Prefer list args over `shell=True`** — avoids injection bugs and quoting headaches.
2. **Always set `check=True`** unless you're explicitly handling non-zero exit codes yourself.
3. **Set `timeout=`** for anything that could hang (network calls, external tools).
4. **Use `shutil.which()`** first if you need to confirm a binary exists before calling it.

---

## 6. `datetime`

The `datetime` module handles dates, times, and durations — indispensable for logging, scheduling, and any automation involving timestamps.

### Core classes
- `datetime.date` — year, month, day
- `datetime.time` — hour, minute, second, microsecond
- `datetime.datetime` — combination of date + time
- `datetime.timedelta` — a duration/difference between two datetimes
- `datetime.timezone` — fixed-offset timezone info

### Getting the current time
```python
from datetime import datetime, date, timedelta, timezone

datetime.now()                 # local naive datetime
datetime.now(timezone.utc)     # timezone-aware UTC datetime
date.today()                   # today's date
```

### Creating specific dates/times
```python
datetime(2026, 9, 5, 14, 30, 0)      # year, month, day, hour, min, sec
date(2026, 9, 5)
```

### Formatting (datetime → string): `strftime`
```python
now = datetime.now()
now.strftime("%Y-%m-%d")             # '2026-09-05'
now.strftime("%Y-%m-%d %H:%M:%S")    # '2026-09-05 14:30:00'
now.strftime("%A, %B %d, %Y")        # 'Saturday, September 05, 2026'
now.isoformat()                       # '2026-09-05T14:30:00.123456'
```

### Parsing (string → datetime): `strptime`
```python
datetime.strptime("2026-09-05", "%Y-%m-%d")
datetime.strptime("05/09/2026 14:30", "%d/%m/%Y %H:%M")
```

**Common format codes**: `%Y` (4-digit year), `%y` (2-digit), `%m` (month), `%d` (day), `%H` (24h hour), `%I` (12h hour), `%M` (minute), `%S` (second), `%p` (AM/PM), `%A`/`%a` (full/short weekday), `%B`/`%b` (full/short month), `%Z` (timezone name), `%z` (UTC offset).

### Arithmetic with `timedelta`
```python
tomorrow = datetime.now() + timedelta(days=1)
one_week_ago = datetime.now() - timedelta(weeks=1)
duration = datetime(2026, 1, 1) - datetime(2025, 1, 1)   # timedelta
duration.days             # 365
duration.total_seconds()  # duration expressed in seconds
```

### Comparing dates
```python
if datetime.now() > deadline:
    print("overdue")
```

### Timezones
Naive datetimes (no tzinfo) are dangerous for automation across timezones/servers. Prefer timezone-aware datetimes:
```python
from datetime import timezone
utc_now = datetime.now(timezone.utc)
```
For full IANA timezone database support (e.g. `"Europe/Paris"`, DST-aware), use the standard library's `zoneinfo` (3.9+):
```python
from zoneinfo import ZoneInfo
paris_time = datetime.now(ZoneInfo("Europe/Paris"))
```

### Converting to/from Unix timestamps
```python
datetime.fromtimestamp(1700000000)                     # local time
datetime.fromtimestamp(1700000000, tz=timezone.utc)     # UTC
datetime.now().timestamp()                              # → float epoch seconds
```

---

## 7. `time`

`time` is lower-level than `datetime` — closer to the OS clock. It's what you reach for when pausing execution, measuring performance, or working with raw epoch timestamps.

### Sleeping (the #1 automation use case)
```python
import time

time.sleep(2.5)   # pause execution for 2.5 seconds
```

### Measuring elapsed time
```python
start = time.perf_counter()   # highest-resolution monotonic clock — best for benchmarking
# ... do work ...
elapsed = time.perf_counter() - start

# time.monotonic() is similar but slightly lower resolution guarantee; never goes backwards
# time.time() gives wall-clock epoch seconds — DO NOT use for measuring durations
# (it can jump if the system clock is adjusted, e.g. by NTP)
```

### Current time as epoch / struct
```python
time.time()               # float, seconds since epoch (Jan 1 1970 UTC)
time.localtime()          # struct_time in local timezone
time.gmtime()             # struct_time in UTC
time.strftime("%Y-%m-%d", time.localtime())
```

### Practical automation pattern: polling loop
```python
def wait_until_file_exists(path, poll_interval=1, timeout=60):
    start = time.monotonic()
    while not path.exists():
        if time.monotonic() - start > timeout:
            raise TimeoutError(f"{path} did not appear in time")
        time.sleep(poll_interval)
```

### `time` vs `datetime` — when to use which
- Use **`time.sleep`** for pausing/throttling in scripts (rate-limiting API calls, retry backoff).
- Use **`time.perf_counter`/`time.monotonic`** for measuring durations/benchmarks.
- Use **`datetime`** for anything involving calendar dates, formatting for humans, or timezone-aware logic.

---

## 8. `collections`

`collections` provides specialized, high-performance container types beyond the built-in `list`/`dict`/`tuple`/`set`.

### `Counter` — counting hashable items
```python
from collections import Counter

c = Counter(["a", "b", "a", "c", "a", "b"])
c                      # Counter({'a': 3, 'b': 2, 'c': 1})
c.most_common(2)       # [('a', 3), ('b', 2)]
c["a"]                 # 3
c["z"]                 # 0 (missing keys default to 0, no KeyError!)
c.update(["a", "z"])   # add more counts
c + Counter(["a"])     # combine counters
sum(c.values())        # total count of all items
```
Great for word-frequency counts, log analysis, or tallying anything.

### `defaultdict` — dicts with automatic default values
```python
from collections import defaultdict

d = defaultdict(list)
d["fruits"].append("apple")   # no KeyError, list auto-created
d["fruits"].append("banana")
# d == {'fruits': ['apple', 'banana']}

count = defaultdict(int)
for word in text.split():
    count[word] += 1   # no need to check "if word in count"
```

### `OrderedDict` — dict that remembers insertion order (mostly historical; regular `dict` is ordered since 3.7, but `OrderedDict` still adds `move_to_end()` and is explicit about order-sensitivity)
```python
from collections import OrderedDict
od = OrderedDict()
od["a"] = 1
od.move_to_end("a")            # move key to the end
od.move_to_end("a", last=False)  # move to the front
```

### `namedtuple` — lightweight, immutable classes with named fields
```python
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])
p = Point(1, 2)
p.x, p.y      # 1, 2
p[0], p[1]    # 1, 2 (still tuple-like)
p._replace(x=10)   # returns a NEW namedtuple with x changed
p._asdict()         # OrderedDict-like mapping
```
Great lightweight alternative to a full class when you just need a labeled tuple of data. (For more features, consider `typing.NamedTuple` or `dataclasses.dataclass`.)

### `deque` — double-ended queue, O(1) appends/pops from both ends
```python
from collections import deque

dq = deque([1, 2, 3])
dq.append(4)         # add to right
dq.appendleft(0)      # add to left
dq.pop()               # remove from right
dq.popleft()           # remove from left
dq.rotate(1)            # rotate right by 1
dq = deque(maxlen=5)    # bounded deque — auto-discards oldest when full (great for "last N" buffers)
```
Use `deque` instead of `list` whenever you need to push/pop from **both ends** frequently — a `list`'s `pop(0)`/`insert(0, x)` are O(n); `deque`'s equivalents are O(1).

### `ChainMap` — view multiple dicts as one
```python
from collections import ChainMap
combined = ChainMap(user_settings, default_settings)
combined["theme"]   # checks user_settings first, then default_settings
```
Useful for layered configuration (CLI args > env vars > config file > defaults).

---

## 9. `itertools`

`itertools` provides fast, memory-efficient iterator building blocks — essential for looping patterns that would otherwise require nested loops or extra memory.

### Infinite iterators
```python
import itertools

itertools.count(10, 2)        # 10, 12, 14, 16, ... (start, step)
itertools.cycle([1, 2, 3])    # 1, 2, 3, 1, 2, 3, ... forever
itertools.repeat("x", 3)      # 'x', 'x', 'x'
```

### Combinatorics
```python
list(itertools.permutations([1, 2, 3]))          # all orderings, length 3
list(itertools.permutations([1, 2, 3], 2))        # all orderings of length 2
list(itertools.combinations([1, 2, 3], 2))         # [(1,2), (1,3), (2,3)] — order doesn't matter
list(itertools.combinations_with_replacement([1, 2], 2))  # [(1,1), (1,2), (2,2)]
list(itertools.product([1, 2], ["a", "b"]))         # cartesian product: [(1,'a'), (1,'b'), (2,'a'), (2,'b')]
list(itertools.product([0, 1], repeat=3))            # all 3-bit combinations
```

### Combining/chaining iterables
```python
list(itertools.chain([1, 2], [3, 4], [5]))    # [1, 2, 3, 4, 5] — flatten several iterables
list(itertools.chain.from_iterable([[1, 2], [3, 4]]))  # same, but from a single iterable-of-iterables
```

### Grouping
```python
data = [("a", 1), ("a", 2), ("b", 3)]
for key, group in itertools.groupby(data, key=lambda x: x[0]):
    print(key, list(group))
# a [('a', 1), ('a', 2)]
# b [('b', 3)]
```
**Gotcha:** `groupby` only groups *consecutive* matching elements — sort your data by the grouping key first if it isn't already grouped.

### Slicing iterators
```python
list(itertools.islice(range(100), 5))          # first 5 items: [0,1,2,3,4]
list(itertools.islice(range(100), 10, 15))      # items 10-14
list(itertools.islice(itertools.count(), 5))     # take 5 from an infinite iterator
```

### Filtering
```python
list(itertools.filterfalse(lambda x: x % 2 == 0, range(10)))  # keep odd numbers
list(itertools.takewhile(lambda x: x < 5, [1, 3, 5, 2, 1]))    # [1, 3] (stops at first failure)
list(itertools.dropwhile(lambda x: x < 5, [1, 3, 5, 2, 1]))    # [5, 2, 1] (drops until first failure, then keeps rest)
```

### Accumulation
```python
list(itertools.accumulate([1, 2, 3, 4]))                       # [1, 3, 6, 10] running sum
list(itertools.accumulate([1, 2, 3, 4], func=lambda a, b: a*b)) # running product: [1, 2, 6, 24]
```

### Practical automation pattern: batching an iterable
```python
def batched(iterable, n):
    it = iter(iterable)
    while batch := list(itertools.islice(it, n)):
        yield batch

for batch in batched(range(23), 5):
    process(batch)   # processes in chunks of 5
```
(Python 3.12+ actually ships `itertools.batched` natively.)

---

## 10. `functools`

`functools` provides tools for working with functions and callables — caching, partial application, decorators, and functional-style reductions.

### `lru_cache` / `cache` — memoization
```python
from functools import lru_cache, cache

@lru_cache(maxsize=None)   # unbounded cache
def fib(n):
    return n if n < 2 else fib(n-1) + fib(n-2)

@cache   # Python 3.9+, same as lru_cache(maxsize=None), simpler syntax
def expensive(x):
    ...

fib.cache_info()    # CacheInfo(hits=.., misses=.., maxsize=.., currsize=..)
fib.cache_clear()   # wipe the cache
```
Enormously useful for speeding up repeated/recursive computation or expensive I/O lookups within a single run.

### `partial` — pre-fill some arguments of a function
```python
from functools import partial

def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
square(5)   # 25

# Common automation use: pre-configuring a callback
import subprocess
run_in_dir = partial(subprocess.run, cwd="/my/project")
run_in_dir(["ls"])
```

### `reduce` — cumulative reduction of an iterable to a single value
```python
from functools import reduce

reduce(lambda acc, x: acc + x, [1, 2, 3, 4])          # 10 (like sum, but generalizable)
reduce(lambda acc, x: acc * x, [1, 2, 3, 4], 1)        # 24 (product, with an initial value)
```

### `wraps` — write well-behaved decorators
```python
from functools import wraps

def timer(func):
    @wraps(func)   # preserves func.__name__, __doc__, etc.
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.perf_counter() - start:.3f}s")
        return result
    return wrapper

@timer
def slow_task():
    time.sleep(1)
```
Without `@wraps`, `slow_task.__name__` would become `"wrapper"` — `wraps` fixes introspection/debugging for decorated functions.

### `total_ordering` — generate comparison methods from a minimal set
```python
from functools import total_ordering

@total_ordering
class Version:
    def __init__(self, num): self.num = num
    def __eq__(self, other): return self.num == other.num
    def __lt__(self, other): return self.num < other.num
    # total_ordering fills in __le__, __gt__, __ge__ automatically
```

### `singledispatch` — function overloading based on argument type
```python
from functools import singledispatch

@singledispatch
def describe(obj):
    return f"object: {obj}"

@describe.register
def _(obj: int):
    return f"integer: {obj}"

@describe.register
def _(obj: list):
    return f"list of {len(obj)} items"
```

---

## 11. `re` (regular expressions)

`re` provides regex-based pattern matching and text manipulation — a core tool for parsing logs, validating input, and text automation.

### Core functions
```python
import re

re.search(pattern, string)     # find FIRST match anywhere in string; returns Match or None
re.match(pattern, string)      # match only at the START of string
re.fullmatch(pattern, string)  # match must consume the ENTIRE string
re.findall(pattern, string)    # returns a LIST of all matches (strings, or tuples if groups)
re.finditer(pattern, string)   # returns an ITERATOR of Match objects (memory-efficient)
re.sub(pattern, repl, string)  # substitute matches with repl string/function
re.split(pattern, string)      # split string by pattern
re.compile(pattern)            # precompile a pattern for reuse/performance
```

### Match objects
```python
m = re.search(r"(\d{4})-(\d{2})-(\d{2})", "Date: 2026-09-05")
m.group(0)      # '2026-09-05' — full match
m.group(1)      # '2026' — first capture group
m.groups()      # ('2026', '09', '05') — all groups
m.start(), m.end()  # positions of the match in the original string
m.span()             # (start, end) tuple
```

### Named groups
```python
m = re.search(r"(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})", "2026-09-05")
m.group("year")     # '2026'
m.groupdict()        # {'year': '2026', 'month': '09', 'day': '05'}
```

### Common pattern syntax cheat-sheet
| Syntax | Meaning |
|---|---|
| `.` | any character except newline |
| `\d` `\D` | digit / non-digit |
| `\w` `\W` | word character (alnum+`_`) / non-word |
| `\s` `\S` | whitespace / non-whitespace |
| `^` `$` | start / end of string (or line, with `re.MULTILINE`) |
| `*` `+` `?` | 0+, 1+, 0-or-1 of preceding token |
| `{n,m}` | between n and m repetitions |
| `[abc]` `[^abc]` | character class / negated class |
| `(...)` | capture group |
| `(?:...)` | non-capturing group |
| `(?=...)` `(?!...)` | positive / negative lookahead |
| `\|` | alternation (OR) |
| `\b` | word boundary |

### Flags
```python
re.search(pattern, string, re.IGNORECASE)   # or re.I
re.compile(pattern, re.MULTILINE)            # ^/$ match at line boundaries, not just string ends
re.compile(pattern, re.DOTALL)                # '.' also matches newline
re.compile(pattern, re.VERBOSE)               # allow whitespace/comments in the pattern for readability
```

### Practical patterns
```python
# Extract all email addresses from text
emails = re.findall(r"[\w.+-]+@[\w-]+\.[\w.-]+", text)

# Replace all whitespace runs with a single space
clean = re.sub(r"\s+", " ", messy_text).strip()

# Replace using a function (dynamic replacement)
def upper_match(m):
    return m.group(0).upper()
result = re.sub(r"\berror\b", upper_match, log_text, flags=re.IGNORECASE)

# Precompiling for performance when reused many times (e.g. in a loop over thousands of lines)
LOG_PATTERN = re.compile(r"^\[(?P<level>\w+)\] (?P<message>.+)$")
for line in log_lines:
    if m := LOG_PATTERN.match(line):
        print(m["level"], m["message"])
```

### When NOT to use regex
For structured formats like HTML/XML/JSON, use a proper parser (`json`, `html.parser`, `xml.etree.ElementTree`, or a library like BeautifulSoup) — regex on HTML is a classic trap for edge cases (nested tags, attributes with special characters, etc.).

---

## 12. `json`

`json` reads and writes JSON (JavaScript Object Notation) — the near-universal format for config files, API responses, and data interchange.

### Core functions
```python
import json

json.dumps(obj)                     # Python object → JSON string
json.dumps(obj, indent=2)           # pretty-printed, 2-space indent
json.dumps(obj, sort_keys=True)     # deterministic key order (great for diffs)
json.loads(json_string)             # JSON string → Python object

json.dump(obj, file_obj)            # write directly to an open file
json.load(file_obj)                 # read directly from an open file
```

### Reading/writing files
```python
from pathlib import Path
import json

# Write
with open("config.json", "w") as f:
    json.dump({"name": "script", "retries": 3}, f, indent=2)

# Read
with open("config.json") as f:
    config = json.load(f)

# Shortcut with pathlib
data = json.loads(Path("config.json").read_text())
Path("out.json").write_text(json.dumps(data, indent=2))
```

### Type mapping
| Python | JSON |
|---|---|
| `dict` | object |
| `list`, `tuple` | array |
| `str` | string |
| `int`, `float` | number |
| `True`/`False` | true/false |
| `None` | null |

### Handling non-standard types
`json` can't natively serialize `datetime`, custom classes, sets, etc. — you need a custom encoder:
```python
from datetime import datetime

def default_encoder(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    if isinstance(obj, set):
        return list(obj)
    raise TypeError(f"Not serializable: {type(obj)}")

json.dumps({"when": datetime.now(), "tags": {"a", "b"}}, default=default_encoder)
```

### Custom decoding (object_hook)
```python
def as_datetime(d):
    if "timestamp" in d:
        d["timestamp"] = datetime.fromisoformat(d["timestamp"])
    return d

json.loads(raw, object_hook=as_datetime)
```

### Useful parameters
- `ensure_ascii=False` — keep non-ASCII characters (e.g. accented letters, emoji) as-is instead of escaping to `\uXXXX`.
- `separators=(",", ":")` — compact output with no extra whitespace (good for minimizing file size).
- `indent=2` — human-readable pretty printing.

### Common errors
- `json.JSONDecodeError` — raised on malformed JSON; wrap `json.loads` in `try/except json.JSONDecodeError` when parsing untrusted/external input.
- JSON has **no trailing commas** and **no comments** — unlike Python dict/list literals.

---

## 13. `csv`

`csv` reads and writes CSV (comma-separated values) files correctly — handling quoting, embedded commas, and newlines within fields, which naive `line.split(",")` code gets wrong.

### Reading
```python
import csv

with open("data.csv", newline="") as f:   # newline="" is REQUIRED on all platforms for csv
    reader = csv.reader(f)
    header = next(reader)
    for row in reader:
        print(row)   # row is a list of strings

# Better: DictReader — rows as dicts keyed by header
with open("data.csv", newline="") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row["name"], row["email"])
```

### Writing
```python
with open("out.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["name", "email"])
    writer.writerows([["Alice", "a@x.com"], ["Bob", "b@x.com"]])

# DictWriter — write dicts, using fieldnames to fix column order
with open("out.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "email"])
    writer.writeheader()
    writer.writerow({"name": "Alice", "email": "a@x.com"})
```

### Dialects & delimiters
```python
csv.reader(f, delimiter=";")           # semicolon-separated (common in EU locales)
csv.reader(f, delimiter="\t")          # TSV (tab-separated)
csv.reader(f, quotechar='"', quoting=csv.QUOTE_MINIMAL)

# Sniff the dialect of an unknown CSV file automatically
sample = open("mystery.csv").read(2048)
dialect = csv.Sniffer().sniff(sample)
```

### Quoting options
- `csv.QUOTE_MINIMAL` (default) — quote only fields containing special characters.
- `csv.QUOTE_ALL` — quote every field.
- `csv.QUOTE_NONNUMERIC` — quote all non-numeric fields; read back non-quoted fields as floats.
- `csv.QUOTE_NONE` — never quote (requires an escape character).

### Gotcha: always open with `newline=""`
Without `newline=""`, the `csv` module's own line-ending logic can conflict with Python's universal newline translation, causing blank rows on Windows. This is the single most common `csv` bug.

### When to reach for something bigger
For heavy data wrangling (filtering, joining, aggregating large CSVs), the standard library `csv` module is fine for simple I/O, but consider `pandas` once analysis gets complex — that's outside the standard library, though.

---

## 14. `argparse`

`argparse` builds professional command-line interfaces: positional/optional arguments, help text, type validation, subcommands — everything needed to turn a script into a proper CLI tool.

### Basic setup
```python
import argparse

parser = argparse.ArgumentParser(
    prog="mytool",
    description="Do something useful with files.",
)
parser.add_argument("input", help="Path to the input file")             # positional (required)
parser.add_argument("-o", "--output", help="Path to output file", default="out.txt")
parser.add_argument("-v", "--verbose", action="store_true", help="Enable verbose logging")
parser.add_argument("-n", "--number", type=int, default=1, help="How many times to run")
parser.add_argument("--mode", choices=["fast", "safe"], default="safe")

args = parser.parse_args()
print(args.input, args.output, args.verbose, args.number, args.mode)
```
Running `mytool.py --help` auto-generates a formatted help message from these definitions — no extra work needed.

### Key `add_argument` parameters
| Parameter | Purpose |
|---|---|
| `type=` | Cast the string input, e.g. `type=int`, `type=float`, `type=Path` |
| `default=` | Value used if the argument is omitted |
| `required=True` | Make an optional flag (`--foo`) mandatory |
| `choices=[...]` | Restrict to a fixed set of valid values |
| `action="store_true"` / `"store_false"` | Boolean flags (no value needed after them) |
| `action="append"` | Allow the flag to be repeated, collecting values into a list |
| `nargs="+"` | Accept one-or-more values into a list |
| `nargs="*"` | Accept zero-or-more values |
| `nargs="?"` | Accept zero-or-one value |
| `help=` | Description shown in `--help` |
| `metavar=` | Custom placeholder name shown in usage/help text |
| `dest=` | Custom attribute name to store the value under |

### Mutually exclusive arguments
```python
group = parser.add_mutually_exclusive_group()
group.add_argument("--quiet", action="store_true")
group.add_argument("--verbose", action="store_true")
# argparse errors if both --quiet and --verbose are given together
```

### Subcommands (like `git commit`, `git push`)
```python
parser = argparse.ArgumentParser(prog="mytool")
subparsers = parser.add_subparsers(dest="command", required=True)

push_parser = subparsers.add_parser("push", help="Push data somewhere")
push_parser.add_argument("target")

pull_parser = subparsers.add_parser("pull", help="Pull data from somewhere")
pull_parser.add_argument("source")

args = parser.parse_args()
if args.command == "push":
    do_push(args.target)
elif args.command == "pull":
    do_pull(args.source)
```

### Full pattern: a proper automation script entry point
```python
#!/usr/bin/env python3
import argparse
import sys

def main():
    parser = argparse.ArgumentParser(description="Backup a directory to a zip file.")
    parser.add_argument("source", type=str, help="Directory to back up")
    parser.add_argument("-o", "--output", default="backup.zip", help="Output archive name")
    parser.add_argument("-v", "--verbose", action="store_true")
    args = parser.parse_args()

    try:
        run_backup(args.source, args.output, verbose=args.verbose)
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    main()
```

---

## 15. `logging`

`logging` is the standard, production-grade way to emit diagnostic output — far more capable than scattering `print()` statements: severity levels, timestamps, multiple output destinations, and per-module control.

### Quick start
```python
import logging

logging.basicConfig(
    level=logging.INFO,                                  # minimum severity to show
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)

logger = logging.getLogger(__name__)   # module-scoped logger — best practice
logger.debug("Detailed diagnostic info")     # not shown at INFO level
logger.info("Normal operational message")
logger.warning("Something unexpected but not fatal")
logger.error("A real problem occurred")
logger.critical("Severe error, may crash the program")
```

### Severity levels (ascending)
`DEBUG` (10) < `INFO` (20) < `WARNING` (30) < `ERROR` (40) < `CRITICAL` (50). Setting `level=logging.WARNING` hides `DEBUG`/`INFO` messages — useful to quiet noisy logs in production while keeping them available for troubleshooting (`level=logging.DEBUG`).

### Logging exceptions with full traceback
```python
try:
    risky_operation()
except Exception:
    logger.exception("risky_operation failed")   # automatically includes the traceback
```

### Logging to a file (and console at the same time)
```python
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

file_handler = logging.FileHandler("app.log")
file_handler.setLevel(logging.DEBUG)

console_handler = logging.StreamHandler()
console_handler.setLevel(logging.WARNING)   # only warnings+ shown on screen

formatter = logging.Formatter("%(asctime)s %(levelname)s %(message)s")
file_handler.setFormatter(formatter)
console_handler.setFormatter(formatter)

logger.addHandler(file_handler)
logger.addHandler(console_handler)
```

### Rotating log files (avoid infinitely growing log files — critical for long-running automation/daemons)
```python
from logging.handlers import RotatingFileHandler, TimedRotatingFileHandler

# Rotate when file reaches 5MB, keep 3 backups
handler = RotatingFileHandler("app.log", maxBytes=5_000_000, backupCount=3)

# Rotate daily at midnight, keep 7 days
handler = TimedRotatingFileHandler("app.log", when="midnight", backupCount=7)
```

### Why not just `print()`?
- Severity filtering (turn off debug noise without deleting code).
- Structured, consistent formatting with timestamps automatically.
- Can route to files, syslog, email, or remote servers via different `Handler` types, all without touching business logic.
- Per-module loggers (`getLogger(__name__)`) let you enable verbose logging for just one part of a large program.

---

## 16. `threading`, `multiprocessing` & `concurrent.futures`

Python has three standard-library approaches to concurrency, each suited to a different problem.

### The Global Interpreter Lock (GIL) — why this matters
CPython has a GIL that prevents true parallel execution of Python bytecode across threads in a single process. That means:
- **`threading`** is great for **I/O-bound** work (network requests, file I/O, waiting on subprocesses) where threads spend most of their time waiting, not computing.
- **`multiprocessing`** is needed for **CPU-bound** work (heavy computation) because it uses separate processes, each with its own GIL/interpreter, achieving real parallelism across cores.

### `threading` — basic usage
```python
import threading

def worker(name):
    print(f"Thread {name} starting")

threads = [threading.Thread(target=worker, args=(i,)) for i in range(5)]
for t in threads:
    t.start()
for t in threads:
    t.join()   # wait for all threads to finish
```

### Thread synchronization: `Lock`
```python
counter = 0
lock = threading.Lock()

def increment():
    global counter
    with lock:          # prevents race conditions when multiple threads touch shared state
        counter += 1
```

### `multiprocessing` — basic usage
```python
import multiprocessing

def square(n):
    return n * n

if __name__ == "__main__":   # REQUIRED on Windows/macOS (spawn method) to avoid infinite process recursion
    with multiprocessing.Pool(processes=4) as pool:
        results = pool.map(square, range(10))
    print(results)   # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

### `concurrent.futures` — the modern, unified high-level API (recommended default)
Provides `ThreadPoolExecutor` and `ProcessPoolExecutor` with the same simple interface — pick threads for I/O-bound, processes for CPU-bound, and swap between them with one line changed.

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed

def fetch(url):
    import urllib.request
    return urllib.request.urlopen(url).read()

urls = ["https://example.com", "https://example.org", "https://example.net"]

# I/O-bound: use threads
with ThreadPoolExecutor(max_workers=5) as executor:
    futures = {executor.submit(fetch, url): url for url in urls}
    for future in as_completed(futures):
        url = futures[future]
        try:
            data = future.result()
            print(url, len(data))
        except Exception as e:
            print(url, "failed:", e)

# CPU-bound: use processes (same API, just swap the executor class)
def heavy_compute(n):
    return sum(i * i for i in range(n))

with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(heavy_compute, [10_000_000] * 4))
```

### `executor.map` vs `submit` + `as_completed`
- `.map(func, iterable)` — simplest; results come back **in submission order**, blocking until each is ready.
- `.submit(func, *args)` + `as_completed(futures)` — more flexible; process results **as they finish** (useful when tasks take varying time and you want to react to the fastest ones first).

### Practical automation example: parallel downloads
```python
from concurrent.futures import ThreadPoolExecutor
import urllib.request
from pathlib import Path

def download(url, dest_dir):
    filename = url.split("/")[-1]
    dest = Path(dest_dir) / filename
    urllib.request.urlretrieve(url, dest)
    return dest

urls = [...]
with ThreadPoolExecutor(max_workers=8) as ex:
    list(ex.map(lambda u: download(u, "downloads"), urls))
```

### `asyncio` (mentioned for completeness)
For very high-volume concurrent I/O (thousands of simultaneous connections), the `asyncio` module offers cooperative single-threaded concurrency with `async`/`await`. It's more complex to adopt than thread pools but scales further for I/O-heavy workloads — worth learning once `ThreadPoolExecutor` starts to feel limiting.

---

## 17. `sqlite3`

`sqlite3` gives you a full embedded SQL database with zero setup — no server, no separate install, just a file on disk. Perfect for automation scripts that need structured, queryable persistent storage without the overhead of a "real" database server.

### Connecting & creating tables
```python
import sqlite3

conn = sqlite3.connect("app.db")     # creates the file if it doesn't exist
conn = sqlite3.connect(":memory:")    # in-memory DB, useful for tests

cur = conn.cursor()
cur.execute("""
    CREATE TABLE IF NOT EXISTS tasks (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        done BOOLEAN DEFAULT 0,
        created_at TEXT
    )
""")
conn.commit()
```

### Inserting data — ALWAYS use parameterized queries
```python
cur.execute(
    "INSERT INTO tasks (title, created_at) VALUES (?, ?)",
    ("Write report", "2026-09-05"),
)
conn.commit()

# Bulk insert
cur.executemany(
    "INSERT INTO tasks (title) VALUES (?)",
    [("Task A",), ("Task B",), ("Task C",)],
)
conn.commit()
```
**Never** use f-strings/`%` formatting to build SQL with user input — that's a SQL injection vulnerability. The `?` placeholder syntax handles escaping safely.

### Querying
```python
cur.execute("SELECT id, title FROM tasks WHERE done = ?", (0,))
rows = cur.fetchall()        # list of tuples
row = cur.fetchone()          # single row (or None)
for row in cur.execute("SELECT * FROM tasks"):   # cursor is iterable directly
    print(row)
```

### Getting dict-like rows instead of tuples
```python
conn.row_factory = sqlite3.Row
cur = conn.cursor()
cur.execute("SELECT * FROM tasks")
row = cur.fetchone()
row["title"]    # access by column name
row["id"]
```

### Transactions & context managers
```python
with conn:   # commits automatically on success, rolls back on exception
    conn.execute("UPDATE tasks SET done = 1 WHERE id = ?", (1,))
```

### Closing connections properly
```python
conn.close()

# Or, as a full context-managed pattern:
with sqlite3.connect("app.db") as conn:
    conn.execute(...)
```
Note: `with sqlite3.connect(...)` only wraps the transaction (commit/rollback) — it does **not** close the connection automatically; call `.close()` explicitly, or wrap in `contextlib.closing()`.

### Practical automation pattern: simple task/state tracker for a script
```python
def get_or_create_db(path="state.db"):
    conn = sqlite3.connect(path)
    conn.execute("CREATE TABLE IF NOT EXISTS processed_files (path TEXT PRIMARY KEY, processed_at TEXT)")
    return conn

def already_processed(conn, filepath):
    return conn.execute(
        "SELECT 1 FROM processed_files WHERE path = ?", (str(filepath),)
    ).fetchone() is not None

def mark_processed(conn, filepath):
    with conn:
        conn.execute(
            "INSERT OR REPLACE INTO processed_files VALUES (?, datetime('now'))",
            (str(filepath),),
        )
```
This pattern — tracking "what's already been done" in a small SQLite file — is extremely common in automation scripts that run repeatedly (e.g. cron jobs that shouldn't reprocess the same files twice).

---

## 18. `urllib` & `http`

The standard library includes HTTP client and server tooling. For serious HTTP client work, most people reach for the third-party `requests` library — but `urllib.request` is always available with zero installs, which matters for lightweight scripts, restricted environments, or dependency-free tools.

### Making a GET request
```python
import urllib.request
import json

with urllib.request.urlopen("https://api.example.com/data", timeout=10) as response:
    status = response.status
    body = response.read().decode("utf-8")
    data = json.loads(body)
```

### GET with query parameters
```python
from urllib.parse import urlencode

params = {"q": "python", "page": 2}
url = "https://api.example.com/search?" + urlencode(params)
```

### POST requests with JSON
```python
import urllib.request
import json

payload = json.dumps({"name": "Alice"}).encode("utf-8")
req = urllib.request.Request(
    "https://api.example.com/users",
    data=payload,
    headers={"Content-Type": "application/json"},
    method="POST",
)
with urllib.request.urlopen(req, timeout=10) as response:
    print(response.status, response.read().decode())
```

### Custom headers (e.g. auth tokens)
```python
req = urllib.request.Request(
    url,
    headers={"Authorization": "Bearer YOUR_TOKEN", "User-Agent": "my-script/1.0"},
)
```

### Error handling
```python
from urllib.error import HTTPError, URLError

try:
    urllib.request.urlopen(url, timeout=10)
except HTTPError as e:
    print("Server returned an error:", e.code, e.reason)
except URLError as e:
    print("Failed to reach server:", e.reason)
```

### Downloading files
```python
urllib.request.urlretrieve("https://example.com/file.zip", "file.zip")
```

### URL parsing/building: `urllib.parse`
```python
from urllib.parse import urlparse, urljoin, quote, unquote

parsed = urlparse("https://example.com/path?query=1")
parsed.scheme, parsed.netloc, parsed.path, parsed.query

urljoin("https://example.com/a/b/", "../c")   # 'https://example.com/a/c'
quote("hello world!")     # 'hello%20world%21' — URL-encode a string
unquote("hello%20world")  # 'hello world' — decode
```

### Simple built-in HTTP server (great for quick local testing/serving files)
```bash
python -m http.server 8000
```
```python
from http.server import HTTPServer, BaseHTTPRequestHandler

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header("Content-Type", "text/plain")
        self.end_headers()
        self.wfile.write(b"Hello, world!")

HTTPServer(("localhost", 8000), Handler).serve_forever()
```

### `urllib.request` vs `requests` (third-party)
`urllib.request` is verbose but dependency-free. `requests` (install via `uv add requests`) offers a much friendlier API (`requests.get(url).json()`), automatic connection pooling, and better ergonomics for anything beyond the simplest calls — worth adding as a dependency for real projects, while `urllib` remains the right choice for zero-dependency scripts.

---

## 19. `random` & `secrets`

### `random` — pseudo-random numbers (NOT for security purposes)
```python
import random

random.random()                 # float in [0.0, 1.0)
random.uniform(1.5, 5.5)        # float in a range
random.randint(1, 10)           # int in [1, 10], inclusive both ends
random.randrange(0, 100, 5)     # like range(): start, stop, step
random.choice(["a", "b", "c"])  # pick one random element
random.choices(["a", "b", "c"], k=5)                     # pick 5, WITH replacement
random.choices(["a", "b"], weights=[0.9, 0.1], k=10)     # weighted random choice
random.sample(["a", "b", "c", "d"], k=2)                  # pick 2, WITHOUT replacement
random.shuffle(my_list)                                    # shuffle a list in place
random.seed(42)                                             # make results reproducible (testing/simulation)
```

### Practical patterns
```python
# Pick a random file from a directory
import random
from pathlib import Path
files = list(Path("photos").glob("*.jpg"))
pick = random.choice(files)

# Shuffle and split a dataset
random.shuffle(data)
split = int(len(data) * 0.8)
train, test = data[:split], data[split:]
```

### `secrets` — cryptographically secure randomness (use for tokens, passwords, security)
```python
import secrets

secrets.token_hex(16)         # random hex string, good for API keys/session tokens
secrets.token_urlsafe(16)     # URL-safe random string
secrets.choice(["a", "b", "c"])   # secure random choice
secrets.randbelow(100)         # secure random int in [0, 100)

# Generate a secure random password
import string
alphabet = string.ascii_letters + string.digits
password = "".join(secrets.choice(alphabet) for _ in range(16))
```
**Rule of thumb: never use `random` for anything security-related** (passwords, tokens, session IDs) — it's not cryptographically secure and its output can potentially be predicted. Always use `secrets` for those cases.

---

## 20. `math` & `statistics`

### `math` — mathematical functions on individual numbers
```python
import math

math.sqrt(16)          # 4.0
math.pow(2, 10)         # 1024.0 (note: ** operator is usually preferred and handles ints)
math.floor(4.7)         # 4
math.ceil(4.2)          # 5
math.trunc(4.7)          # 4 (toward zero)
math.pi, math.e, math.tau, math.inf, math.nan

math.log(100, 10)       # log base 10 of 100 → 2.0
math.log2(8)             # 3.0
math.log10(1000)         # 3.0

math.factorial(5)        # 120
math.gcd(12, 18)          # 6
math.lcm(4, 6)             # 12 (Python 3.9+)

math.sin(math.radians(90))   # trig functions work in radians
math.degrees(math.pi)         # convert radians → degrees

math.isclose(0.1 + 0.2, 0.3)     # True — safe float comparison (avoids ==  precision bugs)
math.isnan(x) / math.isinf(x)     # check for NaN / infinity
math.hypot(3, 4)                    # 5.0 — Euclidean distance / hypotenuse
math.dist((0, 0), (3, 4))            # 5.0 — distance between two points (Python 3.8+)
```

### `statistics` — descriptive statistics on data
```python
import statistics as stats

data = [1, 2, 2, 3, 4, 5, 5, 5]

stats.mean(data)          # arithmetic mean
stats.median(data)         # median
stats.mode(data)            # most common value
stats.stdev(data)            # sample standard deviation
stats.pstdev(data)            # population standard deviation
stats.variance(data)           # sample variance
stats.pvariance(data)           # population variance
stats.quantiles(data, n=4)       # quartiles

stats.fmean(data)                 # faster float-based mean
stats.median_low(data) / stats.median_high(data)   # for even-length datasets
```

### When to reach for `numpy`/`pandas` instead
`math`/`statistics` are perfect for simple, one-off calculations and small datasets. Once you're processing large numeric arrays, doing matrix math, or need vectorized performance, third-party `numpy`/`pandas` (installable via `uv add numpy pandas`) become far more appropriate — they're not standard library, but they're the natural next step.

---

## 21. Bonus modules worth knowing

These didn't make the top 20 as full sections, but come up constantly enough to deserve a quick mention.

### `glob` — shell-style wildcard file matching
```python
import glob
glob.glob("*.txt")                 # non-recursive
glob.glob("**/*.py", recursive=True)  # recursive with **
```
(`pathlib.Path.glob`/`rglob`, shown earlier, is the modern equivalent.)

### `fnmatch` — Unix shell-style pattern matching against strings (not the filesystem)
```python
import fnmatch
fnmatch.fnmatch("report_2026.csv", "report_*.csv")   # True
```

### `tempfile` — safe temporary files/directories
```python
import tempfile

with tempfile.TemporaryDirectory() as tmpdir:
    # tmpdir is auto-deleted when the block exits
    ...

with tempfile.NamedTemporaryFile(delete=False, suffix=".txt") as tmp:
    tmp.write(b"scratch data")
    tmp_path = tmp.name
```
Essential for automation scripts that need scratch space without leaving junk files behind or risking name collisions.

### `zipfile` / `tarfile` — archive creation & extraction (lower-level than `shutil.make_archive`)
```python
import zipfile

with zipfile.ZipFile("archive.zip", "w") as zf:
    zf.write("file1.txt")
    zf.writestr("generated.txt", "content created in memory")

with zipfile.ZipFile("archive.zip") as zf:
    zf.extractall("extracted/")
    print(zf.namelist())

import tarfile
with tarfile.open("archive.tar.gz", "w:gz") as tf:
    tf.add("my_folder")
```

### `hashlib` — hashing (checksums, file integrity, deduplication)
```python
import hashlib

hashlib.sha256(b"some data").hexdigest()

# Hash a large file without loading it all into memory
def file_hash(path, algo="sha256"):
    h = hashlib.new(algo)
    with open(path, "rb") as f:
        for chunk in iter(lambda: f.read(8192), b""):
            h.update(chunk)
    return h.hexdigest()
```
Common automation use: detect whether a file has changed since last run by comparing hashes, or deduplicate files by content.

### `pickle` — serialize arbitrary Python objects (use with caution)
```python
import pickle

with open("state.pkl", "wb") as f:
    pickle.dump(my_object, f)

with open("state.pkl", "rb") as f:
    my_object = pickle.load(f)
```
**Security warning:** never unpickle data from an untrusted source — `pickle.load` can execute arbitrary code. Use `json` instead whenever the data is simple enough to represent as JSON.

### `configparser` — read/write INI-style config files
```python
import configparser

config = configparser.ConfigParser()
config.read("settings.ini")
config["DEFAULT"]["retries"]
config["server"]["host"]
```

### `shlex` — safely split/quote shell-style command strings
```python
import shlex
shlex.split('cp "my file.txt" dest/')   # ['cp', 'my file.txt', 'dest/']
shlex.quote("some value; rm -rf /")      # safely quote a string for shell use
```
Useful when you need to build a command list for `subprocess.run` from a string, or safely embed a value in a shell command.

### `textwrap` — wrapping/indenting/dedenting text blocks
```python
import textwrap
textwrap.fill(long_text, width=70)          # wrap to 70 chars per line
textwrap.dedent("""
    removes
    common leading
    whitespace
""")
textwrap.shorten(long_text, width=50, placeholder="...")
```

### `enum` — named constants with type safety
```python
from enum import Enum, auto

class Status(Enum):
    PENDING = auto()
    RUNNING = auto()
    DONE = auto()

Status.RUNNING.name    # 'RUNNING'
Status.RUNNING.value   # 2
```

### `dataclasses` — boilerplate-free classes for holding data
```python
from dataclasses import dataclass, field

@dataclass
class Task:
    title: str
    done: bool = False
    tags: list = field(default_factory=list)

t = Task("Write guide", tags=["writing"])
print(t)   # Task(title='Write guide', done=False, tags=['writing'])
```
Auto-generates `__init__`, `__repr__`, and `__eq__` — a huge readability win over hand-written classes for simple data containers.

### `contextlib` — tools for building/using context managers
```python
from contextlib import contextmanager, suppress, closing

@contextmanager
def timer(label):
    start = time.perf_counter()
    yield
    print(f"{label}: {time.perf_counter() - start:.3f}s")

with timer("my block"):
    do_work()

with suppress(FileNotFoundError):   # silently ignore this specific exception
    os.remove("maybe_missing.txt")
```

### `typing` — type hints for clearer, tool-checkable code
```python
from typing import Optional, Union, Callable

def process(items: list[str], limit: Optional[int] = None) -> dict[str, int]:
    ...
```
Doesn't affect runtime behavior but massively improves editor autocompletion, static analysis (via `mypy`/`pyright`), and code readability — especially valuable in automation scripts that grow beyond a few dozen lines.

---

# Part II — Automating Tasks with Python

This section covers real-world automation: file management, scheduling, web/API automation, and building scripts that are robust enough to run unattended.

## 1. Anatomy of a good automation script

A script you'll run once is different from one that runs unattended on a schedule. Automation scripts should:

1. **Never crash silently.** Wrap the main logic in `try/except`, log errors, and exit with a non-zero status code on failure so calling systems (cron, CI, task schedulers) know it failed.
2. **Be idempotent where possible.** Running the script twice shouldn't cause duplicate side effects — check "has this already been done?" before doing it (see the SQLite pattern in section 17).
3. **Log instead of print**, so you have a persistent record of what happened on unattended runs.
4. **Accept configuration via CLI args, environment variables, or a config file** — never hardcode paths/credentials in the script body.
5. **Fail fast and loud on missing prerequisites** (missing files, unset environment variables, unreachable services) rather than proceeding with bad state.

```python
#!/usr/bin/env python3
"""A template for a robust automation script."""
import argparse
import logging
import sys
from pathlib import Path

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    handlers=[logging.FileHandler("automation.log"), logging.StreamHandler()],
)
logger = logging.getLogger(__name__)

def main():
    parser = argparse.ArgumentParser(description="Example automation task")
    parser.add_argument("target_dir", type=Path)
    args = parser.parse_args()

    if not args.target_dir.is_dir():
        logger.error("Target directory does not exist: %s", args.target_dir)
        sys.exit(1)

    try:
        run(args.target_dir)
    except Exception:
        logger.exception("Automation task failed")
        sys.exit(1)

    logger.info("Task completed successfully")

def run(target_dir: Path):
    ...

if __name__ == "__main__":
    main()
```

## 2. File & folder automation

Combining `pathlib`, `shutil`, and `hashlib` covers most file-automation needs.

### Bulk rename files
```python
from pathlib import Path

for i, path in enumerate(sorted(Path("photos").glob("*.jpg")), start=1):
    path.rename(path.with_name(f"photo_{i:03d}.jpg"))
```

### Organize files into folders by extension
```python
from pathlib import Path
import shutil

def organize_by_extension(source_dir):
    source = Path(source_dir)
    for file in source.iterdir():
        if file.is_file():
            ext_folder = source / file.suffix.lstrip(".").lower()
            ext_folder.mkdir(exist_ok=True)
            shutil.move(str(file), str(ext_folder / file.name))
```

### Find and remove duplicate files by content hash
```python
import hashlib
from pathlib import Path
from collections import defaultdict

def find_duplicates(directory):
    hashes = defaultdict(list)
    for path in Path(directory).rglob("*"):
        if path.is_file():
            h = hashlib.sha256(path.read_bytes()).hexdigest()
            hashes[h].append(path)
    return {h: paths for h, paths in hashes.items() if len(paths) > 1}
```

### Watch a folder for new files (simple polling approach)
```python
import time
from pathlib import Path

def watch_folder(folder, interval=5):
    seen = set(Path(folder).iterdir())
    while True:
        time.sleep(interval)
        current = set(Path(folder).iterdir())
        new_files = current - seen
        for f in new_files:
            print("New file detected:", f)
            process(f)
        seen = current
```
For production-grade filesystem watching (event-driven, not polling), the third-party `watchdog` package (`uv add watchdog`) is the standard choice and far more efficient than polling.

### Backup a directory with a timestamped archive
```python
import shutil
from datetime import datetime

def backup(source_dir, backup_root="backups"):
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    archive_name = f"{backup_root}/backup_{timestamp}"
    shutil.make_archive(archive_name, "zip", source_dir)
    return f"{archive_name}.zip"
```

## 3. Scheduling automation

Python itself doesn't have a built-in scheduler for running things "every day at 9am" while the machine is off — for that, use your OS's scheduler to *launch* a Python script, or use a long-running Python process with a scheduling loop.

### Option A: OS-level scheduling (recommended for most cases)

**Linux/macOS — cron**
```bash
# Edit crontab
crontab -e

# Run a script every day at 9:00 AM
0 9 * * * /path/to/.venv/bin/python /path/to/script.py >> /path/to/cron.log 2>&1
```
Cron field order: `minute hour day-of-month month day-of-week`.

**Windows — Task Scheduler**
```powershell
schtasks /create /tn "MyPythonTask" /tr "C:\path\to\python.exe C:\path\to\script.py" /sc daily /st 09:00
```

Always use the **full absolute path** to the Python interpreter (ideally the one inside your project's virtual environment) and to the script — cron/Task Scheduler don't use your shell's PATH or working directory by default.

### Option B: In-process scheduling with a loop + `time.sleep`
Good for simple long-running scripts (not ideal for anything that must survive reboots):
```python
import time
import schedule  # third-party: uv add schedule

def job():
    print("Running scheduled task...")

schedule.every(10).minutes.do(job)
schedule.every().day.at("09:00").do(job)
schedule.every().monday.do(job)

while True:
    schedule.run_pending()
    time.sleep(1)
```

### Option C: pure standard-library polling loop (no dependencies)
```python
import time
from datetime import datetime

def run_daily_at(hour, minute, task):
    while True:
        now = datetime.now()
        if now.hour == hour and now.minute == minute:
            task()
            time.sleep(60)   # avoid re-triggering within the same minute
        time.sleep(15)
```

## 4. Web & API automation

### Polling an API and reacting to changes
```python
import urllib.request
import json
import time

def poll_api(url, interval=60):
    last_value = None
    while True:
        with urllib.request.urlopen(url, timeout=10) as resp:
            data = json.loads(resp.read())
        if data["value"] != last_value:
            print("Change detected:", data["value"])
            last_value = data["value"]
        time.sleep(interval)
```

### Downloading many files concurrently
```python
from concurrent.futures import ThreadPoolExecutor
import urllib.request
from pathlib import Path

def download_all(urls, dest="downloads"):
    Path(dest).mkdir(exist_ok=True)
    def _dl(url):
        name = url.split("/")[-1]
        urllib.request.urlretrieve(url, Path(dest) / name)
    with ThreadPoolExecutor(max_workers=8) as ex:
        list(ex.map(_dl, urls))
```

### Sending automated email notifications (standard library `smtplib` + `email`)
```python
import smtplib
from email.message import EmailMessage

def send_email(subject, body, to_addr, from_addr, smtp_host, smtp_port, username, password):
    msg = EmailMessage()
    msg["Subject"] = subject
    msg["From"] = from_addr
    msg["To"] = to_addr
    msg.set_content(body)

    with smtplib.SMTP_SSL(smtp_host, smtp_port) as server:
        server.login(username, password)
        server.send_message(msg)
```
Never hardcode credentials — read them from environment variables (`os.environ`) or a secrets manager.

## 5. Automating other programs

Combine `subprocess` with `shutil.which` to build scripts that orchestrate external CLI tools (git, ffmpeg, rsync, docker, etc.):

```python
import subprocess
import shutil
import sys

def require(binary):
    if shutil.which(binary) is None:
        sys.exit(f"Required tool not found: {binary}")

def convert_video(input_path, output_path):
    require("ffmpeg")
    subprocess.run(
        ["ffmpeg", "-i", input_path, "-vcodec", "libx264", output_path],
        check=True,
        capture_output=True,
    )

def git_sync(repo_dir):
    subprocess.run(["git", "-C", repo_dir, "pull"], check=True)
```

## 6. Retry logic for flaky operations (network calls, external services)

```python
import time
import functools

def retry(times=3, delay=2, backoff=2, exceptions=(Exception,)):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            current_delay = delay
            for attempt in range(1, times + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == times:
                        raise
                    logging.warning("Attempt %d failed (%s), retrying in %ds", attempt, e, current_delay)
                    time.sleep(current_delay)
                    current_delay *= backoff
        return wrapper
    return decorator

@retry(times=4, delay=1, exceptions=(urllib.error.URLError,))
def fetch(url):
    return urllib.request.urlopen(url, timeout=5).read()
```

## 7. Putting it together: a complete example automation script

A script that watches a folder for new CSV files, validates them, loads them into SQLite, archives the original, and logs everything — a realistic composite of the patterns above.

```python
#!/usr/bin/env python3
import argparse
import csv
import logging
import shutil
import sqlite3
import sys
import time
from pathlib import Path

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    handlers=[logging.FileHandler("ingest.log"), logging.StreamHandler()],
)
logger = logging.getLogger(__name__)

def setup_db(db_path):
    conn = sqlite3.connect(db_path)
    conn.execute("""
        CREATE TABLE IF NOT EXISTS records (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT, value REAL, source_file TEXT, ingested_at TEXT
        )
    """)
    return conn

def ingest_file(conn, csv_path):
    with open(csv_path, newline="") as f:
        reader = csv.DictReader(f)
        rows = [(r["name"], float(r["value"]), csv_path.name) for r in reader]
    with conn:
        conn.executemany(
            "INSERT INTO records (name, value, source_file, ingested_at) VALUES (?, ?, ?, datetime('now'))",
            rows,
        )
    logger.info("Ingested %d rows from %s", len(rows), csv_path.name)

def main():
    parser = argparse.ArgumentParser(description="Watch a folder and ingest CSV files into SQLite.")
    parser.add_argument("watch_dir", type=Path)
    parser.add_argument("--db", type=Path, default=Path("data.db"))
    parser.add_argument("--archive", type=Path, default=Path("archive"))
    parser.add_argument("--interval", type=int, default=10)
    args = parser.parse_args()

    args.archive.mkdir(exist_ok=True)
    conn = setup_db(args.db)

    logger.info("Watching %s for CSV files...", args.watch_dir)
    while True:
        for csv_file in args.watch_dir.glob("*.csv"):
            try:
                ingest_file(conn, csv_file)
                shutil.move(str(csv_file), args.archive / csv_file.name)
            except Exception:
                logger.exception("Failed to ingest %s", csv_file)
        time.sleep(args.interval)

if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        sys.exit(0)
```

---

# Part III — Command Line Tips & Managing Python with `uv`

[`uv`](https://docs.astral.sh/uv/) is a modern, extremely fast Python package and project manager (written in Rust) that replaces `pip`, `pip-tools`, `virtualenv`, `pyenv`, and `pipx` with one unified tool. This section covers the `uv` workflow for automation and everyday development, plus general CLI tips for Python scripts.

## 1. Installing `uv`

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

# Or via pip, if you already have Python
pip install uv
```
Verify with:
```bash
uv --version
```

## 2. Managing Python versions with `uv`

`uv` can download and manage Python interpreters itself — no need for `pyenv`.
```bash
uv python install 3.12          # install a specific Python version
uv python install 3.11 3.12 3.13  # install several
uv python list                    # list installed/available versions
uv python pin 3.12                # pin the Python version for the current project (writes .python-version)
```

## 3. Running scripts with `uv run`

```bash
uv run script.py                 # runs script.py using the project's environment (auto-created if needed)
uv run python                     # drop into a REPL using the project's environment
uv run -- python script.py --flag  # pass args after --
```
`uv run` automatically ensures the virtual environment exists and is in sync with your dependencies **before every run** — no manual "activate the venv" step required, which is a huge quality-of-life win for automation (cron jobs, CI) since there's no shell-activation state to get wrong.

### Inline script dependencies (great for standalone automation scripts)
`uv` supports [PEP 723](https://peps.python.org/pep-0723/) inline script metadata — declare a script's dependencies right at the top of the file, and `uv run` will set up an ephemeral environment for it automatically:
```python
# /// script
# dependencies = [
#   "requests",
#   "rich",
# ]
# ///
import requests
from rich import print

print(requests.get("https://example.com").status_code)
```
```bash
uv run myscript.py   # uv reads the header, installs requests+rich into a temp env, and runs it
```
This is the ideal pattern for one-off automation scripts you want to share or schedule via cron **without** managing a separate project/virtualenv for each one.

You can also add dependencies to a script's inline metadata directly:
```bash
uv add --script myscript.py requests rich
```

## 4. Project management with `uv`

### Starting a new project
```bash
uv init myproject          # creates pyproject.toml, a starter script, .python-version, README
cd myproject
```

### Adding / removing dependencies
```bash
uv add requests                  # adds to pyproject.toml AND installs it
uv add "requests>=2.31"          # with a version constraint
uv add --dev pytest ruff          # development-only dependency group
uv remove requests
```

### Syncing the environment to match `pyproject.toml`/`uv.lock`
```bash
uv sync                     # create/update .venv to exactly match the lockfile
uv sync --frozen             # sync without updating the lockfile (fast, reproducible — great for CI)
uv lock                       # regenerate uv.lock without installing
uv lock --upgrade              # upgrade all dependencies to latest allowed versions
```
`uv.lock` should be committed to version control — it pins exact versions for fully reproducible installs across machines, a core requirement for reliable automation deployments.

## 5. `uv` as a drop-in `pip` replacement

For workflows not using the full project system, `uv pip` mirrors `pip`'s interface but runs dramatically faster:
```bash
uv venv                          # create a .venv in the current directory
uv pip install requests          # install into the active/detected venv
uv pip install -r requirements.txt
uv pip freeze > requirements.txt
uv pip list
uv pip uninstall requests
```

## 6. Installing standalone CLI tools with `uv tool` (replaces `pipx`)

For CLI tools you want available globally (not tied to one project's environment) — linters, formatters, generic utilities:
```bash
uv tool install ruff             # installs ruff in an isolated environment, exposes it on PATH
uv tool install black
uv tool list                      # see installed tools
uv tool upgrade ruff
uv tool uninstall ruff

# Run a tool once without installing it permanently
uvx ruff check .                  # 'uvx' is a shorthand for 'uv tool run'
uvx cowsay -t "hello"
```
`uvx` is perfect for automation scripts/CI pipelines that need a one-off tool without polluting the environment permanently.

## 7. Common `uv` automation patterns

**Cron job running a `uv`-managed script:**
```bash
# crontab entry — uv run handles environment setup itself
0 6 * * * cd /opt/myproject && /home/user/.cargo/bin/uv run script.py >> /var/log/myscript.log 2>&1
```

**CI pipeline step (e.g. GitHub Actions):**
```yaml
- name: Set up uv
  uses: astral-sh/setup-uv@v3
- name: Install dependencies
  run: uv sync --frozen
- name: Run tests
  run: uv run pytest
```

**Reproducible one-off automation script, shareable as a single file:**
```bash
# script.py has inline PEP 723 metadata (see §3)
uv run https://raw.githubusercontent.com/you/repo/main/script.py
```

## 8. General command-line tips for Python scripts

### Make scripts directly executable (Unix)
```bash
#!/usr/bin/env python3
# ...script content...
```
```bash
chmod +x script.py
./script.py            # runs directly, no need to type "python script.py"
```

### Reading piped input / building Unix-style filters
```python
import sys
for line in sys.stdin:
    print(line.strip().upper())
```
```bash
cat data.txt | uv run touppercase.py
```

### Exit codes matter for automation chains
```python
import sys
sys.exit(0)   # success
sys.exit(1)   # generic failure
sys.exit(2)   # (by convention) misuse of command / bad arguments
```
```bash
uv run script.py && echo "succeeded" || echo "failed"
```

### Environment variables for configuration (12-factor style)
```python
import os
DEBUG = os.environ.get("DEBUG", "false").lower() == "true"
API_KEY = os.environ["API_KEY"]   # raises KeyError loudly if missing — good, fail fast
```
```bash
API_KEY=secret123 uv run script.py
```

### `.env` files for local secrets (with `python-dotenv`)
```bash
uv add python-dotenv
```
```python
from dotenv import load_dotenv
load_dotenv()   # reads .env into os.environ
```
```
# .env
API_KEY=secret123
DEBUG=true
```
Always add `.env` to `.gitignore` — never commit secrets.

### Profiling a script's runtime from the command line
```bash
uv run python -m cProfile -s cumulative script.py
```

### Running a module as a script
```bash
uv run python -m http.server 8000
uv run python -m json.tool data.json   # pretty-print/validate a JSON file from the CLI
uv run python -m venv .venv             # (rarely needed directly once you're using uv)
```

### Quick REPL one-liners for automation debugging
```bash
uv run python -c "import json,sys; print(json.load(sys.stdin))" < data.json
```

## 9. Suggested `pyproject.toml` shape for an automation project

```toml
[project]
name = "my-automation"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "requests>=2.31",
    "python-dotenv>=1.0",
]

[dependency-groups]
dev = ["pytest>=8.0", "ruff>=0.6"]

[project.scripts]
my-automation = "my_automation.cli:main"   # exposes `uv run my-automation` as a CLI command
```
With `[project.scripts]` defined, `uv run my-automation` (or, once installed as a tool, plain `my-automation`) becomes a first-class command — turning your automation script into a proper installable CLI tool.

---

## Quick reference: standard library cheat-sheet

| Task | Module |
|---|---|
| File paths | `pathlib` |
| Copy/move/delete trees, archives | `shutil` |
| Run external programs | `subprocess` |
| Environment vars, low-level OS | `os`, `sys` |
| Dates & times | `datetime`, `zoneinfo` |
| Sleep, benchmarking | `time` |
| Specialized containers | `collections` |
| Looping patterns | `itertools` |
| Caching, decorators | `functools` |
| Pattern matching in text | `re` |
| Config/data interchange | `json` |
| Tabular data files | `csv` |
| CLI argument parsing | `argparse` |
| Diagnostics/output | `logging` |
| Concurrency (I/O-bound) | `threading`, `concurrent.futures.ThreadPoolExecutor` |
| Concurrency (CPU-bound) | `multiprocessing`, `concurrent.futures.ProcessPoolExecutor` |
| Embedded database | `sqlite3` |
| HTTP requests | `urllib.request` |
| Randomness (general) | `random` |
| Randomness (security) | `secrets` |
| Math on numbers | `math` |
| Stats on datasets | `statistics` |
| Temp files | `tempfile` |
| Hashing/checksums | `hashlib` |
| Archives | `zipfile`, `tarfile` |
| Email | `smtplib`, `email` |
| Data containers (typed) | `dataclasses`, `enum`, `typing` |

---

*End of guide.*
