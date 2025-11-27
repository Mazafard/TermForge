# TermForge

> Forked from [jazik/termenv](https://github.com/jazik/termenv).

TermForge is an Ansible playbook that forges a full-featured terminal
environment on Linux and macOS hosts. It keeps the original
termenv spirit while adding macOS polish, automatic iTerm2 setup, and
Kubernetes helper tooling that lights up as soon as `kubectl` is in your
`PATH`.

## Highlights

- **Shell stack** – `zsh`, `oh-my-zsh`, Powerlevel10k, color + autosuggest
	plugins, and curated defaults.
- **Terminal UX** – `tmux` with a powerline theme, `fzf`, custom keymaps, and
	smart Neovim ↔ tmux navigation glue.
- **Editor batteries** – Neovim + Lua config, Telescope/Ripgrep, Treesitter,
	Copilot(optional), and CSpell ready to go.
- **Fonts + UI** – Nerd Fonts installer, macOS font placement, automatic
	iTerm2 install via Homebrew Cask.
- **Kubernetes helpers** – `k9s`, `kubectx`, and `kubens` automatically
	installed when `kubectl` is detected.

Inspired by:
- [Terminal History Auto Suggestions As You Type With Oh My Zsh](https://www.dev-diaries.com/blog/terminal-history-auto-suggestions-as-you-type/)
- [bcampolo/nvim-starter-kit](https://github.com/bcampolo/nvim-starter-kit)

## Prerequisites

Run the playbook with a user that has `sudo` privileges.

### Fedora

```
sudo dnf install git ansible
```

### Ubuntu

```
sudo apt-get update
sudo apt-get install git ansible
```

### macOS

Install Command Line Tools and [Homebrew](https://brew.sh/), then install
Ansible and Git:

```
xcode-select --install
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install ansible git
```

Homebrew is also required so TermForge can install `zsh`, `tmux`, `node`,
`k9s`, `kubectx`, `kubens`, and iTerm2.

> **Heads-up:** On Apple Silicon you might see an interpreter-discovery warning
> similar to `/opt/homebrew/bin/python3.14`. This is harmless, but you can silence
> it by exporting `ANSIBLE_PYTHON_INTERPRETER=/opt/homebrew/bin/python3` (or by
> setting `ansible_python_interpreter` in `hosts`).

## Install

```
git clone https://github.com/<your-user>/termforge.git
cd termforge
ansible-playbook -i hosts --ask-become-pass termforge.yml
```

> The legacy `termenv.yml` playbook now just imports `termforge.yml`, so
> existing scripts keep working while you migrate.

## Customization with Tags

All roles are enabled by default. Use tags to skip or run a subset:

```
ansible-playbook -i hosts termforge.yml --skip-tags tmux
ansible-playbook -i hosts termforge.yml --tags "neovim,k8s-tools"
```

Tag names match the role names (see `termforge.yml`).

## Fonts

The `nerd-fonts` tag installs the latest Fira Mono Nerd Font by default. To
install another Nerd Font:

```
ansible-playbook -i hosts nerdfonts.yml -e "font_name=YourFontName"
```

The `YourFontName` value is the Nerd Font zip filename without `.zip`. See the
[Nerd Fonts download page](https://www.nerdfonts.com/font-downloads).

On macOS fonts land in `~/Library/Fonts/<FontName>` and are immediately
available, so `fc-cache` is not required.

## macOS + iTerm2

When TermForge runs on macOS it automatically applies the `iterm2` tag and
installs iTerm2 via Homebrew Cask. You can still target or skip the role:

```
ansible-playbook -i hosts termforge.yml --skip-tags iterm2
ansible-playbook -i hosts termforge.yml --tags iterm2
```

After provisioning:
- Launch iTerm2 → *Preferences → Profiles → Text* and select the Nerd Font you
	installed (Fira Mono Nerd Font by default).
- Toggle *Use ligatures* if you want stylistic glyphs.
- If you sync settings, open *Preferences → General → Preferences* and point it
	to your sync folder before rerunning the playbook.

## Kubernetes helpers

The `kubernetes-tools` role installs `k9s`, `kubectx`, and `kubens` whenever
`kubectl` is already available. Nothing is installed if `kubectl` is missing,
so your environment stays lean until you actually use Kubernetes.

## Neovim Copilot

Copilot/CopilotChat are optional to avoid noisy errors on fresh systems. To add
them later:

```
ansible-playbook -i hosts neovim-copilot.yml
```

## Neovim Cheat Sheet

| Key | Command | Key | Command |
|-----|---------|-----|---------|
| `<C-h>` | Jump to left window | `<Space>` | `<leader>` |
| `<C-l>` | Jump to right window | `<leader>wq` | Save + quit |
| `<C-j>` | Jump down | `<leader>qq` | Quit without saving |
| `<C-k>` | Jump up | `<leader>ww` | Save |
| `<leader>sv` | Split vertically | `<leader>ee` | Toggle file explorer |
| `<leader>sh` | Split horizontally | `<leader>ff` | Find file |
| `<leader>sx` | Close split | `<leader>fg` | Live grep |
| `:Git` | Fugitive | `:Neogit` | Neogit |
| `<leader>al` | Git log | `<leader>ar` | Git log (selection) |
| `<leader>af` | Git log (file) | `<leader>gb` | Inline git blame |
| `<leader>ha` | Add file to Harpoon | `<leader>hh` | Toggle Harpoon UI |
| `<leader>gg` | LSP definition | `<leader>gD` | LSP declaration |
| `<leader>gd` | Jump to definition | `<leader>gi` | Jump to implementation |
| `<C-Space>` | Completion | `<leader>fw` | Grep word under cursor |
| `:Copilot setup` | Configure Copilot | `:CopilotChat` | Chat window |
| `<C-j>` | Accept Copilot suggestion |  |  |

## Screenshots

![TermForge prompt](../media/termenv.png?raw=true)

![TermForge tmux](../media/nvim.png?raw=true)

## Testing the playbooks

### Docker / Podman / Moby

Use the [Dockerfile](Dockerfile) to spin up a disposable TermForge shell.

```
docker build -t termforge . --build-arg DISTRO=fedora
docker build -t termforge . --build-arg DISTRO=ubuntu
docker run --rm -it termforge
docker run --rm -it -v .:/home/termforge/termforge:Z termforge
```

### With Vagrant

Install `vagrant`, plus `libvirt` and `VirtualBox` providers if you plan to use
them.

Fedora quick-start (VirtualBox requires [RPMFusion](https://rpmfusion.org/Configuration/)):

```
sudo dnf install vagrant vagrant-libvirt VirtualBox libvirt
sudo usermod -a -G libvirt ${USER}
```

Allow NFS inside the `libvirt` firewall zone when using Fedora:

```
sudo systemctl --now enable virtnetworkd.service
sudo firewall-cmd --permanent --zone=libvirt --add-service=nfs
sudo firewall-cmd --reload
```

`Vagrantfile` exposes Fedora and Ubuntu definitions with a few handy flags:

```
vagrant [--local=no|yes] [--do-install=yes|no] <command> [fedora|ubuntu]

--local       Run playbook from GitHub (no, default) or locally (yes)
--do-install  Install dependencies on the guest (yes, default) or skip (no)
<command>     Standard vagrant verbs (up, destroy, ssh, ...)
fedora|ubuntu Target one distro; omit to manage both
```

Example (local Fedora test run):

```
vagrant --local=yes up fedora
vagrant ssh fedora
vagrant destroy fedora
```

Provisioning fails fast if the Ansible run fails, so verify the resulting
environment manually after the VM boots.
