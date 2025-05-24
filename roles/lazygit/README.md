# Lazygit Installation Role (`lazygit`)

This Ansible role installs Lazygit, a simple terminal UI for git commands. It downloads the specified or latest version of the Lazygit binary from the official GitHub releases, extracts it, and places it in a system PATH directory.

## Purpose

The primary goals of this role are to:
* Download the Lazygit binary for the appropriate architecture.
* Install the Lazygit CLI to a standard location (default: `/usr/local/bin/lazygit`).
* Allow specification of a particular Lazygit version or install the latest available.
* Ensure the installation is idempotent.

## How it Works

The role performs the following steps:

1.  **Determine Target Version:** If `lazygit_version` is set to "latest", it queries the GitHub API to find the latest release tag. Otherwise, it uses the specified version.
2.  **Check Current Version:** If Lazygit is already installed, its version is checked.
3.  **Decide on Installation:** Installs or updates Lazygit if it's not present or if the installed version doesn't match the target version.
4.  **Download Archive:** Downloads the Lazygit `tar.gz` archive for the target version and system architecture from GitHub releases to a temporary location.
5.  **Extract Binary:** Extracts the `lazygit` binary from the archive to a temporary directory.
6.  **Install Binary:** Copies the extracted `lazygit` binary to the specified installation directory (e.g., `/usr/local/bin/lazygit`) and sets execute permissions. This step requires `become: true` (root privileges).
7.  **Clean Up:** Removes the temporary downloaded archive and extracted files.

## Requirements

* `curl` or `wget` (implicitly used by `ansible.builtin.get_url` and `ansible.builtin.uri`).
* Internet access to download from GitHub.
* `become` privileges for installing to the system PATH directory.
* The `community.general` collection might be needed for `regex_search` if not available by default (though usually it is).

## Role Variables

Default variables for this role are defined in `roles/lazygit/defaults/main.yml`:

* `lazygit_version`: The version of Lazygit to install (e.g., `"0.50.0"`). Set to `"latest"` to attempt to download the most recent release.
    * Default: `"latest"`
* `lazygit_architecture_suffix`: Automatically determined based on `ansible_architecture` (e.g., `x86_64`, `arm64`).
* `lazygit_install_dir`: The directory where the `lazygit` binary will be installed.
    * Default: `"/usr/local/bin"`
* `lazygit_install_path`: The full path to the installed `lazygit` binary.
    * Default: `{{ lazygit_install_dir }}/lazygit`

These variables can be overridden when the role is called in a playbook.

## Usage

Include this `lazygit` role in your main playbook (`local.yml`).

```yaml
# local.yml
# ...
  roles:
    # ... other roles
    - role: lazygit
      # Optionally override variables here if needed:
      # vars:
      #   lazygit_version: "0.40.2" # Example for a specific version
# ...
```
