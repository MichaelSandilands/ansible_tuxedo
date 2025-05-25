# Ansible Role: Neovim from Source (`neovim`)

This Ansible role installs the **latest stable version** of Neovim by building it from source on Debian/Ubuntu-based systems.

## Purpose

The primary goals of this role are to:

1.  **Install Build Dependencies:** Ensure all necessary packages for compiling Neovim are present.
2.  **Clone Neovim Repository:** Fetch the `stable` branch of the Neovim source code from the official GitHub repository.
3.  **Build Neovim:** Compile the source code.
4.  **Install Neovim:** Place the compiled binaries in a system-wide location (typically `/usr/local/bin/nvim`).
5.  **Idempotency:** The role will avoid re-compiling and re-installing if Neovim is already present, unless a rebuild is forced.

## Requirements

* A Debian/Ubuntu-based system.
* `git` installed (usually handled by a base setup or another role like your `apt` role).
* Sudo privileges for installing packages and installing Neovim to a system directory.

## Role Variables

Variables that can be configured are defined in `defaults/main.yml`:

* `neovim_version`: Fixed to `stable`. This variable is kept for structure but the role is hardcoded to build the stable branch.
* `neovim_repo_url`: The URL of the Neovim git repository.
    * Default: `https://github.com/neovim/neovim.git`
* `neovim_build_parent_dir`: The parent directory where the Neovim source code will be cloned. A `neovim_source` subdirectory will be created here.
    * Default: `{{ ansible_env.HOME }}/build` (e.g., `/home/user/build/neovim_source`)
* `neovim_cmake_build_type`: The CMake build type.
    * Default: `RelWithDebInfo`
* `neovim_install_prefix`: The prefix where Neovim will be installed. The binary will typically be at `{{ neovim_install_prefix }}/bin/nvim`.
    * Default: `/usr/local`
* `neovim_force_rebuild`: Set to `true` to force a rebuild and reinstall of the stable branch even if Neovim appears to be installed.
    * Default: `false`

Internal variables for build dependencies are in `vars/main.yml`.

## How it Works

1.  **Check for Existing Installation:** The role first checks if `{{ neovim_install_prefix }}/bin/nvim` exists. If `neovim_force_rebuild` is `false`, many subsequent steps might be skipped.
2.  **Install Dependencies:** Installs packages like `ninja-build`, `gettext`, `cmake`, `curl`, and `build-essential` using `apt`.
3.  **Ensure Build Directory:** Creates the `{{ neovim_build_parent_dir }}/neovim_source` directory.
4.  **Clone/Update Source:** Clones the `stable` branch of the Neovim repository to `{{ neovim_build_parent_dir }}/neovim_source` or updates it if it already exists. A shallow clone (`depth: 1`) is used.
5.  **Build:** Runs `make CMAKE_BUILD_TYPE={{ neovim_cmake_build_type }}` within the source directory.
6.  **Install:** Runs `make install` (with `sudo`) to install Neovim to `{{ neovim_install_prefix }}`.

## Usage

Include this `neovim` role in your main playbook:

```yaml
# local.yml
# ...
  roles:
    # ... other roles
    - role: neovim
      # Optionally override variables:
      # vars:
      #   neovim_force_rebuild: true # To force a rebuild of stable
```
