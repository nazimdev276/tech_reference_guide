# The Complete Neovim Reference Guide

A single-file, cross-referenced guide to using and configuring Neovim: modes, motions, configuration structure, keymaps, plugin management, LSP, treesitter, debugging, and workflow tips. Every major section has a short ID in brackets (e.g. `[CFG-1]`) so you can jump-search this file (`Ctrl-F` / `/` in a reader) or link to a section from your own notes.

---

## Table of Contents

1. [Core Concepts](#1-core-concepts-core)
2. [Installation & First Run](#2-installation--first-run-install)
3. [Configuration Structure](#3-configuration-structure-cfg)
4. [Options (`vim.opt`)](#4-options-vimopt-opt)
5. [Keymaps](#5-keymaps-map)
6. [Autocommands](#6-autocommands-auto)
7. [Modes & Built-in Motions Cheat Sheet](#7-modes--built-in-motions-cheat-sheet-motion)
8. [Windows, Buffers, Tabs](#8-windows-buffers-tabs-win)
9. [Plugin Managers](#9-plugin-managers-pm)
10. [Essential Plugin Categories](#10-essential-plugin-categories-plug)
    - 10.1 [Fuzzy Finder — Telescope](#101-fuzzy-finder--telescope-plug-telescope)
    - 10.2 [File Explorer](#102-file-explorer-plug-explorer)
    - 10.3 [Treesitter (syntax)](#103-treesitter-syntax-plug-ts)
    - 10.4 [LSP (Language Servers)](#104-lsp-language-servers-plug-lsp)
    - 10.5 [Autocompletion](#105-autocompletion-plug-cmp)
    - 10.6 [Formatting & Linting](#106-formatting--linting-plug-fmt)
    - 10.7 [Git Integration](#107-git-integration-plug-git)
    - 10.8 [Statusline & UI](#108-statusline--ui-plug-ui)
    - 10.9 [Debugging (DAP)](#109-debugging-dap-plug-dap)
    - 10.10 [Snippets](#1010-snippets-plug-snip)
11. [LSP Deep Dive](#11-lsp-deep-dive-lsp)
12. [Sample Full `init.lua`](#12-sample-full-initlua-sample)
13. [Workflow Tips & Troubleshooting](#13-workflow-tips--troubleshooting-tips)
14. [Quick Reference Cheat Sheet](#14-quick-reference-cheat-sheet-cheat)

---

## 1. Core Concepts `[CORE]`

Neovim is a fork of Vim built for extensibility. Key ideas to internalize before configuring anything:

- **`[CORE-1]` Modal editing** — Neovim has distinct modes (Normal, Insert, Visual, Command, Replace, Terminal). Commands mean different things in different modes. Almost every config problem traces back to "which mode was I in?"
- **`[CORE-2]` Everything is scriptable** — Neovim embeds a Lua runtime (`vim.lua` API) alongside legacy Vimscript. New configs should be written in **Lua**, not Vimscript, unless you're maintaining an old config.
- **`[CORE-3]` `vim` global table** — Inside Lua config files, the `vim` table is your entry point to everything: `vim.opt`, `vim.keymap`, `vim.api`, `vim.lsp`, `vim.fn`, `vim.cmd`, `vim.g`, `vim.diagnostic`.
- **`[CORE-4]` No plugins by default** — A stock Neovim is close to stock Vim. All the IDE-like behavior (autocomplete, LSP UI, fuzzy finding, file trees) comes from plugins you install.
- **`[CORE-5]` Leader key** — A user-defined "namespace" key for custom shortcuts, so you don't clash with built-ins. Commonly set to Space or `,`.

```lua
vim.g.mapleader = " "
vim.g.maplocalleader = " "
```
Must be set **before** any plugin manager/plugin loads, since plugins register leader-based keymaps at load time.

---

## 2. Installation & First Run `[INSTALL]`

- **`[INSTALL-1]` Install Neovim** (≥0.9 recommended, ≥0.10 for the newest LSP/plugin APIs):
  - macOS: `brew install neovim`
  - Ubuntu/Debian: `sudo apt install neovim` (or the PPA/AppImage for a newer version)
  - Arch: `sudo pacman -S neovim`
  - Windows: `winget install Neovim.Neovim` or via Scoop/Chocolatey
- **`[INSTALL-2]` Verify**: `nvim --version`
- **`[INSTALL-3]` Config location**:
  - Linux/macOS: `~/.config/nvim/`
  - Windows: `~/AppData/Local/nvim/`
  - Override with `$XDG_CONFIG_HOME`
- **`[INSTALL-4]` Data/state locations** (plugin installs, shada file, swap files):
  - Linux/macOS: `~/.local/share/nvim/`, `~/.local/state/nvim/`
  - Windows: `~/AppData/Local/nvim-data/`
- **`[INSTALL-5]` Health check** — run `:checkhealth` inside Neovim after any config change or plugin install. It reports missing dependencies (Python provider, Node provider, compilers, clipboard tool, etc.) before they cause confusing bugs.
- **`[INSTALL-6]` Clipboard** — for `"+y` / `"+p` (system clipboard) to work you need `xclip`/`xsel` (Linux X11), `wl-clipboard` (Wayland), or it's automatic on macOS/Windows.

---

## 3. Configuration Structure `[CFG]`

- **`[CFG-1]` Entry point**: `init.lua` (or legacy `init.vim`) at the root of your config directory. Neovim reads this file first, once, at startup.
- **`[CFG-2]` Modular layout** — Don't put everything in one file. Standard community pattern:

```
~/.config/nvim/
├── init.lua                  -- requires everything below
├── lua/
│   └── user/                 -- (or your username / "config")
│       ├── options.lua       -- vim.opt settings
│       ├── keymaps.lua       -- global keymaps
│       ├── autocmds.lua      -- autocommands
│       └── lazy.lua          -- plugin manager bootstrap + spec
└── lua/
    └── plugins/              -- one file per plugin, each returns a lazy.nvim spec table
        ├── telescope.lua
        ├── treesitter.lua
        ├── lsp.lua
        └── ...
```

- **`[CFG-3]` Why `lua/` matters** — Anything under `lua/<name>/` is `require()`-able as `<name>` or `<name>.submodule`, mirroring Lua's module system. `init.lua` typically just does:

```lua
require("user.options")
require("user.keymaps")
require("user.autocmds")
require("user.lazy")   -- bootstraps plugin manager, which lazy-loads lua/plugins/*.lua
```

- **`[CFG-4]` Per-plugin files** — With `lazy.nvim`, each file in `lua/plugins/` returns a table (a "plugin spec") describing one plugin: source, dependencies, config function, lazy-load triggers. This is the modern idiom — keeps config for each plugin colocated and independently lazy-loadable.
- **`[CFG-5]` Version control your config** — Init the config dir as a git repo (`~/.config/nvim` → GitHub). This is standard practice; most "dotfiles" repos you see online are exactly this.
- **`[CFG-6]` `:source %`** — reload the current config file without restarting Neovim, useful while iterating (full reload of everything still generally needs a restart, since some plugin setup only runs once).

---

## 4. Options (`vim.opt`) `[OPT]`

Options configure editor *behavior* (not keybinds, not plugins). Set with `vim.opt.<name> = value`, or `vim.o` (raw string form), or `vim.wo`/`vim.bo` for window-/buffer-local options.

**`[OPT-1]` Commonly set options:**

```lua
local opt = vim.opt

opt.number = true              -- show line numbers
opt.relativenumber = true      -- relative numbers (great for motions like 5j)
opt.tabstop = 2                -- width of a tab character
opt.shiftwidth = 2             -- width used for auto-indent
opt.expandtab = true           -- convert tabs to spaces
opt.smartindent = true
opt.wrap = false               -- don't soft-wrap long lines
opt.ignorecase = true          -- case-insensitive search...
opt.smartcase = true           -- ...unless you type an uppercase letter
opt.cursorline = true          -- highlight current line
opt.termguicolors = true       -- true color support (needed by most colorschemes)
opt.scrolloff = 8              -- keep 8 lines visible above/below cursor
opt.signcolumn = "yes"         -- always show sign column (avoids text shifting for LSP icons)
opt.updatetime = 250           -- faster CursorHold events (used by LSP hover, gitsigns, etc.)
opt.clipboard = "unnamedplus"  -- use system clipboard for yank/paste
opt.splitright = true          -- vertical splits open to the right
opt.splitbelow = true          -- horizontal splits open below
opt.undofile = true            -- persistent undo across sessions
opt.swapfile = false
```

- **`[OPT-2]` Discovering options** — `:help option-list`, or `:set <name>?` to check current value, `:help '<name>'` for docs on a specific one.
- **`[OPT-3]` Scopes** — `vim.opt` sets global+local sensibly for most options; `vim.bo.<name>` = buffer-local only, `vim.wo.<name>` = window-local only (e.g. `vim.wo.wrap = true` for just the current window).

---

## 5. Keymaps `[MAP]`

- **`[MAP-1]` API**: `vim.keymap.set(mode, lhs, rhs, opts)`
  - `mode`: string or table — `"n"` normal, `"i"` insert, `"v"` visual, `"x"` visual-only, `"t"` terminal, `"c"` command-line, or `{"n","v"}` for multiple.
  - `lhs`: the key sequence typed, e.g. `"<leader>ff"`.
  - `rhs`: a string (Ex command / keys) or a Lua function.
  - `opts`: table — `desc` (shows in which-key/telescope keymap picker), `silent`, `noremap` (default true), `buffer` (buffer-local map).

```lua
local map = vim.keymap.set

map("n", "<leader>w", ":w<CR>", { desc = "Save file" })
map("n", "<leader>q", ":q<CR>", { desc = "Quit" })
map("i", "jk", "<Esc>", { desc = "Exit insert mode" })
map("v", "J", ":m '>+1<CR>gv=gv", { desc = "Move selection down" })
map("v", "K", ":m '<-2<CR>gv=gv", { desc = "Move selection up" })
map("n", "<C-d>", "<C-d>zz", { desc = "Half-page down, keep centered" })
map("n", "<C-u>", "<C-u>zz", { desc = "Half-page up, keep centered" })
map("n", "n", "nzzzv", { desc = "Next search result, centered" })
```

- **`[MAP-2]` Buffer-local keymaps** — set inside an `LspAttach` autocommand or `ftplugin` so they only apply where relevant (e.g. `gd` for go-to-definition only in buffers with an active LSP client). See [LSP-4].
- **`[MAP-3]` Discover current keymaps** — `:map`, `:nmap`, `:imap` (list by mode); `:verbose map <lhs>` shows *where* a mapping was defined (essential for debugging conflicts between plugins).
- **`[MAP-4]` `which-key.nvim`** — a plugin that pops up a menu of available keymaps after you press a prefix (like `<leader>`) and pause. Effectively turns your keymap config into self-documenting UI. Extremely commonly paired with a leader-key-heavy setup.
- **`[MAP-5]` Naming convention people use** — group leader-prefixed keys by purpose: `<leader>f*` = find/file, `<leader>g*` = git, `<leader>l*` = LSP, `<leader>b*` = buffer, `<leader>c*` = code actions. Keeps a large keymap set navigable.

---

## 6. Autocommands `[AUTO]`

Autocommands run code automatically on editor **events** (file opened, saved, filetype detected, etc.). Modern Lua API:

```lua
local augroup = vim.api.nvim_create_augroup("UserConfig", { clear = true })

vim.api.nvim_create_autocmd("TextYankPost", {
  group = augroup,
  desc = "Highlight yanked text",
  callback = function()
    vim.highlight.on_yank({ timeout = 200 })
  end,
})

vim.api.nvim_create_autocmd("BufWritePre", {
  group = augroup,
  pattern = "*.lua",
  desc = "Trim trailing whitespace on save",
  command = [[%s/\s\+$//e]],
})
```

- **`[AUTO-1]` Groups** — always attach autocommands to a named group with `clear = true`. Without this, reloading your config re-registers the same autocommand repeatedly (duplicated behavior after every `:source`).
- **`[AUTO-2]` Common events**: `BufEnter`, `BufWritePre`/`BufWritePost`, `VimEnter`, `FileType`, `InsertEnter`/`InsertLeave`, `LspAttach`, `CursorHold`, `TextYankPost`.
- **`[AUTO-3]` `:help autocmd-events`** for the full list.

---

## 7. Modes & Built-in Motions Cheat Sheet `[MOTION]`

- **`[MOTION-1]` Modes**: Normal (`<Esc>`), Insert (`i`,`a`,`o`,`I`,`A`,`O`), Visual (`v`,`V`,`Ctrl-v`), Command (`:`), Replace (`R`), Terminal (`i` inside `:terminal`, `<C-\><C-n>` to exit to Normal).
- **`[MOTION-2]` Movement**: `h j k l` (left/down/up/right), `w`/`b`/`e` (word forward/back/end), `0`/`^`/`$` (line start/first-non-blank/end), `gg`/`G` (top/bottom of file), `{`/`}` (paragraph), `%` (matching bracket), `f<char>`/`t<char>` (find/till char on line), `Ctrl-d`/`Ctrl-u` (half-page).
- **`[MOTION-3]` Editing**: `x` delete char, `dd` delete line, `yy` yank line, `p`/`P` paste after/before, `cc` change line, `cw` change word, `u` undo, `Ctrl-r` redo, `.` repeat last change.
- **`[MOTION-4]` Text objects** (combine with `d`/`c`/`y`/`v`): `iw`/`aw` (inner/a word), `i"`/`a"` (inside/around quotes), `ib`/`ab` or `i(`/`a(` (inside/around parens), `it`/`at` (inside/around tag). Example: `di"` deletes inside quotes, `ci(` changes inside parens.
- **`[MOTION-5]` Search/replace**: `/pattern` search forward, `?pattern` search back, `n`/`N` next/prev match, `:%s/old/new/g` substitute in whole file, `:%s/old/new/gc` with confirmation.
- **`[MOTION-6]` Marks & jumps**: `ma` set mark "a", `` `a `` jump to it, `Ctrl-o`/`Ctrl-i` jump back/forward in jumplist.
- **`[MOTION-7]` Registers**: `"ayy` yank line into register a, `"ap` paste from register a, `:reg` to list registers.
- **`[MOTION-8]` Macros**: `qa` start recording into register a, `q` stop, `@a` play macro, `@@` repeat last macro, `5@a` run 5 times.

Full reference: `:help motion.txt`, `:help text-objects`.

---

## 8. Windows, Buffers, Tabs `[WIN]`

These three are frequently confused:

- **`[WIN-1]` Buffer** = an in-memory representation of a file's content. Can exist without being displayed.
- **`[WIN-2]` Window** = a viewport into a buffer. `:split`/`Ctrl-w s` (horizontal), `:vsplit`/`Ctrl-w v` (vertical). Navigate with `Ctrl-w h/j/k/l`, close with `Ctrl-w q` or `:q`.
- **`[WIN-3]` Tab** = a collection of windows (a "workspace"), *not* like a browser tab holding one file. `:tabnew`, `gt`/`gT` to cycle.
- **`[WIN-4]` Buffer commands**: `:ls`/`:buffers` (list), `:b <name/number>` (switch), `:bn`/`:bp` (next/prev), `:bd` (delete buffer). Most people bind `<leader>bn`/`<leader>bp` and a fuzzy buffer-picker (Telescope) rather than memorizing numbers.

---

## 9. Plugin Managers `[PM]`

- **`[PM-1]` lazy.nvim** *(current de-facto standard, recommended for new configs)* — written in Lua, supports true lazy-loading (load a plugin only on a keymap/command/filetype/event), lockfile for reproducibility, built-in UI (`:Lazy`) and profiler (`:Lazy profile`).

Bootstrap snippet (goes in e.g. `lua/user/lazy.lua`):

```lua
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git", "clone", "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require("lazy").setup("plugins")  -- loads every spec file under lua/plugins/
```

A plugin spec looks like:

```lua
-- lua/plugins/telescope.lua
return {
  "nvim-telescope/telescope.nvim",
  tag = "0.1.8",
  dependencies = { "nvim-lua/plenary.nvim" },
  cmd = "Telescope",              -- lazy-load: only load when :Telescope is run
  keys = {
    { "<leader>ff", "<cmd>Telescope find_files<cr>", desc = "Find files" },
  },
  config = function()
    require("telescope").setup({})
  end,
}
```

- **`[PM-2]` Key lazy.nvim commands**: `:Lazy` (UI: install/update/clean), `:Lazy sync`, `:Lazy profile` (startup time breakdown per plugin).
- **`[PM-3]` Alternatives**:
  - **packer.nvim** — older Lua manager, now largely unmaintained; many existing configs still use it.
  - **vim-plug** — Vimscript-based, simple, still widely used especially in Vimscript-heavy configs.
  - **mini.deps** — minimal manager, part of the `mini.nvim` plugin suite.
  - **Native `vim.pack`** (Neovim 0.12+) — built-in package management with no external plugin manager required, for people who want zero extra dependencies.
- **`[PM-4]` "Distributions"** — Pre-built configs like **LazyVim**, **AstroNvim**, **NvChad**, and **kickstart.nvim** package a plugin manager + a curated plugin set + sane defaults. `kickstart.nvim` in particular is a single, heavily-commented `init.lua` meant to be read and modified rather than treated as a black box — a good starting point for learning how the pieces fit together before building a fully custom config.

---

## 10. Essential Plugin Categories `[PLUG]`

### 10.1 Fuzzy Finder — Telescope `[PLUG-TELESCOPE]`
`nvim-telescope/telescope.nvim` — fuzzy-find files, live grep text across the project, list buffers, keymaps, git commits, LSP symbols, and more, all through one extensible UI.
- Typical keymaps: `<leader>ff` find_files, `<leader>fg` live_grep, `<leader>fb` buffers, `<leader>fh` help_tags, `<leader>fd` diagnostics.
- Requires `ripgrep` (`rg`) for live_grep and `fd` for faster file finding.
- Alternative: **fzf-lua** (wraps `fzf`, faster on huge repos), **snacks.nvim** picker module.

### 10.2 File Explorer `[PLUG-EXPLORER]`
- **nvim-tree.lua** or **neo-tree.nvim** — sidebar file tree, create/rename/delete files, git status icons.
- Typical keymap: `<leader>e` toggle explorer.
- Many users increasingly prefer Telescope's `find_files`/`oil.nvim` (edits the filesystem *as a buffer*, using normal Vim editing to rename/delete files) over a persistent tree sidebar.

### 10.3 Treesitter (syntax) `[PLUG-TS]`
`nvim-treesitter/nvim-treesitter` — replaces regex-based syntax highlighting with real incremental parsing, giving accurate highlighting, indentation, and structural text objects (e.g. select a whole function body).

```lua
require("nvim-treesitter.configs").setup({
  ensure_installed = { "lua", "python", "javascript", "rust", "go" },
  highlight = { enable = true },
  indent = { enable = true },
})
```
- `:TSInstall <lang>` to add a parser manually.
- `:TSUpdate` after upgrades.

### 10.4 LSP (Language Servers) `[PLUG-LSP]`
Covered in full in [Section 11](#11-lsp-deep-dive-lsp). Core pieces:
- **nvim-lspconfig** — ready-made configs for hundreds of language servers.
- **mason.nvim** — installs/manages language servers, linters, formatters, DAP adapters from inside Neovim (`:Mason` UI).
- **mason-lspconfig.nvim** — glue between the two above.

### 10.5 Autocompletion `[PLUG-CMP]`
- **nvim-cmp** — the long-standing completion engine, pulls suggestions from sources (LSP, buffer words, snippets, paths) via source plugins (`cmp-nvim-lsp`, `cmp-buffer`, `cmp-path`).
- **blink.cmp** — newer, faster (Rust-backed fuzzy matcher), increasingly the default in new configs/distributions.
- Typical keymaps: `<Tab>`/`<C-n>` next item, `<S-Tab>`/`<C-p>` previous, `<CR>` confirm, `<C-Space>` manually trigger.

### 10.6 Formatting & Linting `[PLUG-FMT]`
- **conform.nvim** — runs formatters (prettier, stylua, black, gofmt, ...) on save or on demand, per filetype.
- **nvim-lint** — runs linters (eslint, ruff, ...) asynchronously and shows results as diagnostics.
- (Some LSP servers also format directly via `vim.lsp.buf.format()` — conform.nvim is preferred when you need a different formatter than your LSP server, or need to chain several.)

### 10.7 Git Integration `[PLUG-GIT]`
- **gitsigns.nvim** — gutter signs for added/changed/removed lines, inline blame, stage/reset hunks (`<leader>hs`/`<leader>hr`), `:Gitsigns preview_hunk`.
- **vim-fugitive** — full git porcelain inside Neovim (`:G`, `:Git blame`, `:Gdiffsplit`, interactive rebase).
- **diffview.nvim** — rich side-by-side diff and merge-conflict UI.
- **lazygit** (external TUI, not a plugin) is often launched from inside Neovim via a terminal keymap for full git operations.

### 10.8 Statusline & UI `[PLUG-UI]`
- **lualine.nvim** — configurable statusline (mode, filename, git branch, diagnostics count).
- **bufferline.nvim** — tab-style buffer bar along the top.
- **noice.nvim** — replaces the command line and messages with a nicer floating UI.
- **which-key.nvim** — see [MAP-4].
- Colorschemes: **tokyonight.nvim**, **catppuccin**, **gruvbox.nvim**, **kanagawa.nvim** are among the most widely used.

### 10.9 Debugging (DAP) `[PLUG-DAP]`
- **nvim-dap** — implements the Debug Adapter Protocol, giving breakpoints, step-in/over/out, variable inspection, from inside Neovim.
- **nvim-dap-ui** — a UI layer (variable panes, breakpoint list, REPL) on top of nvim-dap.
- **mason-nvim-dap.nvim** — installs debug adapters (debugpy, delve, codelldb, ...) via Mason.
- Typical keymaps: `<F5>` continue, `<F10>` step over, `<F11>` step into, `<leader>b` toggle breakpoint.

### 10.10 Snippets `[PLUG-SNIP]`
- **LuaSnip** — the standard snippet engine; integrates with nvim-cmp/blink.cmp so snippets appear as completion items.
- **friendly-snippets** — a large community-maintained snippet collection you load into LuaSnip rather than writing every snippet by hand.

---

## 11. LSP Deep Dive `[LSP]`

LSP (Language Server Protocol) is what gives Neovim IDE features: diagnostics, go-to-definition, hover docs, rename-across-files, code actions. Neovim ships a **built-in LSP client** (`vim.lsp`); plugins just make it easier to configure and to install the servers themselves.

- **`[LSP-1]` The pieces and how they relate**:
  1. A **language server** is a separate program (e.g. `pyright` for Python, `tsserver`/`vtsls` for TS/JS, `rust-analyzer` for Rust, `gopls` for Go, `lua_ls` for Lua) that speaks the LSP protocol.
  2. **mason.nvim** installs that binary for you (`:Mason`, search, press `i`).
  3. **nvim-lspconfig** provides the default launch command/settings for each server so you don't hand-write it.
  4. Neovim's built-in `vim.lsp.start()` (wrapped by lspconfig) actually connects to it.

- **`[LSP-2]` Minimal modern setup** (Neovim ≥0.10 style):

```lua
-- lua/plugins/lsp.lua
return {
  {
    "williamboman/mason.nvim",
    config = true, -- calls require("mason").setup()
  },
  {
    "neovim/nvim-lspconfig",
    dependencies = { "mason.nvim", "williamboman/mason-lspconfig.nvim" },
    config = function()
      require("mason-lspconfig").setup({
        ensure_installed = { "lua_ls", "pyright", "tsserver" },
      })
      local lspconfig = require("lspconfig")
      lspconfig.lua_ls.setup({})
      lspconfig.pyright.setup({})
      lspconfig.tsserver.setup({})
    end,
  },
}
```

- **`[LSP-3]` Diagnostics** — errors/warnings from the server show as virtual text + signcolumn icons automatically. Navigate with:

```lua
vim.keymap.set("n", "[d", vim.diagnostic.goto_prev, { desc = "Prev diagnostic" })
vim.keymap.set("n", "]d", vim.diagnostic.goto_next, { desc = "Next diagnostic" })
vim.keymap.set("n", "<leader>e", vim.diagnostic.open_float, { desc = "Show diagnostic" })
vim.diagnostic.setqflist()  -- send all diagnostics to the quickfix list
```

- **`[LSP-4]` Buffer-local LSP keymaps via `LspAttach`** — the correct pattern is to bind LSP-specific keys only in buffers that actually have a client attached:

```lua
vim.api.nvim_create_autocmd("LspAttach", {
  group = vim.api.nvim_create_augroup("UserLspConfig", {}),
  callback = function(ev)
    local opts = { buffer = ev.buf }
    vim.keymap.set("n", "gd", vim.lsp.buf.definition, opts)
    vim.keymap.set("n", "gD", vim.lsp.buf.declaration, opts)
    vim.keymap.set("n", "gr", vim.lsp.buf.references, opts)
    vim.keymap.set("n", "gI", vim.lsp.buf.implementation, opts)
    vim.keymap.set("n", "K", vim.lsp.buf.hover, opts)
    vim.keymap.set("n", "<leader>rn", vim.lsp.buf.rename, opts)
    vim.keymap.set("n", "<leader>ca", vim.lsp.buf.code_action, opts)
    vim.keymap.set("n", "<leader>f", function() vim.lsp.buf.format({ async = true }) end, opts)
  end,
})
```

- **`[LSP-5]` Inspecting LSP state** — `:LspInfo` (attached clients for current buffer), `:Mason` (installed tools), `:checkhealth vim.lsp`.
- **`[LSP-6]` Per-project settings** — many servers read project files (`pyrightconfig.json`, `tsconfig.json`, `.luarc.json`, `rust-project.json`); Neovim-side `settings = {...}` in the `setup()` call is a fallback/override, not the only place to configure a server.

---

## 12. Sample Full `init.lua` `[SAMPLE]`

A compact, single-file starting point (not modular, but shows every category from this guide wired together). Save as `~/.config/nvim/init.lua`:

```lua
-- === Leader ===
vim.g.mapleader = " "
vim.g.maplocalleader = " "

-- === Options ===
local opt = vim.opt
opt.number = true
opt.relativenumber = true
opt.tabstop = 2
opt.shiftwidth = 2
opt.expandtab = true
opt.ignorecase = true
opt.smartcase = true
opt.termguicolors = true
opt.signcolumn = "yes"
opt.clipboard = "unnamedplus"
opt.undofile = true
opt.scrolloff = 8

-- === Keymaps ===
local map = vim.keymap.set
map("n", "<leader>w", ":w<CR>")
map("i", "jk", "<Esc>")
map("n", "<leader>e", ":Neotree toggle<CR>")

-- === Plugin manager bootstrap (lazy.nvim) ===
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git", "clone", "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git", "--branch=stable", lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require("lazy").setup({
  { "folke/tokyonight.nvim", priority = 1000, config = function() vim.cmd("colorscheme tokyonight") end },
  { "nvim-lualine/lualine.nvim", config = true },
  { "nvim-neo-tree/neo-tree.nvim", branch = "v3.x",
    dependencies = { "nvim-lua/plenary.nvim", "nvim-tree/nvim-web-devicons", "MunifTanjim/nui.nvim" } },
  { "nvim-telescope/telescope.nvim", tag = "0.1.8", dependencies = { "nvim-lua/plenary.nvim" },
    keys = { { "<leader>ff", "<cmd>Telescope find_files<cr>" }, { "<leader>fg", "<cmd>Telescope live_grep<cr>" } } },
  { "nvim-treesitter/nvim-treesitter", build = ":TSUpdate",
    config = function() require("nvim-treesitter.configs").setup({ ensure_installed = { "lua", "python" }, highlight = { enable = true } }) end },
  { "williamboman/mason.nvim", config = true },
  { "neovim/nvim-lspconfig", dependencies = { "mason.nvim" },
    config = function() require("lspconfig").lua_ls.setup({}) end },
  { "lewis6991/gitsigns.nvim", config = true },
})
```

---

## 13. Workflow Tips & Troubleshooting `[TIPS]`

- **`[TIPS-1]` `:checkhealth`** — first stop for almost any weird plugin/LSP behavior.
- **`[TIPS-2]` `:messages`** — shows recent error/echo messages you may have missed.
- **`[TIPS-3]` `:verbose map <key>`** — find out which plugin/file set a confusing keymap.
- **`[TIPS-4]` `:Lazy profile`** — find out which plugin is slowing down startup.
- **`[TIPS-5]` Read `:help` before Googling** — Neovim's built-in help (`:help <topic>`) is thorough and version-accurate; `:help lua-guide` is a good on-ramp specifically for configuring in Lua.
- **`[TIPS-6]` Start from a distribution, then eject** — if hand-rolling a config feels overwhelming, start with kickstart.nvim (single readable file) or LazyVim (batteries-included), use it for a few weeks, then gradually replace pieces you understand with your own config.
- **`[TIPS-7]` Keep plugin count intentional** — every plugin is startup time and a surface for breakage; periodically run `:Lazy clean` and remove things you don't actually use.
- **`[TIPS-8]` Version pinning** — pin plugins to a tag/commit (`lazy-lock.json`, generated automatically by lazy.nvim) and commit that lockfile so your config is reproducible across machines.
- **`[TIPS-9]` Terminal inside Neovim** — `:terminal` opens a terminal buffer; `<C-\><C-n>` drops it to Normal mode so you can navigate/copy; many configs map a quick toggle (e.g. via `toggleterm.nvim`).

---

## 14. Quick Reference Cheat Sheet `[CHEAT]`

| Category | Command / Key | Effect |
|---|---|---|
| Save / quit | `:w`, `:q`, `:wq`, `:q!` | write / quit / write+quit / force quit |
| Reload config | `:source %` | re-run current Lua/Vimscript file |
| Health check | `:checkhealth` | diagnose config/plugin/provider issues |
| Plugin manager UI | `:Lazy` | install/update/clean/profile plugins |
| Tool installer UI | `:Mason` | install LSP servers, linters, formatters, DAP adapters |
| Find files | `<leader>ff` (Telescope) | fuzzy find files in project |
| Live grep | `<leader>fg` (Telescope) | search text across project |
| Toggle file tree | `<leader>e` | open/close sidebar explorer |
| Go to definition | `gd` | jump to symbol definition (LSP) |
| Hover docs | `K` | show docs/type info for symbol under cursor |
| Rename symbol | `<leader>rn` | project-wide rename (LSP) |
| Code action | `<leader>ca` | show available fixes/refactors (LSP) |
| Format buffer | `<leader>f` | run formatter on current buffer |
| Next/prev diagnostic | `]d` / `[d` | jump between errors/warnings |
| Git hunk stage | `<leader>hs` (gitsigns) | stage current git hunk |
| Split window | `Ctrl-w v` / `Ctrl-w s` | vertical / horizontal split |
| Switch window | `Ctrl-w h/j/k/l` | move focus between splits |
| List keymaps for a key | `:verbose map <lhs>` | see where a mapping was defined |
| List buffers | `:ls` | show all open buffers |
| Macro record/play | `qa ... q`, `@a` | record into register a, replay it |

---

### Where to go next
- `:help nvim-lua-guide` — official primer on Lua configuration.
- `:help lsp` — full built-in LSP documentation.
- The repos for **kickstart.nvim**, **LazyVim**, and **AstroNvim** on GitHub are excellent to read even if you don't adopt them wholesale — they demonstrate this exact file's concepts as working code.
