# Fedora Workstation Provisioning

Ansible playbook and roles to automate the setup of a Fedora workstation. It
adds the required system repositories, installs development tools (Docker,
Nvidia, Neovim, uv, Ollama, and more), deploys SSH keys and a git identity,
and lays down dotfiles using GNU Stow.

## Quick Start

Run the following on the target machine:

```bash
# Bring the base system up to date
sudo dnf upgrade -y

# Install Ansible and git
sudo dnf install ansible git -y

# Provision: pull this repository and run local.yml
ansible-pull -U "https://github.com/MichaelSandilands/ansible_tuxedo.git" -K
```

You will be prompted for your sudo password (`-K`) and then for the Ansible
Vault password (because `ansible.cfg` sets `ask_vault_pass = True`), which is
used to decrypt the SSH private key(s).

> **Dependencies:** the playbook uses modules from the `community.general`
> collection (`copr`, `ini_file`, `git_config`). These ship bundled with
> Fedora's `ansible` package, so installing `ansible` (not `ansible-core`)
> as shown above is all that is required. There is **no** `requirements.yml`
> and the playbook depends on **no** Ansible Galaxy roles.

## Command Explanation

`ansible-pull` runs in "pull mode": the target machine clones this repo and
applies `local.yml` to itself.

- `-U`: the repository URL.
- `-K`: prompts for the sudo password so Ansible can install system packages
  and write to `/etc`.

## Post-Installation Notes

1. **Reboot.** Recommended after the first provision, especially because of
   the Nvidia (`akmod-nvidia`) and Docker installs.
2. **Neovim & Molten.** See the [dotfiles repo](https://github.com/MichaelSandilands/dotfiles.git).

## Layout

```
ansible_tuxedo/
├── ansible.cfg          # ask_vault_pass = True
├── local.yml            # play: sudo guard (pre_tasks) + ordered roles
└── roles/
    ├── repositories/    # third-party repos + GPG keys + COPR + openh264
    ├── packages/        # all dnf packages (optional R stack via a toggle)
    ├── docker/          # enable the service + add user to docker group
    ├── dev_tools/       # uv + Ollama (vendor install scripts)
    ├── ssh_keys/        # vault-encrypted SSH keys, config, git identity
    ├── dotfiles/        # clone + stow dotfiles, tmux/neovim/starship setup
    └── auto_updates/    # dnf5-automatic unattended updates
```

The play body is deliberately thin: `pre_tasks` does nothing but assert the
playbook is being run via `sudo` from a normal user (so configuration never
lands in `/root`). Everything else is a role, applied in order.

## Role Overview

- **repositories** — adds the Tuxedo, Docker CE, RPM Fusion, openh264, and
  COPR (Starship / RStudio / WezTerm) sources and imports their GPG keys.
- **packages** — installs every system and dev package in one idempotent
  pass. An optional R / data-science stack is gated behind
  `packages_install_r` (off by default).
- **docker** — enables and starts the Docker service and adds the user to the
  `docker` group.
- **dev_tools** — installs/updates Astral's `uv` (user space) and `Ollama`
  (system service) via their official scripts, with `set -o pipefail`.
- **ssh_keys** — deploys SSH private/public keys (private keys are Ansible
  Vault encrypted) and the SSH config, and sets the global git identity.
- **dotfiles** — clones the dotfiles repo, symlinks configs with `stow`,
  installs TPM and tmux plugins, builds the Neovim provider venv, and wires
  up the Starship prompt.
- **auto_updates** — configures `dnf5-automatic`. See its README for the
  note about unattended kernel updates on this Nvidia machine.

## Optional Features

| Feature     | How to enable                                 |
| ----------- | --------------------------------------------- |
| R / RStudio | `ansible-pull ... -e packages_install_r=true` |

## Vault

SSH private keys live in `roles/ssh_keys/files/` encrypted with Ansible
Vault (AES-256) and are safe to commit. See `roles/ssh_keys/README.md` for
the full workflow of adding, viewing, and rotating keys.
