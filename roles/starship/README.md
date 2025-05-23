# Starship Prompt Role (`starship`)

This Ansible role installs the Starship cross-shell prompt and configures it for Bash. Starship is a minimal, blazing-fast, and infinitely customizable prompt for any shell.

## Purpose

The primary goals of this role are to:
1.  **Install Starship:** Download and execute the official Starship installer script, which typically places the `starship` binary in `~/.local/bin/`.
2.  **Configure Bash Integration:** Add the necessary initialization line to the user's `~/.bashrc` file to enable Starship for Bash sessions.
3.  **Ensure Idempotency:** The role will only install Starship if it's not already detected and will only add the initialization block to `.bashrc` if it's not already present.

## How it Works

The role performs the following steps (detailed in `tasks/main.yml`):

1.  **Ensure `~/.local/bin` Directory Exists:**
    * Creates the `~/.local/bin` directory if it doesn't exist, as this is a common target for user-installed binaries and the default for the Starship installer.

2.  **Check if Starship is Already Installed:**
    * Uses `ansible.builtin.stat` to check for the existence of `~/.local/bin/starship`. This helps make the installation task idempotent.

3.  **Download and Install Starship:**
    * If Starship is not found, it executes the command `curl -sS https://starship.rs/install.sh | sh -s -- --yes`.
    * The `-s -- --yes` arguments ensure the script runs non-interactively.
    * This task is conditioned on Starship not already being installed.

4.  **Ensure Starship is Initialized in `.bashrc`:**
    * Uses the `ansible.builtin.blockinfile` module to add (or ensure the presence of) the Starship initialization block in the user's `~/.bashrc` file.
    * The block added is:
      ```bash
      # Starship Prompt
      if command -v starship &> /dev/null; then
        eval "$(starship init bash)"
      fi
      ```
    * This ensures that `eval "$(starship init bash)"` is only run if the `starship` command is actually available, preventing errors.
    * A marker is used to make this block idempotent and clearly identify it as managed by Ansible.

## Requirements

* `curl` must be installed on the target system for the installer script to be downloaded. (Your `apt` role likely installs this).
* The user's shell must be Bash for the `.bashrc` modification to take effect as intended.
* It's assumed that `~/.local/bin` is (or will be) in the user's `PATH`. If not, Starship might be installed but not directly executable without specifying its full path.

## Role Variables

This role, as implemented based on our discussion, does not currently use any specific configurable variables from `defaults/main.yml` or `vars/main.yml` for its core functionality. The installation path and `.bashrc` modification are standard.

If you wish to customize the Starship configuration itself (e.g., by deploying a `starship.toml` file), you would add tasks and potentially variables for that.

## Usage

Include this `starship` role in your main playbook (`local.yml`). It's generally safe to run after basic user setup and package installations.

```yaml
# local.yml
# ...
  roles:
    # ... other roles (apt, nerdfont, kitty, etc.)
    - role: starship
```    

After the playbook run that includes this role, new Bash terminal sessions should automatically use the Starship prompt. You might need to open a new terminal or source your `~/.bashrc` (e.g., `source ~/.bashrc`) for the changes to apply to an existing session.
