# Role: repositories

Adds the third-party package sources the workstation needs and imports their
GPG keys. Extracted from the playbook `pre_tasks` so all package origins live
in one auditable place.

## What it configures

- **Tuxedo Computers** repo + GPG key.
- **Docker CE** repo + GPG key.
- **RPM Fusion** free and non-free release RPMs, then imports the bundled
  GPG keys from disk. (The release RPMs are installed with
  `disable_gpg_check: true` as a one-time bootstrap, since the keys are not
  present until those RPMs are installed.)
- **fedora-cisco-openh264** repo enabled via `ini_file`.
- **COPR** repos: `atim/starship`, `iucar/rstudio`, `wez/wezterm`.

## Variables

None. The role reads the `ansible_distribution_major_version` fact directly to
build version-specific URLs, so it is self-contained.

## Requirements

- Fedora, `become: true`.
- `community.general` collection (bundled with Fedora's `ansible` package) for
  the `ini_file` and `copr` modules.

## Usage

```yaml
roles:
  - role: repositories
```

Run this before the `packages` role so the new sources are available when
packages are installed.
