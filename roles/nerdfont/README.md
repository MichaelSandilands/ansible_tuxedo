# Nerd Font Installation Role (`nerdfont`)

This Ansible role downloads and installs a specified Nerd Font into the user's local font directory (`~/.local/share/fonts`) and then rebuilds the system's font cache. Nerd Fonts are popular for providing a wide range of glyphs and icons useful in terminals and code editors.

## Purpose

The primary goals of this role are to:

- Download a specific Nerd Font archive (e.g., Hack, FiraCode, JetBrainsMono) from the official Nerd Fonts GitHub releases.

- Install the font files into a user-specific directory, making them available to that user's applications.

- Ensure the system font cache is updated so the newly installed font is recognized.

- Provide a configurable way to select the font name and version.

## How it Works

The role performs the following steps (detailed in `tasks/main.yml`):

- Ensures User Font Directory Exists: Creates `~/.local/share/fonts` if it's not already present.

- Checks for Existing Installation: Determines if the specified Nerd Font has already been installed by checking for its dedicated subdirectory (e.g., ~/.local/share/fonts/JetBrainsMonoNerdFont). This makes the role idempotent.

- Downloads Font Archive: If the font isn't already installed, it downloads the `.zip` archive for the specified `nerdfont_name` and `nerdfont_version` from `https://github.com/ryanoasis/nerd-fonts/releases/download/...` to a temporary location (`/tmp/`).

- Creates Font Subdirectory: Creates a specific subdirectory within `~/.local/share/fonts` (e.g., `~/.local/share/fonts/JetBrainsMonoNerdFont`) to keep the font files organized.

- Unarchives Font: Extracts the contents of the downloaded `.zip` file into the newly created subdirectory.

- Cleans Up Archive: Removes the temporary `.zip` file from `/tmp/`.

- Rebuilds Font Cache: Executes `fc-cache -fv` to update the system's font information. This step only runs if new font files were actually unarchived.

## Role Variables

The main variables for this role are defined in `vars/main.yml` (or can be overridden in `defaults/main.yml` or when calling the role). Key variables include:

- `nerdfont_name`: The name of the Nerd Font to install (e.g., `"Hack"`, `"FiraCode"`, `"JetBrainsMono"`). This must match a font name available in the Nerd Fonts releases.

-     Default: `"JetBrainsMono"` (as per your `vars/main.yml`)

- `nerdfont_version`: The tagged version of the Nerd Font to download (e.g., `"v3.2.1"`, `"v3.4.0"`). Check the Nerd Fonts releases page for available versions.

-     Default: `"v3.4.0"` (as per your `vars/main.yml`)

- `user_font_dir`: The base directory for user-specific fonts.

-     Default: `{{ ansible_env.HOME }}/.local/share/fonts`

- `nerdfont_install_subdir_name`: The name of the subdirectory created for this font.

-     Default: `{{ nerdfont_name }}NerdFont`

- `nerdfont_install_path`: The full path where the font will be installed.

-     Default: `{{ user_font_dir }}/{{ nerdfont_install_subdir_name }}`

Usage

To use this role, include it in your main playbook (`local.yml` in this project) under the roles section.

```yaml
# local.yml
  roles:
    # ... other roles
    - role: nerdfont
      # Optionally override variables here if needed:
      # vars:
      #   nerdfont_name: "FiraCode"
      #   nerdfont_version: "v3.1.1" # Ensure this version exists for FiraCode
```

This role is designed to be idempotent. If the specified Nerd Font (based on the subdirectory name) is already installed, the download and extraction steps will be skipped. The font cache will only be rebuilt if new font files are actually installed.
