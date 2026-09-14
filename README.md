# Neovim Config

A [LazyVim](https://www.lazyvim.org/) setup with the Python language extra enabled. Plugin versions are pinned by `lazy-lock.json`, so a fresh clone gives you the exact same environment this repo was tested with.

Instructions below cover **Ubuntu**, **Windows 11**, and **macOS** only. Windows commands assume [Windows Terminal](https://aka.ms/terminal) (preinstalled on Win 11) running **PowerShell**. If you use WSL on Windows, just follow the Ubuntu instructions instead.

## TL;DR

If you already have all the dependencies (see table below):

```sh
# Ubuntu / macOS
mv ~/.config/nvim ~/.config/nvim.bak          # optional: back up existing config
git clone <your-repo-url> ~/.config/nvim
nvim
```

```powershell
# Windows 11 (PowerShell)
Rename-Item "$env:LOCALAPPDATA\nvim" "nvim.bak"    # optional: back up existing config
git clone <your-repo-url> "$env:LOCALAPPDATA\nvim"
nvim
```

Plugins install themselves on first launch. Restart Neovim once when it finishes.

---

## Dependencies

| Dependency | Required | Needed for |
| --- | --- | --- |
| Neovim >= 0.11 | Yes | The editor itself |
| git | Yes | Bootstrapping lazy.nvim and cloning all plugins |
| curl | Yes | Mason (downloads LSP servers / tools) |
| C compiler (gcc or clang) | Yes | Compiling Treesitter parsers locally |
| Node.js + npm | Yes | pyright (the Python LSP, installed via Mason/npm) |
| Python 3 + pip + venv | Yes | Python extra tooling (mypy, ruff, virtual envs) |
| A Nerd Font | Recommended | Icons in the UI (show up as boxes otherwise) |
| fd, ripgrep | Recommended | Much faster file/text searching in the picker |
| lazygit | Optional | `<leader>gg` opens a lazygit floating window |
| uv | Optional | Faster virtualenv management for venv-selector |

Each one is covered in detail below. Run the "Check" command for each before installing anything — you likely already have several of them. On macOS everything installs through [Homebrew](https://brew.sh) (`brew --version` to check you have it; if not, install it from [brew.sh](https://brew.sh)).

### 1. Neovim >= 0.11

**Check (all OSes):**

```sh
nvim --version
```

- If the first line reads `NVIM v0.11.x` or higher: done, skip to the next dependency.
- If you get `command not found` / "not recognized": not installed.
- If it reads `v0.10` or lower: too old. **Warning (Ubuntu):** `sudo apt install neovim` is often an outdated version — use the official binary below instead.

**Install — Ubuntu (official prebuilt binary):**

```sh
curl -LO https://github.com/neovim/neovim/releases/latest/download/nvim-linux-x86_64.tar.gz
sudo rm -rf /opt/nvim
sudo mkdir -p /opt/nvim
sudo tar -C /opt -xzf nvim-linux-x86_64.tar.gz
```

Add it to your `PATH` by appending this line to `~/.bashrc` (or `~/.zshrc`):

```sh
export PATH="$PATH:/opt/nvim-linux-x86_64/bin"
```

Then reload your shell config:

```sh
source ~/.bashrc   # or: source ~/.zshrc
```

**Install — Windows 11:**

```powershell
winget install Neovim.Neovim
```

(Close and reopen your terminal afterwards so the updated `PATH` is picked up.)

**Install — macOS:**

```sh
brew install neovim
```

**Verify:**

```sh
which nvim        # Ubuntu / macOS — should point to the one you installed
where.exe nvim    # Windows — run in a NEW terminal after installing
nvim --version    # first line must be NVIM v0.11.x or higher
```

If this shows an old copy (e.g. `/usr/bin/nvim` on Ubuntu, or an old winget install on Windows), your `PATH` puts that one first — reorder so the new install wins, or remove the old one.

### 2. git

**Check (all OSes):**

```sh
git --version
```

Any recent version is fine.

**Install:**

```sh
sudo apt update && sudo apt install git   # Ubuntu
```

```powershell
winget install Git.Git                    # Windows (reopen terminal after)
```

```sh
xcode-select --install                    # macOS (git is included in the Command Line Tools)
# or: brew install git
```

### 3. curl

**Check:**

```sh
curl --version      # Ubuntu / macOS
curl.exe --version  # Windows — the ".exe" matters (in PowerShell, plain "curl" is an alias for something else)
```

Usually nothing to do: curl is preinstalled on Ubuntu, macOS, and Windows 10/11. Only if missing on Ubuntu:

```sh
sudo apt install curl
```

### 4. C compiler (gcc or clang)

Treesitter parsers are C programs that get compiled on your machine the first time Neovim installs them. Without a compiler you'll see parser build errors on first launch.

**Check:**

```sh
gcc --version
make --version
```

**Install — Ubuntu** (this bundle includes gcc, make, and friends):

```sh
sudo apt install build-essential
```

**Install — Windows 11**, pick one:

Option A — MinGW via MSYS2 (recommended by LazyVim):

```powershell
winget install MSYS2.MSYS2
```

Open the **"MSYS2 UCRT64"** app from the Start menu (not a regular PowerShell) and run:

```sh
pacman -S mingw-w64-ucrt-x86_64-gcc
```

Then add `C:\msys64\ucrt64\bin` to your user `PATH`, either via Settings → System → About → Advanced system settings → Environment Variables, or in PowerShell:

```powershell
[Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path", "User") + ";C:\msys64\ucrt64\bin", "User")
```

Reopen your terminal and verify `gcc --version` works in PowerShell.

Option B — Visual Studio Build Tools (MSVC):

```powershell
winget install Microsoft.VisualStudio.2022.BuildTools --override "--add Microsoft.VisualStudio.Workload.VCTools --includeRecommended --passive"
```

**Install — macOS** (Apple clang):

```sh
xcode-select --install
```

### 5. Node.js + npm

The Python LSP (`pyright`) is an npm package. Mason installs it for you on first use, but Mason itself needs `node`/`npm` on your system to do that.

**Check (all OSes):**

```sh
node --version
npm --version
```

Both should print a version (Node 18+ is a safe bet).

**Install — Ubuntu:** the `apt` copy tends to be old; [nvm](https://github.com/nvm-sh/nvm) is the easiest reliable route:

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
# close and reopen your terminal, then:
nvm install --lts
```

(Check nvm's repo for the latest installer version before running. Or plain `sudo apt install nodejs npm` if you don't mind an older Node.)

**Install — Windows 11:**

```powershell
winget install OpenJS.NodeJS.LTS    # reopen terminal after
```

**Install — macOS:**

```sh
brew install node
```

### 6. Python 3 + pip + venv

The Python extra runs `mypy`/`ruff` (installed by Mason via pip) and works with virtual environments, so you need Python itself plus the `venv` module.

**Check — Ubuntu / macOS:**

```sh
python3 --version                            # want 3.8+
python3 -m pip --version                     # want a version, not an error
python3 -c "import venv; print('venv ok')"   # want "venv ok"
```

**Check — Windows** (note: the command is `python`, not `python3`):

```powershell
python --version
py -m pip --version
python -c "import venv; print('venv ok')"
```

**Install — Ubuntu** (the `-venv` package is frequently missing by default!):

```sh
sudo apt install python3 python3-pip python3-venv
```

**Install — Windows 11:**

```powershell
winget install Python.Python.3.12    # reopen terminal after
```

**Install — macOS:**

```sh
brew install python
```

### 7. A Nerd Font (recommended)

Icons throughout the UI come from [Nerd Fonts](https://www.nerdfonts.com/). Without one, icons appear as little boxes or question marks. This is purely cosmetic — everything else works.

**Check — Ubuntu:**

```sh
fc-list | grep -i "nerd"
```

**Install — Ubuntu:**

```sh
mkdir -p ~/.local/share/fonts
cd ~/.local/share/fonts
curl -LO https://github.com/ryanoasis/nerd-fonts/releases/latest/download/JetBrainsMono.tar.gz
tar -xzf JetBrainsMono.tar.gz
rm JetBrainsMono.tar.gz
fc-cache -f
```

**Install — Windows 11:**

```powershell
curl.exe -LO https://github.com/ryanoasis/nerd-fonts/releases/latest/download/JetBrainsMono.zip
Expand-Archive JetBrainsMono.zip -DestinationPath .\JetBrainsMono-NF
```

Open the `JetBrainsMono-NF` folder in Explorer, select all the `.ttf` files, right-click → **Install** (or "Install for all users").

**Install — macOS:**

```sh
brew install --cask font-jetbrains-mono-nerd-font
```

**Important (all OSes):** installing the font is only half the job — you must also select "JetBrainsMono Nerd Font" in your terminal's settings:

- Windows Terminal: Settings → your profile → Appearance → Font face
- Ubuntu: your terminal emulator's preferences (GNOME Terminal, Konsole, etc.)
- macOS: Terminal.app or iTerm2 preferences

Then restart the terminal. Verify with `fc-list | grep -i "nerd"` on Ubuntu; on Windows/macOS, just check that icons render in Neovim.

### 8. Optional extras

**fd** — fast file finding for the picker:

```sh
fd --version               # check (Ubuntu note: the binary is called "fdfind")
sudo apt install fd-find   # Ubuntu
```

```powershell
winget install sharkdt.fd  # Windows
```

```sh
brew install fd            # macOS
```

**ripgrep** — fast live text search (`<leader>/` and grep-related pickers):

```sh
rg --version          # check
sudo apt install ripgrep   # Ubuntu
```

```powershell
winget install BurntSushi.ripgrep.MSVC   # Windows
```

```sh
brew install ripgrep  # macOS
```

**lazygit** — LazyVim's `<leader>gg` opens lazygit inside Neovim. Without it that keybinding just errors, which is fine if you don't use it:

```sh
lazygit --version          # check
```

Ubuntu (no repo package; grab the latest .deb):

```sh
LAZYGIT_VERSION=$(curl -s "https://api.github.com/repos/jesseduffield/lazygit/releases/latest" | grep -Po '"tag_name": "v\K[^"]*')
curl -Lo lazygit.deb "https://github.com/jesseduffield/lazygit/releases/download/v${LAZYGIT_VERSION}/lazygit_${LAZYGIT_VERSION}_Linux_x86_64.deb"
sudo dpkg -i lazygit.deb
```

```powershell
winget install JesseDuffield.lazygit   # Windows
```

```sh
brew install lazygit        # macOS
```

**uv** — fast Python package/venv manager that venv-selector can use:

```sh
uv --version    # check
curl -LsSf https://astral.sh/uv/install.sh | sh          # Ubuntu / macOS
```

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"   # Windows
```

---

## Installing this config

**1. Back up any existing Neovim config** (skip if you don't have one or don't care about it):

```sh
mv ~/.config/nvim ~/.config/nvim.bak    # Ubuntu / macOS
```

```powershell
Rename-Item "$env:LOCALAPPDATA\nvim" "nvim.bak"    # Windows
```

**2. Optional clean slate** — if you've used Neovim before, old plugins/state/cache can conflict with this setup. Remove them if you want a guaranteed-fresh start (if you backed up in step 1 and want to keep its data, don't run this — the tradeoff is yours):

```sh
rm -rf ~/.local/share/nvim ~/.local/state/nvim ~/.cache/nvim    # Ubuntu / macOS
```

```powershell
Remove-Item "$env:LOCALAPPDATA\nvim-data" -Recurse -Force       # Windows (holds plugins + state + data)
```

**3. Clone the repo:**

```sh
git clone <your-repo-url> ~/.config/nvim    # Ubuntu / macOS
```

```powershell
git clone <your-repo-url> "$env:LOCALAPPDATA\nvim"    # Windows
```

**4. Open Neovim:**

```sh
nvim
```

## What happens on first launch

You don't have to do anything — just watch:

1. **lazy.nvim bootstraps itself.** The plugin manager is cloned automatically — no manual plugin manager setup, ever. (On Ubuntu/macOS it lands in `~/.local/share/nvim/lazy/lazy.nvim`; on Windows in `~\AppData\Local\nvim-data\lazy\lazy.nvim`.)
2. **All plugins install.** A floating window shows progress. Versions come from `lazy-lock.json`, so you get the exact commits this repo was tested with.
3. **Treesitter parsers compile** during that install (this is where a missing C compiler would fail).
4. **Restart Neovim** (`:qa`, then `nvim` again) once the install window says it's done.
5. **The first time you open a Python file**, Mason automatically installs the language tooling — `pyright` (LSP), `ruff` (formatting/linting), and `mypy` (type checking). This needs `node`/`npm` to be present (see dependency 5).

## Verifying the install

Inside Neovim, run the built-in health check:

```
:checkhealth
```

It inspects your system and reports anything missing in a readable report. Focus on **ERROR** lines; warnings are often optional niceties. Useful focused checks:

- `:checkhealth lazy` — the plugin manager's own dependency report.
- `:Mason` — open the Mason UI; after opening a Python file you should see `pyright`, `ruff`, etc. listed as installed. If an install failed, `:MasonLog` shows why (a Node/npm error there almost always means dependency 5 is missing).

## Daily use notes

- **Leader key:** `<Space>`. Press it and wait a second — which-key pops up showing all available keybindings from that prefix. This is the best way to explore the config.
- **Pick a virtual environment:** run `:VenvSelect` (from venv-selector) inside a Python project to activate one of its venvs for LSP and tooling.
- **Update plugins:** `:Lazy update`, then commit the updated `lazy-lock.json` to this repo so your other machines get the same versions.
- **Manage LSP tools:** `:Mason` to install/update/remove things like pyright, ruff, mypy.

## Troubleshooting

| Symptom | Likely cause / fix |
| --- | --- |
| Icons render as boxes | Nerd Font missing, or terminal isn't set to use it (dependency 7) |
| Treesitter / parser errors on startup | No C compiler (dependency 4). Windows: make sure `gcc --version` works in PowerShell, i.e. the MSYS2 `PATH` entry is set |
| pyright fails in `:Mason` | Node/npm missing (dependency 5) — check `:MasonLog` for the exact error |
| `nvim --version` still shows 0.10 or older | Old binary earlier in your `PATH` — check `which nvim` (Ubuntu/macOS) or `where.exe nvim` (Windows) |
| Plugin clone failures at startup | git missing or no network access (dependency 2) |
| `curl` behaves oddly on Windows | Use `curl.exe`, not `curl`, in PowerShell (plain `curl` is an alias for `Invoke-WebRequest`) |
| winget not found on Windows | Install "App Installer" from the Microsoft Store, then reopen the terminal |
