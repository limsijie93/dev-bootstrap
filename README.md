# dev-bootstrap

Idempotent Ansible playbook to install a shared productivity CLI toolchain on **Ubuntu/Debian laptops and servers**.

## What it installs

| Tool | Why |
|------|-----|
| **fzf** | Fuzzy-find files, history, and branches — fastest navigation upgrade |
| **bat** | Syntax-highlighted `cat` for reading configs and logs |
| **eza** | Modern `ls` with git status and tree view |
| **Starship** | Fast, informative shell prompt (git + exit status) |
| **ripgrep (`rg`)** | Fast code/log search; respects `.gitignore` |
| **fd** | Simple, fast file finder; pairs with fzf |
| **zoxide** | Smart `cd` that learns directories you use |
| **delta** | Readable `git diff` / `git show` |
| **jq** / **yq** | JSON and YAML processing for APIs and manifests |
| **btop** | Interactive CPU/memory/disk monitor |
| **tmux** | Persistent multiplexed SSH sessions |
| **lazygit** | Terminal UI for everyday git workflows |
| **duf** / **dust** | Clear disk usage overview and directory sizes |
| **tealdeer (`tldr`)** | Practical command examples |
| **xh** | Clean HTTP client for API debugging |
| **direnv** | Auto-load per-project environment variables |
| **atuin** | Searchable shell history (opt-in; off by default) |

Warp is intentionally omitted (desktop terminal client, not a server package).

## Prerequisites

On the **control machine** (the laptop you run Ansible from):

```bash
sudo apt update
sudo apt install -y ansible
```

Targets must be Ubuntu/Debian with SSH access (or local for the laptop itself) and sudo.

## Quick start

```bash
cd dev-bootstrap

# Bootstrap this laptop
ansible-playbook playbooks/bootstrap.yml --limit laptops --ask-become-pass

# Bootstrap a remote server (add it under inventory/hosts.yml first)
ansible-playbook playbooks/bootstrap.yml --limit servers --ask-become-pass
```

After a successful run, open a new shell (or `source ~/.bashrc`) so aliases and Starship load.

## Add a server

Edit [`inventory/hosts.yml`](inventory/hosts.yml):

```yaml
servers:
  hosts:
    my-vps:
      ansible_host: 203.0.113.10
      ansible_user: ubuntu
```

Then:

```bash
ansible-playbook playbooks/bootstrap.yml --limit my-vps --ask-become-pass
```

## Customize

Feature flags and pinned versions live in [`group_vars/all.yml`](group_vars/all.yml). Examples:

```yaml
install_lazygit: false   # skip on a minimal host
install_atuin: true      # enable synced/searchable history
eza_version: "0.21.0"    # bump deliberately for upgrades
```

Group overrides:

- [`group_vars/laptops.yml`](group_vars/laptops.yml)
- [`group_vars/servers.yml`](group_vars/servers.yml)

Re-run the playbook anytime — it is idempotent and safe to re-apply for upgrades or new machines.

## Layout

```
dev-bootstrap/
  ansible.cfg
  inventory/hosts.yml
  playbooks/bootstrap.yml
  group_vars/
  roles/
    cli_tools/      # apt + GitHub release installs
    starship/       # binary + lean prompt config
    common_shell/   # bash/zsh aliases and init hooks
```

## Notes

- Debian/Ubuntu ship `bat` as `batcat` and `fd` as `fdfind`; the playbook symlinks them to `bat` / `fd` under `/usr/local/bin`.
- Shell snippets are wrapped in a managed `DEV-BOOTSTRAP` block so re-runs update in place without duplicating config.
- Non-Debian OS families (RHEL, Alpine, macOS) are out of scope for v1.
