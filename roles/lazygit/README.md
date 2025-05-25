# Ansible Role: Lazygit from Source (`lazygit`)

This Ansible role installs Lazygit by cloning the source code and building it using Go on Debian/Ubuntu-based systems.

## Purpose

The primary goals of this role are to:

1.  **Ensure Go is Available:** Go is a prerequisite and should be installed (e.g., via the `apt` role).
2.  **Clone Lazygit Repository:** Fetch the Lazygit source code from the official GitHub repository.
3.  **Build and Install Lazygit:** Compile the source code using `go install .` from within the cloned directory. This typically places the binary in `~/go/bin/lazygit`.
4.  **Ensure `~/go/bin` is in PATH:** Add `~/go/bin` to the user's `.bashrc` to make `lazygit` accessible from the command line.
5.  **Idempotency:** The role will attempt to avoid re-cloning and re-installing if Lazygit is already present.

## Requirements

* A Debian/Ubuntu-based system.
* `git` installed (usually handled by your `apt` role).
* `golang-go` installed (can be added to your `apt` role).
* User with permissions to write to `~/build` and `~/go/bin`.

## Role Variables

Variables that can be configured are defined in `defaults/main.yml`:

* `lazygit_repo_url`: The URL of the Lazygit git repository.
    * Default: `https://github.com/jesseduffield/lazygit.git`
* `lazygit_version`: The git branch, tag, or commit hash to checkout.
    * Default: `master`
* `lazygit_build_parent_dir`: The parent directory where the Lazygit source code will be cloned.
    * Default: `{{ ansible_env.HOME }}/build`
* `lazygit_force_rebuild`: Set to `true` to force a re-clone, rebuild, and reinstall.
    * Default: `false`

Internal variables for paths are in `vars/main.yml`.

## How it Works

1.  **Check for Existing Installation:** The role first checks if `{{ ansible_env.HOME }}/go/bin/lazygit` exists. If `lazygit_force_rebuild` is `false`, subsequent build steps might be skipped.
2.  **Ensure Build Directory:** Creates `{{ lazygit_build_parent_dir }}`.
3.  **Clone/Update Source:** Clones the Lazygit repository to `{{ lazygit_build_parent_dir }}/lazygit`.
4.  **Build & Install:** Runs `go install .` from within the source directory.
5.  **Update `.bashrc`:** Ensures `export PATH="$HOME/go/bin:$PATH"` is present in `~/.bashrc`.

## Usage

1.  Ensure `golang-go` is added to your `apt` role's package list (or install Go via another method).
2.  Include this `lazygit` role in your main playbook:

```yaml
# local.yml
# ...
  roles:
    # ... apt, other roles ...
    - role: lazygit
      # Optionally override variables:
      # vars:
      #   lazygit_version: "v0.40.2" # Example tag
      #   lazygit_force_rebuild: true
```
