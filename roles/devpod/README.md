# DevPod Installation Role (`devpod`)

This Ansible role installs the DevPod Command Line Interface (CLI) on a Linux system. It downloads the specified or latest version of the DevPod binary from the official GitHub releases and places it in `/usr/local/bin`.

## Purpose

The primary goals of this role are to:
* Download the DevPod CLI binary for the appropriate architecture.
* Install the DevPod CLI to a standard location in the system PATH (`/usr/local/bin/devpod`).
* Allow specification of a particular DevPod version or install the latest available.
* Ensure the installation is idempotent.

## How it Works

The role performs the following steps:

1.  **Checks for Existing Installation:** Determines if DevPod is already installed at `/usr/local/bin/devpod` and checks its version.
2.  **Determines if Installation/Update is Needed:** Based on whether DevPod is installed and if the requested version (specific or "latest") matches the current version.
3.  **Downloads DevPod Binary:** If installation/update is needed, it downloads the appropriate binary for the system's architecture (amd64 or arm64) from `https://github.com/loft-sh/devpod/releases`.
4.  **Installs DevPod Binary:** Copies the downloaded binary to `/usr/local/bin/devpod` and ensures it has execute permissions. This step requires `become: true` (root privileges).
5.  **Cleans Up:** Removes the temporary downloaded binary.

## Requirements

* `curl` or `wget` (implicitly used by `ansible.builtin.get_url`).
* Internet access to download from GitHub.
* `become` privileges for installing to `/usr/local/bin`.

## Role Variables

Default variables for this role are defined in `roles/devpod/defaults/main.yml`:

* `devpod_version`: The version of DevPod to install (e.g., `"v0.6.15"`). Set to `"latest"` to attempt to download the most recent release.
    * Default: `"latest"`
* `devpod_architecture`: Automatically determined based on `ansible_architecture`. Defaults to `amd64`.
* `devpod_download_url`: Automatically constructed URL for the download.
* `devpod_install_path`: The full path where the DevPod CLI will be installed.
    * Default: `"/usr/local/bin/devpod"`
* `devpod_tmp_download_path`: Temporary path for the downloaded binary.
    * Default: `"/tmp/devpod-{{ devpod_architecture }}"`

These variables can be overridden when the role is called in a playbook.

## Usage

Include this `devpod` role in your main playbook (`local.yml`).

```yaml
# local.yml
# ...
  roles:
    # ... other roles
    - role: devpod
      # Optionally override variables here if needed:
      # vars:
      #   devpod_version: "v0.6.10" # Example for a specific version
```

