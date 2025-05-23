# flatpak Role

This role is responsible for installing and managing Flatpak applications on the target system. It ensures that specified Flatpak packages are installed for the current user, utilizing the Flathub repository.

## Purpose

The primary goal of this role is to:
* Install a list of specified Flatpak applications.
* Ensure that these applications are installed for the `user`, not system-wide.
* Provide a centralized way to manage Flatpak application installations.

This role relies on the Flathub repository being configured, which is handled in the `pre_tasks` section of the main `local.yml` playbook.

## Configuration

The list of Flatpak applications to be installed is defined in the `vars/main.yml` file within this role (i.e., `roles/flatpak/vars/main.yml`).

**Example `vars/main.yml`:**
```yaml
---
# roles/flatpak/vars/main.yml
flatpak_packages:
  - io.neovim.nvim
  - md.obsidian.Obsidian
  # Add other Flatpak application IDs here
#```

To add or remove Flatpak applications, modify the `flatpak_packages` list in this variables file.

## Tasks

The `tasks/main.yml` file contains a single task that iterates over the `flatpak_packages` list and uses the `community.general.flatpak` module to ensure each package is present.

## Usage

To use this role, include it in your main playbook (`local.yml` in this project) under the `roles` section:

```yaml
# local.yml
# ...
  roles:
    # ... other roles
    - role: flatpak
# ...
```

This role is designed to be idempotent. It will only install Flatpak packages that are not already present or update them if a newer version is available on Flathub (depending on Flatpak's default behavior).
