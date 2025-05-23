# Custom R Installation and Setup Role (`R`)

This Ansible role is a custom wrapper designed to install a specific version of R using the `appsilon.r_language` Galaxy role and then ensure that this specific R installation is accessible via the system PATH by creating necessary symbolic links. This role is configurable via variables defined in `defaults/main.yml`.

## Purpose

The primary goals of this role are to:

1.  **Install a Specific R Version:** Leverage the `appsilon.r_language` role to download and install a designated version of R (e.g., R 4.5.0) from pre-compiled `.deb` packages. The target R version is configurable.

2.  **Configure System PATH:** Create symbolic links for the `R` and `Rscript` executables from the version-specific installation directory (typically under `/opt/R/`) to a standard PATH location (e.g., `/usr/local/bin`). This makes the chosen R version the default when `R` or `Rscript` is called from the terminal.

This role provides a convenient and configurable way to manage a specific R installation and its accessibility.

## How it Works

The role performs the following steps (detailed in `tasks/main.yml`):

1.  **Install R version `{{ r_target_version }}` using `appsilon.r_language` role:**
    * Dynamically includes the `appsilon.r_language` role using `ansible.builtin.include_role`.
    * Passes the `r_target_version` variable (defined in this role's `defaults/main.yml` or overridden) to `appsilon.r_language` via its `r_versions` parameter, instructing it to install only the specified version.
    * This step requires `become: true` as the `appsilon.r_language` role installs system packages and R itself.

2.  **Ensure symlink target directory `{{ r_symlink_target_dir }}` exists:**
    * Uses `ansible.builtin.file` to create the target directory for symlinks (e.g., `/usr/local/bin`) if it doesn't already exist.
    * This step requires `become: true`.

3.  **Create symlinks for R `{{ r_target_version }}` executables in `{{ r_symlink_target_dir }}`:**
    * Uses `ansible.builtin.file` to create symbolic links for each executable listed in `r_executables_to_symlink` (e.g., `R`, `Rscript`).
    * The source of the symlinks points to the specific version installed by `appsilon.r_language` within `r_install_base_path` (e.g., `/opt/R/4.5.0/bin/R`).
    * The destination for the symlinks is the `r_symlink_target_dir`.
    * This step also requires `become: true`.

## Requirements

* The `appsilon.r_language` role must be installed and available in Ansible's roles path (e.g., via `ansible-galaxy install appsilon.r_language` or listed in `requirements.yml`).
* The system must be a Debian-based distribution supported by the `.deb` packages that `appsilon.r_language` uses (e.g., Ubuntu).
* The playbook executing this role must allow for privilege escalation (`become: true`) as both installing R and creating system-wide symlinks require root permissions. The `become: true` directives are handled within this role's tasks.

## Role Variables

Default variables for this role are defined in `roles/R/defaults/main.yml`:

* `r_target_version`: The specific version of R to install and symlink.
    * Default: `"4.5.0"`
* `r_install_base_path`: The base directory where `appsilon.r_language` installs different R versions.
    * Default: `"/opt/R"`
* `r_symlink_target_dir`: The directory where symlinks for R executables will be created (should be in system PATH).
    * Default: `"/usr/local/bin"`
* `r_executables_to_symlink`: A list of R executables to symlink.
    * Default: `["R", "Rscript"]`

These variables can be overridden when the role is called in a playbook if needed.

## Usage

Include this `R` role in your main playbook (`local.yml`).

```yaml
# local.yml
# ...
  roles:
    # ... other roles (apt, docker, quarto, etc.)

    # Install and set up a specific R version
    - role: R # Your custom R role
      # Vars can be overridden here if defaults are not suitable:
      # vars:
      #   r_target_version: "4.4.1"
      #   r_symlink_target_dir: "/usr/bin"
```
