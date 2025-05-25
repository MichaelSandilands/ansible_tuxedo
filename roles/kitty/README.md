# Kitty Terminal Role

This role installs the Kitty terminal emulator on the target system. It uses the official Kitty installer script to ensure the latest version is installed and then configures it for user convenience, including desktop integration.

## Purpose

The primary goals of this role are to:
* Install the latest version of Kitty terminal from the official source.
* Ensure the installation is idempotent.
* Create symbolic links for `kitty` and `kitten` executables in a standard user PATH location (`~/.local/bin`).
* Install and correctly configure `.desktop` files for application menu integration (`kitty.desktop` and `kitty-open.desktop`).
* Set Kitty as the default terminal for `xdg-terminal-exec`.

## Task Breakdown (`tasks/main.yml`)

The `tasks/main.yml` file executes the following steps in order:

1.  **Ensure Target Directories Exist:**
    * Creates essential directories if they don't already exist:
        * `~/.local/bin`: For symbolic links to executables.
        * `~/.local/share/applications`: For user-specific `.desktop` files.
        * `~/.config`: For the `xdg-terminals.list` file.

2.  **Check if Kitty is Already Installed:**
    * Uses the `ansible.builtin.stat` module to check for the existence of `~/.local/kitty.app/bin/kitty`.
    * The result of this check (`kitty_binary_stat.stat.exists`) is used to determine if the installer script needs to be run.

3.  **Download and Run Kitty Installer Script:**
    * Executes the command `curl -L https://sw.kovidgoyal.net/kitty/installer.sh | sh /dev/stdin`.
    * This script downloads and installs Kitty into `~/.local/kitty.app`.
    * This task is made idempotent by using `args: creates: "{{ ansible_env.HOME }}/.local/kitty.app/bin/kitty"` and is also conditioned on the previous `stat` check (`when: not kitty_binary_stat.stat.exists`).
    * It's marked as `changed_when: true` because the shell script's execution implies a change if it runs.

4.  **Create Symbolic Links for `kitty` and `kitten`:**
    * Uses the `ansible.builtin.file` module with `state: link` and `force: yes`.
    * Links `~/.local/kitty.app/bin/kitty` to `~/.local/bin/kitty`.
    * Links `~/.local/kitty.app/bin/kitten` to `~/.local/bin/kitten`.
    * This makes the executables available in the user's PATH.

5.  **Template `kitty.desktop` File:**
    * Uses the `ansible.builtin.template` module.
    * Copies `roles/kitty/templates/kitty.desktop.j2` to `~/.local/share/applications/kitty.desktop`.
    * The template ensures that `Exec=` and `Icon=` paths within the `.desktop` file correctly point to the user's Kitty installation (e.g., `/home/user/.local/kitty.app/...`).

6.  **Template `kitty-open.desktop` File:**
    * Similar to the `kitty.desktop` task, this templates `roles/kitty/templates/kitty-open.desktop.j2` to `~/.local/share/applications/kitty-open.desktop`.
    * This file provides integration for opening files and URLs with Kitty from file managers or other applications.

7.  **Update Desktop Database for User:**
    * Runs the command `update-desktop-database {{ ansible_env.HOME }}/.local/share/applications`.
    * This command informs the desktop environment about the new/updated `.desktop` files, so Kitty appears in application menus.
    * `changed_when: false` is used as this command doesn't reliably indicate a change in a way Ansible detects.
    * `when: ansible_connection != 'chroot'` is a precaution for specific environments.

8.  **Set Kitty as Default XDG Terminal:**
    * Uses `ansible.builtin.copy` with `content: "kitty.desktop\n"`.
    * Creates or overwrites `~/.config/xdg-terminals.list` with `kitty.desktop`.
    * This makes applications that use `xdg-terminal-exec` (like some file managers for "Open in Terminal") use Kitty by default.

## Included Templates

This role uses two template files located in `roles/kitty/templates/`:

* **`kitty.desktop.j2`**: Template for the main Kitty application desktop entry.
* **`kitty-open.desktop.j2`**: Template for the Kitty URL/file opener desktop entry.

These templates are based on the default `.desktop` files created by the official Kitty installer, with paths modified to use `{{ ansible_env.HOME }}` for portability.

## Usage

To use this role, include it in your main playbook (`local.yml` in this project) under the `roles` section:

```yaml
# local.yml
# ...
  roles:
    # ... other roles
    - role: kitty
# ...
```

This role is designed to be idempotent, meaning it can be run multiple times without causing unintended side effects. It will only make changes if Kitty is not installed or if the configuration managed by the role (like symlinks or `.desktop` files) is missing or incorrect.
