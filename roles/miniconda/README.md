# User Miniconda Installation Role (`user_miniconda`)

This Ansible role installs Miniconda into the current user's home directory and initializes it for common shells.

## Purpose

The primary goals of this role are to:
* Download the latest Miniconda installer script (or a specified one).
* Install Miniconda to a user-configurable path within their home directory (default: `~/miniconda3`).
* Initialize Miniconda for specified shells (e.g., bash, zsh) by modifying the user's shell configuration files.
* Perform these actions without requiring root privileges for the installation itself (downloads and installation happen as the Ansible user).

## How it Works

The role performs the following steps:

1.  **Checks for Existing Installation:** Determines if Miniconda is already installed by checking for `~/miniconda3/bin/conda`.
2.  **Downloads Installer:** If not installed, downloads the Miniconda installer script to `/tmp/`.
3.  **Installs Miniconda:** Runs the downloaded installer script with the `-b` (batch mode) and `-p` (prefix) flags to install Miniconda into the specified user directory.
4.  **Initializes Conda:** Runs `conda init` for the specified shells to update the user's shell configuration files (e.g., `~/.bashrc`, `~/.zshrc`).
5.  **Cleans Up:** Removes the temporary downloaded installer script.

## Requirements

* `curl` or `wget` (implicitly used by `ansible.builtin.get_url`).
* Internet access to download from `repo.anaconda.com`.
* The user running the Ansible playbook should have write access to their home directory and `/tmp/`.

## Role Variables

Default variables for this role are defined in `roles/user_miniconda/defaults/main.yml`:

* `miniconda_installer_script_name`: The name of the Miniconda installer script.
    * Default: `"Miniconda3-latest-Linux-x86_64.sh"`
* `miniconda_installer_url`: Fully constructed URL to download the installer.
* `miniconda_install_path`: The target installation directory within the user's home.
    * Default: `{{ ansible_env.HOME }}/miniconda3`
* `miniconda_init_shells`: A list of shells for which `conda init` will be run.
    * Default: `["bash", "zsh"]`

These can be overridden when the role is called.

## Usage

Include this `user_miniconda` role in your main playbook (`local.yml`). Since this role installs into the user's home directory and modifies user-specific shell files, `become: true` is generally not needed for the role itself.

```yaml
# local.yml
# ...
  roles:
    # ... other roles
    - role: user_miniconda
      # Optionally override variables here:
      # vars:
      #   miniconda_installer_script_name: "Miniconda3-py39_4.12.0-Linux-x86_64.sh"
      #   miniconda_init_shells:
      #     - bash
# ...
```
