# Role: dotfiles

Deploys user configuration files with GNU Stow and bootstraps the tmux and
Neovim environments.

## What it does

1. **Clone** the dotfiles repository (`dotfiles_repo`).
2. **Stow** each package in `dotfiles_stow_packages` into the home directory.
   `stow --verbose` is used so the role can report an honest `changed` state;
   a genuine file conflict makes stow fail loudly (the correct behaviour).
3. **TPM** — clone the Tmux Plugin Manager and install plugins headlessly.
4. **Neovim venv** — create `~/.virtualenvs/neovim` and install the Python
   remote-plugin dependencies (`dotfiles_neovim_pip`). Neovim's
   `python3_host_prog` points at this venv.
5. **Starship** — add the prompt initialization to `.bashrc`.

Every task runs as the target user via explicit `become` + `become_user`, so
nothing in the user's home ends up owned by root.

## Variables

| Variable                | Default                          | Description                          |
| ----------------------- | -------------------------------- | ------------------------------------ |
| `dotfiles_user`         | detected sudo user               | Account receiving the dotfiles.      |
| `dotfiles_home`         | `/home/<user>`                   | Home directory.                      |
| `dotfiles_repo`         | the MichaelSandilands dotfiles   | Source repository.                   |
| `dotfiles_stow_packages`| `[kitty, nvim, tmux]`            | Stow packages to symlink.            |
| `dotfiles_neovim_pip`   | provider + Molten deps           | Python packages for the nvim venv.   |

## Requirements

- `stow`, `neovim`, `python3` (with `venv`), `git`, and `starship` — all
  installed by the `packages` role, which must run first.

## Usage

```yaml
roles:
  - role: packages
  - role: dotfiles
    vars:
      dotfiles_user: "{{ target_user }}"
      dotfiles_home: "{{ target_home }}"
```

> Starship is installed by the `packages` role (from the `atim/starship`
> COPR enabled in `repositories`); this role only wires it into `.bashrc`.
> There is no dependency on any Ansible Galaxy role.
