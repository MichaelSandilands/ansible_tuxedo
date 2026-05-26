# Role: auto_updates

Configures `dnf5-automatic` for unattended downloading and installing of
updates. The configuration is rendered from a Jinja2 template driven by role
variables, so behaviour can be changed without editing a static file, and is
written to both the legacy and DNF5 config paths.

## ⚠️ Note for this machine (Nvidia / Tuxedo kernel)

This workstation installs `akmod-nvidia` and runs a Tuxedo kernel. With the
default `auto_updates_upgrade_type: default`, an unattended **kernel** update
combined with `reboot = never` leaves a running-kernel / installed-kernel
mismatch until the next manual reboot, during which `akmod` has to rebuild the
Nvidia module. If you would rather limit unattended changes to security
patches, set:

```yaml
auto_updates_upgrade_type: security
```

The default is kept as `default` so the role does not silently change the
behaviour the machine had before — but this is the variable to reach for.

## Variables

| Variable                              | Default   | Purpose                                            |
| ------------------------------------- | --------- | -------------------------------------------------- |
| `auto_updates_upgrade_type`           | `default` | `default` (all updates) or `security`.             |
| `auto_updates_random_sleep`           | `0`       | `0` runs immediately when the timer fires.         |
| `auto_updates_network_online_timeout` | `60`      | Seconds to wait for network (`0` skips the check). |
| `auto_updates_download_updates`       | `yes`     | Download RPMs to the cache.                        |
| `auto_updates_apply_updates`          | `yes`     | Actually install (Fedora's default is `no`).       |
| `auto_updates_reboot`                 | `never`   | Reboot policy after applying updates.              |
| `auto_updates_emit_via`               | `stdio`   | Send results to the system journal.                |
| `auto_updates_debuglevel`             | `1`       | Log detail level.                                  |

## What it does

1. Installs `dnf5-plugin-automatic`.
2. Renders `automatic.conf.j2` to `/etc/dnf/automatic.conf` and
   `/etc/dnf/dnf5-plugins/automatic.conf`.
3. Enables and starts `dnf5-automatic.timer`, restarting it via a handler if
   the config changes.

## Requirements

- Fedora, `become: true`.

## Usage

```yaml
roles:
  - role: auto_updates
    # vars:
    #   auto_updates_upgrade_type: security
```

View the last run with:

```bash
journalctl -u dnf5-automatic.service
```
