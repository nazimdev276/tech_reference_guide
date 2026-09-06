# The Complete Git Reference — From Daily Workflow to Plumbing

> A study guide and reference. Part 1 covers the "porcelain" (the commands you use
> every day) plus tips and tricks. Part 2 goes under the hood into Git's internals
> and the "plumbing" commands that porcelain is built on.

---

## Table of Contents

**Part 1 — Daily Workflow**
1. [Mental Model: The Three (Four) Areas](#1-mental-model-the-three-four-areas)
2. [Setup & Configuration](#2-setup--configuration)
3. [Starting / Cloning a Repo](#3-starting--cloning-a-repo)
4. [The Core Loop: status, add, commit, diff](#4-the-core-loop-status-add-commit-diff)
5. [Branching & Merging](#5-branching--merging)
6. [Working with Remotes](#6-working-with-remotes)
7. [Undoing Things Safely](#7-undoing-things-safely)
8. [Stashing](#8-stashing)
9. [Tags](#9-tags)
10. [Inspecting History](#10-inspecting-history)
11. [.gitignore & Cleaning](#11-gitignore--cleaning)
12. [Tips, Tricks & Aliases](#12-tips-tricks--aliases)

**Part 2 — Plumbing & Internals**
13. [The Object Model](#13-the-object-model)
14. [Inside .git/](#14-inside-git)
15. [Refs, HEAD & the Ref Namespace](#15-refs-head--the-ref-namespace)
16. [Plumbing Commands Tour](#16-plumbing-commands-tour)
17. [Index (Staging Area) Internals](#17-index-staging-area-internals)
18. [Merges: Strategies & Three-Way Merge](#18-merges-strategies--three-way-merge)
19. [Rebase Internals](#19-rebase-internals)
20. [Packfiles, Compression & gc](#20-packfiles-compression--gc)
21. [Transfer Protocols: fetch/push under the hood](#21-transfer-protocols-fetchpush-under-the-hood)
22. [Reflog & Data Recovery](#22-reflog--data-recovery)
23. [Rewriting History](#23-rewriting-history)
24. [Bisect, Blame, Grep](#24-bisect-blame-grep)
25. [Worktrees & Submodules](#25-worktrees--submodules)
26. [Hooks](#26-hooks)
27. [Sparse Checkout & Partial Clone](#27-sparse-checkout--partial-clone)
28. [Command Cheat Sheet by Use Case](#28-command-cheat-sheet-by-use-case)

---

# PART 1 — DAILY WORKFLOW

## 1. Mental Model: The Three (Four) Areas

Git moves a snapshot of your project through four places. Understanding this
diagram makes almost every command click into place.

```
   ┌───────────────┐   git add    ┌───────────────┐   git commit   ┌───────────────┐
   │  Working Dir  │ ───────────> │  Staging Area │ ─────────────> │  Local Repo   │
   │ (your files)  │              │  (the Index)  │                │  (.git/objects)│
   └───────────────┘ <─────────── └───────────────┘ <───────────── └───────────────┘
          ^      git checkout /          ^         git reset --soft       │
          │      git restore             │                                │
          │                              │                                │ git push
          │         git reset --hard  (moves HEAD+index+worktree)          v
          │                                                        ┌───────────────┐
          └────────────────── git pull / git fetch + merge ─────── │  Remote Repo  │
                                                                    └───────────────┘
```

- **Working directory** — the actual files you edit.
- **Staging area / index** — a snapshot you're building for the *next* commit.
- **Local repository** — committed snapshots, stored as objects in `.git`.
- **Remote repository** — a copy of the repo living elsewhere (GitHub, GitLab, a server).

---

## 2. Setup & Configuration

```bash
$ git config --global user.name  "Ada Lovelace"
$ git config --global user.email "ada@example.com"
$ git config --global init.defaultBranch main
$ git config --global core.editor "vim"
$ git config --global pull.rebase false     # merge (default) | true = rebase | interactive
$ git config --list                         # show effective config
$ git config --global --edit                # open ~/.gitconfig directly
```

Config has three scopes, checked in this priority order (most specific wins):

```
--local   (.git/config)              highest priority, per-repo
--global  (~/.gitconfig)             per-user
--system  (/etc/gitconfig)           machine-wide, lowest priority
```

---

## 3. Starting / Cloning a Repo

```bash
$ git init my-project              # new empty repo
$ git clone <url>                  # copy an existing repo + full history
$ git clone <url> --branch dev     # clone a specific branch
$ git clone <url> --depth 1        # shallow clone (only latest snapshot, faster)
```

---

## 4. The Core Loop: status, add, commit, diff

```bash
$ git status                # what's changed, what's staged
$ git diff                  # working dir  vs  staging area
$ git diff --staged         # staging area vs  last commit  (a.k.a. --cached)
$ git add file.py           # stage a specific file
$ git add -p                # stage interactively, HUNK BY HUNK (very useful!)
$ git add .                 # stage everything in current dir
$ git commit -m "message"   # snapshot the staging area
$ git commit -am "message"  # stage tracked changes AND commit in one step
$ git commit --amend        # fix the last commit (message and/or content)
```

Example `git status` output:

```
$ git status
On branch feature/login
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   src/auth.py

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
        modified:   src/routes.py

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        src/temp_notes.txt
```

**`git add -p` (patch mode)** lets you split one file's edits into multiple
commits — a core "clean history" habit:

```
$ git add -p
diff --git a/src/auth.py b/src/auth.py
@@ -12,6 +12,9 @@ def login(user, pw):
+    if not user:
+        raise ValueError("missing user")
Stage this hunk [y,n,q,a,d,s,e,?]?
```

---

## 5. Branching & Merging

```bash
$ git branch                    # list local branches
$ git branch -a                 # list local + remote branches
$ git branch new-feature        # create a branch (doesn't switch to it)
$ git switch new-feature        # switch to it            (modern)
$ git checkout new-feature      # switch to it            (classic, does more)
$ git switch -c new-feature     # create + switch, one step (modern)
$ git checkout -b new-feature   # create + switch, one step (classic)
$ git branch -d old-feature     # delete a merged branch
$ git branch -D old-feature     # force delete (even if unmerged)
$ git branch -m old-name new-name  # rename
```

Branches are just movable pointers to commits:

```
                     feature
                        │
                        v
   o──o──o──o──o──o─────o
   │              ^
   │              │
   │             main
   │
  (older history)
```

Merging joins two histories:

```bash
$ git switch main
$ git merge feature
```

```
Before:                              After (fast-forward, no diverging commits):
main: A─B                             main: A─B─C─D
feature:  C─D  (main has no          feature:      D
           extra commits)

Before (diverged):                   After (three-way merge, new commit M):
main:    A─B─────E                    main:    A─B─────E─────M
              \                                     \       /
feature:       C─D                    feature:       C─────D
```

- **Fast-forward merge**: `main` just moves the pointer forward, no merge commit.
- **Three-way merge**: creates a new merge commit `M` with two parents.
- Force a merge commit even when fast-forward is possible: `git merge --no-ff feature`

**Conflict markers** look like this in a file:

```
<<<<<<< HEAD
color: blue;
=======
color: green;
>>>>>>> feature
```
Edit to resolve, then `git add <file>` and `git commit` (or `git merge --continue`).

---

## 6. Working with Remotes

```bash
$ git remote -v                       # list remotes and their URLs
$ git remote add origin <url>         # add a remote named "origin"
$ git fetch origin                    # download new commits/branches, don't merge
$ git pull                            # fetch + merge (or rebase, per config)
$ git pull --rebase                   # fetch, then replay your commits on top
$ git push                            # upload your commits
$ git push -u origin main             # push + set upstream tracking branch
$ git push origin --delete old-branch # delete a remote branch
$ git fetch --prune                   # remove local refs to deleted remote branches
```

`fetch` vs `pull`:

```
                 git fetch                    git pull = fetch + merge
   origin/main ──> updated locally      origin/main ──> updated
   main        ──> unchanged            main        ──> merged with origin/main
   (safe, no surprises)                 (your working branch moves)
```

---

## 7. Undoing Things Safely

This is the section people search for most. Know the difference!

```bash
$ git restore file.py                 # discard working-dir changes to a file
$ git restore --staged file.py        # unstage (keep the edits in working dir)
$ git checkout -- file.py             # old-style equivalent of restore

$ git commit --amend -m "new msg"     # edit the most recent commit

$ git reset --soft  HEAD~1            # undo commit, KEEP changes staged
$ git reset --mixed HEAD~1            # undo commit, KEEP changes unstaged (default)
$ git reset --hard  HEAD~1            # undo commit, DISCARD changes  (⚠ destructive)

$ git revert <commit>                 # make a NEW commit that undoes an old one
                                       # (safe for shared/public history)
```

Reset modes visualized (what each one touches):

```
                 Working Dir   Staging Area   HEAD/branch pointer
--soft                -             -                moves
--mixed  (default)    -           reset               moves
--hard              reset         reset               moves
```

Rule of thumb: **`reset` rewrites history (local only) — `revert` is safe to
push because it adds a new commit instead of erasing one.**

---

## 8. Stashing

Temporarily shelve changes without committing:

```bash
$ git stash                    # stash tracked changes
$ git stash -u                 # also stash untracked files
$ git stash list
stash@{0}: WIP on main: 4c2a9f1 add login form
$ git stash show -p stash@{0}  # view the diff
$ git stash pop                # apply + remove from stash list
$ git stash apply stash@{0}    # apply but KEEP it in the list
$ git stash drop stash@{0}     # delete a stash entry
$ git stash branch new-branch  # create a branch from a stash
```

---

## 9. Tags

```bash
$ git tag v1.0.0                              # lightweight tag
$ git tag -a v1.0.0 -m "Release 1.0.0"        # annotated tag (recommended, has metadata)
$ git tag                                     # list tags
$ git push origin v1.0.0                      # push a single tag
$ git push origin --tags                      # push all tags
$ git tag -d v1.0.0                           # delete local tag
$ git push origin --delete v1.0.0             # delete remote tag
```

---

## 10. Inspecting History

```bash
$ git log
$ git log --oneline --graph --all             # the classic "pretty" view
$ git log -p file.py                          # show diffs per commit for a file
$ git log --author="Ada"
$ git log --since="2 weeks ago" --until="yesterday"
$ git log -S"functionName"                    # "pickaxe": find commits adding/removing a string
$ git show <commit>                           # show one commit's metadata + diff
$ git diff main..feature                      # what feature has that main doesn't
$ git diff main...feature                     # diff from the common ancestor (3 dots)
```

`--oneline --graph --all` example output:

```
$ git log --oneline --graph --all
* 9f3a1c4 (HEAD -> main) Merge branch 'feature/login'
|\
| * 7b2e0d1 (feature/login) add password hashing
| * a1c4f09 add login form
* | c3d8e77 fix typo in README
|/
* 4e9a002 initial commit
```

---

## 11. .gitignore & Cleaning

```
# .gitignore example
*.log
node_modules/
.env
build/
!build/keep-this.txt      # exception (un-ignore)
```

```bash
$ git check-ignore -v path/to/file    # debug WHY a file is ignored
$ git clean -n                        # dry run: show what would be deleted
$ git clean -fd                       # delete untracked files AND directories
$ git rm --cached file.txt            # stop tracking a file, keep it on disk
```

---

## 12. Tips, Tricks & Aliases

```bash
# Useful aliases (add to ~/.gitconfig or via `git config --global alias.x "..."`)
$ git config --global alias.st  status
$ git config --global alias.co  checkout
$ git config --global alias.br  branch
$ git config --global alias.cm  commit
$ git config --global alias.lg  "log --oneline --graph --all --decorate"
$ git config --global alias.last "log -1 HEAD --stat"
$ git config --global alias.undo "reset --soft HEAD~1"
```

Other high-value habits:

```bash
$ git commit --fixup <sha>            # marks a commit as "fixing" an earlier one
$ git rebase -i --autosquash main     # auto-arranges fixups during interactive rebase
$ git cherry-pick <sha>               # apply one specific commit onto current branch
$ git blame -L 10,20 file.py          # who last touched lines 10-20
$ git shortlog -sn                    # contributor commit counts
$ git log --pretty=format:"%h %an %s" # custom log formatting
```

**Quick decision guide:**

```
Need to switch context but not commit?  -> git stash
Made a typo in the last commit msg?     -> git commit --amend
Committed but not pushed, want it gone? -> git reset --soft/--hard HEAD~1
Already pushed, need to undo publicly?  -> git revert <sha>
Want one commit from another branch?    -> git cherry-pick <sha>
Lost a commit / branch?                 -> git reflog  (see Part 2 §22)
```

---

# PART 2 — PLUMBING & INTERNALS

## 13. The Object Model

Git is fundamentally a **content-addressable filesystem**. Everything is an
object, identified by the SHA-1 (or SHA-256, in newer repos) hash of its
content. There are four object types:

```
┌───────────┐      ┌───────────┐      ┌───────────┐      ┌───────────┐
│   commit  │ ───> │    tree   │ ───> │    blob   │      │    tag    │
│ (snapshot │      │ (directory│      │  (file    │      │ (named,   │
│  metadata)│      │  listing) │      │  contents)│      │  signed   │
└───────────┘      └───────────┘      └───────────┘      │  pointer) │
      │                   │                               └───────────┘
      v                   v
  parent commit(s)    more trees/blobs (recursive)
```

A commit object, in plain text, looks like:

```
tree 8f2e5a1b3c...
parent 4e9a002d7f...
author Ada Lovelace <ada@example.com> 1699999999 +0000
committer Ada Lovelace <ada@example.com> 1699999999 +0000

Add password hashing
```

A tree object (like a directory listing):

```
100644 blob a1c4f09...   README.md
100644 blob 7b2e0d1...   auth.py
040000 tree 9c1d2e3...   src/
```

Everything is addressed by hash — **change one byte anywhere, and the hash of
that object, and every object pointing to it, changes.** This is what makes
Git tamper-evident.

```
   commit C2 ──tree──> T2 ──> auth.py (blob B2, "new content")
      │
    parent
      v
   commit C1 ──tree──> T1 ──> auth.py (blob B1, "old content")
```

---

## 14. Inside .git/

```bash
$ tree -L 2 .git
```
```
.git/
├── HEAD                 # symbolic ref: which branch you're on
├── config               # repo-local config
├── description          # unused except by gitweb
├── index                # the staging area (binary file)
├── objects/              # ALL objects live here, content-addressed
│   ├── 4e/               # first 2 hex chars = subfolder
│   │   └── 9a002d7f...   # remaining 38 chars = filename
│   └── pack/             # compacted objects (see §20)
├── refs/
│   ├── heads/            # local branches
│   ├── remotes/          # remote-tracking branches
│   └── tags/             # tags
├── logs/                 # reflogs (see §22)
└── hooks/                # local hook scripts (see §26)
```

```bash
$ git cat-file -t <hash>   # show object type
$ git cat-file -p <hash>   # pretty-print object contents
$ git hash-object file.py  # compute the hash Git WOULD use, without storing it
$ git hash-object -w file.py  # compute AND store it as a blob
```

---

## 15. Refs, HEAD & the Ref Namespace

A "ref" is just a text file (or packed line) containing a commit hash.

```bash
$ cat .git/refs/heads/main
4e9a002d7f83c1b...

$ cat .git/HEAD
ref: refs/heads/main        # HEAD normally points to a branch...

$ git checkout 4e9a002      # ...but can point directly at a commit:
$ cat .git/HEAD
4e9a002d7f83c1b...           # this is "detached HEAD" state
```

```
   refs/heads/main  ────────────┐
   refs/heads/feature ──────┐   │
   refs/tags/v1.0.0 ────┐   │   │
   refs/remotes/origin/main─┤   │
                          v   v   v
                        [commit graph in objects/]
```

```bash
$ git symbolic-ref HEAD                    # what branch does HEAD point to
$ git update-ref refs/heads/main <sha>     # move a branch pointer directly (plumbing!)
$ git show-ref                             # list every ref and its hash
$ git rev-parse HEAD                       # resolve a ref/name to a raw hash
$ git rev-parse --abbrev-ref HEAD          # -> "main" (current branch name)
```

---

## 16. Plumbing Commands Tour

Porcelain commands (`commit`, `merge`, `pull`...) are convenience wrappers
around these lower-level building blocks:

```bash
$ git hash-object -w file.py                  # store a blob, print its hash
$ git update-index --add --cacheinfo 100644 <blob-hash> file.py
                                               # manually add an entry to the index
$ git write-tree                              # write index contents as a tree object
$ git commit-tree <tree-hash> -p <parent> -m "msg"
                                               # manually create a commit object
$ git update-ref refs/heads/main <commit-hash># point a branch at that commit
```

Reconstructing `git commit` entirely by hand:

```bash
$ echo "hello" > file.txt
$ git hash-object -w file.txt
ce013625030ba8dba906f756967f9e9ca394464
$ git update-index --add --cacheinfo 100644 ce01362... file.txt
$ TREE=$(git write-tree)
$ COMMIT=$(git commit-tree $TREE -m "manual commit")
$ git update-ref refs/heads/main $COMMIT
```

Other essential plumbing:

```bash
$ git ls-tree HEAD                     # list a tree's contents
$ git ls-files                         # list files tracked in the index
$ git ls-files -s                      # with mode/hash/stage number
$ git rev-list HEAD                    # list all reachable commit hashes
$ git rev-list --count HEAD            # -> total commit count
$ git cat-file --batch-check --batch-all-objects   # inspect every object in the repo
$ git fsck                             # verify object database integrity
$ git count-objects -v                 # loose vs packed object stats
$ git mktree                           # build a tree object from stdin
$ git for-each-ref                     # iterate refs programmatically
```

---

## 17. Index (Staging Area) Internals

The index is a binary file mapping paths to blob hashes + metadata (mode,
size, mtime, stage number). Stage numbers matter during a conflicted merge:

```
Stage 0: normal, unconflicted entry
Stage 1: the common ancestor version   ("base")
Stage 2: "ours"  (current branch / HEAD)
Stage 3: "theirs" (branch being merged in)
```

```bash
$ git ls-files -u                     # show unmerged (conflicted) stage entries
$ git checkout --ours  file.py        # take our version of a conflicted file
$ git checkout --theirs file.py       # take their version
$ git diff --ours                     # (via stage numbers under the hood)
```

---

## 18. Merges: Strategies & Three-Way Merge

Git finds the **merge base** (nearest common ancestor) and does a 3-way diff:

```
              Base (O)
             /        \
        Ours (A)     Theirs (B)

   diff(O, A) = "our changes"
   diff(O, B) = "their changes"
   If both touch the same lines differently -> conflict
   If only one side touches a region -> auto-merged cleanly
```

```bash
$ git merge-base main feature          # find the common ancestor commit
$ git merge -s recursive feature       # default strategy for 2 heads
$ git merge -s ours feature            # keep ours entirely, ignore their changes
$ git merge -X ignore-space-change feature   # merge strategy OPTION
$ git merge --abort                    # bail out of a conflicted merge
$ git mergetool                        # launch a configured visual merge tool
```

---

## 19. Rebase Internals

Rebase **replays** commits onto a new base one at a time (cherry-picking each
in sequence), rather than joining histories with a merge commit:

```
Before:
main:     A─B─C
                 \
feature:          D─E

git rebase main   (while on feature)

After:
main:     A─B─C
                 \
feature:          D'─E'      (D and E rewritten with new hashes/parents!)
```

```bash
$ git rebase main                    # replay current branch onto main
$ git rebase -i HEAD~4               # interactive: reorder/squash/edit/drop commits
$ git rebase --onto main feature~3 feature   # move a whole subrange onto a new base
$ git rebase --continue              # after resolving a conflict mid-rebase
$ git rebase --abort                 # bail out, restore original state
```

Interactive rebase todo list:

```
pick   a1c4f09 add login form
squash 7b2e0d1 fix typo
reword 9f3a1c4 add password hashing
drop   c3d8e77 debug print (oops)
```

⚠ **Golden rule of rebase:** never rebase commits that others have already
pulled — it rewrites hashes and diverges shared history.

---

## 20. Packfiles, Compression & gc

Loose objects (one file per object) are inefficient at scale. Git periodically
compacts them into **packfiles**: a single compressed file of deduplicated
objects, plus an index for fast lookup by hash.

```
objects/           objects/pack/
├── 4e/9a002...     ├── pack-3f2a....pack   (all objects, delta-compressed)
├── 7b/2e0d1...  -> └── pack-3f2a....idx    (hash -> offset lookup table)
├── a1/c4f09...
└── ...
```

```bash
$ git gc                          # garbage collect: pack loose objects, prune unreachable
$ git gc --aggressive             # more thorough repacking (slower, smaller result)
$ git repack -a -d                # repack everything into one pack, delete redundant packs
$ git verify-pack -v .git/objects/pack/pack-xxx.idx   # inspect a packfile's contents
$ git prune                       # remove unreachable loose objects (careful!)
```

Delta compression: instead of storing full copies of similar blobs, Git stores
one full version and small diffs ("deltas") against it — this is why cloning
a repo with huge binary churn stays reasonably compact only when text-like.

---

## 21. Transfer Protocols: fetch/push under the hood

```
   Client                              Server
     │  1. ref advertisement            │
     │ <──────────────────────────────  │  "here are my refs + hashes"
     │  2. "want" list (what I need)    │
     │ ────────────────────────────────>│
     │  3. packfile negotiation          │
     │    (server computes minimal set   │
     │     of objects client is missing) │
     │ <──────────────────────────────  │
     │  4. packfile transferred          │
     │ <──────────────────────────────  │
     │  5. client updates local refs     │
```

```bash
$ git ls-remote <url>                 # see remote refs WITHOUT cloning
$ GIT_TRACE_PACKET=1 git fetch        # debug the wire protocol
$ git fetch origin +refs/heads/*:refs/remotes/origin/*   # explicit refspec
$ git push origin HEAD:refs/heads/main                   # explicit refspec push
$ git bundle create repo.bundle --all # package a whole repo into ONE file (offline transfer!)
$ git clone repo.bundle                # clone from that bundle file
```

Refspec syntax: `<src>:<dst>` — `+` prefix means "force update even if not a
fast-forward" (e.g. `+refs/heads/*:refs/remotes/origin/*`).

---

## 22. Reflog & Data Recovery

The reflog is Git's local "undo history" of where `HEAD` and branches have
pointed — your safety net, even after `reset --hard` or a bad rebase.

```bash
$ git reflog
9f3a1c4 HEAD@{0}: commit: add password hashing
4c2a9f1 HEAD@{1}: reset: moving to HEAD~1
7b2e0d1 HEAD@{2}: commit: add login form

$ git reset --hard HEAD@{1}          # travel back to any reflog entry
$ git checkout HEAD@{2}              # or just look at it in detached HEAD
```

**Recovering a "deleted" commit** (nothing is truly gone until `gc` prunes it):

```bash
$ git branch feature                 # oh no, wrong branch deleted
$ git reflog                         # find the last hash feature pointed to
$ git branch feature <recovered-sha> # recreate the branch there
```

```bash
$ git fsck --lost-found              # find unreachable/dangling commits directly
```

Reflog entries expire (default 90 days for reachable, 30 for unreachable) —
configurable via `gc.reflogExpire`.

---

## 23. Rewriting History

```bash
$ git commit --amend                              # rewrite the tip commit
$ git rebase -i <base>                             # rewrite a range interactively
$ git filter-branch --tree-filter 'rm -f secret.txt' HEAD   # legacy, slow, avoid if possible
$ git filter-repo --path secret.txt --invert-paths # modern replacement (separate tool)
```

⚠ Any of these change commit hashes downstream. After rewriting *shared*
history you must coordinate a force-push and have collaborators re-sync:

```bash
$ git push --force-with-lease origin main   # safer than --force:
                                             # fails if remote has NEW commits
                                             # you don't know about yet
```

---

## 24. Bisect, Blame, Grep

**Bisect** — binary search through history to find the commit that introduced a bug:

```bash
$ git bisect start
$ git bisect bad                 # current commit is broken
$ git bisect good v1.0.0         # this old tag was fine
                                  # Git checks out the midpoint commit...
$ git bisect good                # ...you test it, tell git the result...
$ git bisect bad                 # ...repeat until Git narrows it to ONE commit
$ git bisect reset               # done, return to original HEAD
```

```
good ─────────────?─────────────?─────────────bad
v1.0.0                       (bisect jumps here)                     HEAD
```

Automate it with a script that exits 0 (good) or 1 (bad):
```bash
$ git bisect run ./run_tests.sh
```

**Blame** and **grep**:

```bash
$ git blame -L 40,60 -- src/auth.py    # who wrote lines 40-60, and in what commit
$ git blame -w file.py                 # ignore whitespace-only changes
$ git grep "TODO" $(git rev-list --all)   # search TODO across every commit ever
$ git log -G"regex_pattern"            # find commits whose DIFF matches a regex
```

---

## 25. Worktrees & Submodules

**Worktrees** — check out multiple branches simultaneously in separate folders,
sharing one `.git` object database:

```bash
$ git worktree add ../hotfix hotfix-branch
$ git worktree list
/home/ada/project        4e9a002 [main]
/home/ada/hotfix         7b2e0d1 [hotfix-branch]
$ git worktree remove ../hotfix
```

**Submodules** — embed one repo inside another, pinned to a specific commit:

```bash
$ git submodule add <url> libs/vendor
$ git clone --recurse-submodules <url>       # clone including submodule content
$ git submodule update --init --recursive    # populate submodules after a plain clone
$ git submodule update --remote              # pull latest commit for a submodule
```

```
parent-repo/
├── .gitmodules            # maps submodule paths -> URLs
├── src/
└── libs/vendor/  ──────>  (pinned to commit abc123 of the submodule's own history)
```

---

## 26. Hooks

Local scripts in `.git/hooks/` that fire on Git events (not version-controlled
by default — that's what tools like Husky solve for teams):

```
.git/hooks/
├── pre-commit       # runs before a commit is created (e.g. lint, tests)
├── commit-msg       # validate/edit the commit message
├── pre-push         # runs before pushing (e.g. block push to main)
├── post-checkout    # runs after switching branches
└── post-merge       # runs after a merge completes
```

```bash
#!/bin/sh
# .git/hooks/pre-commit  (must be executable: chmod +x)
npm run lint || exit 1
```

---

## 27. Sparse Checkout & Partial Clone

For huge monorepos, avoid downloading/checking-out everything:

```bash
$ git clone --filter=blob:none <url>       # partial clone: fetch commits/trees now,
                                            # blobs on-demand as needed
$ git sparse-checkout init --cone
$ git sparse-checkout set src/frontend src/shared
                                            # only these dirs appear in working dir,
                                            # full history still available
```

```
Full repo objects:   [ commits ][ trees ][ ALL blobs ]
Partial clone:       [ commits ][ trees ][ blobs fetched lazily on checkout/diff ]
Sparse checkout:     working dir shows only src/frontend/ and src/shared/
```

---

## 28. Command Cheat Sheet by Use Case

```
SETUP               config, init, clone
DAILY LOOP          status, add, restore, commit, diff, log, show
BRANCHING           branch, switch, checkout, merge, rebase
REMOTES             remote, fetch, pull, push, ls-remote
UNDO / SAFETY NET   restore, reset, revert, reflog, fsck
STASH & TAGS        stash, tag
HISTORY SEARCH      log (-S/-G/--author/--since), blame, grep, bisect
CLEANUP             clean, gc, prune, repack
MULTI-CONTEXT       worktree, submodule, sparse-checkout
LOW-LEVEL / PLUMBING hash-object, cat-file, update-index, write-tree,
                    commit-tree, update-ref, rev-parse, rev-list,
                    ls-tree, ls-files, show-ref, symbolic-ref,
                    for-each-ref, mktree, verify-pack, count-objects
REWRITE HISTORY     commit --amend, rebase -i, filter-repo, push --force-with-lease
AUTOMATION          hooks (pre-commit, pre-push, ...), bisect run
```

---

### Suggested study path
1. Live in Part 1 daily until `add/commit/branch/merge/push/pull` are muscle memory.
2. Practice the "undo" table (§7) until reset vs. revert vs. restore is automatic.
3. Once comfortable, rebuild a commit by hand using §16's plumbing walkthrough —
   it's the fastest way to *really* understand what `git commit` does.
4. Learn `reflog` (§22) before you ever need it in a panic.
5. Explore rebase, bisect, and worktrees once the basics are second nature.
