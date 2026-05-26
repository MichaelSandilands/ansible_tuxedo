# Role: packages

Installs every system and development package in a single idempotent pass
(`state: present`), refreshing the dnf cache first. Ongoing patching is the
job of the `auto_updates` role, not this one.

## Variables

| Variable             | Default | Description                                         |
| -------------------- | ------- | --------------------------------------------------- |
| `packages_install_r` | `false` | Install the optional R / data-science stack.        |
| `packages_base`      | list    | Packages installed on every run.                    |
| `packages_r_system`  | list    | R core + build/geospatial deps (when R is enabled). |
| `packages_r_user`    | list    | R packages installed into the user library.         |

The package set installed is:

```
packages_base + (packages_r_system if packages_install_r else [])
```

This replaces the large block of commented-out package names the playbook
used to carry: the R stack is now real, version-controlled data behind a
flag instead of dead comments.

## Enabling the R stack

```bash
ansible-pull -U "<repo>" -K -e packages_install_r=true
```

When enabled, the role also creates `~/R/library` and installs
`packages_r_user` into it with `Rscript`. The R / RStudio packages come from
the `iucar/rstudio` COPR enabled by the `repositories` role.

## Requirements

- Fedora, `become: true`.
- The `repositories` role must run first.
- `target_user` / `target_home` are provided by the playbook (used only for
  the optional R user library).

## Usage

```yaml
roles:
  - role: repositories
  - role: packages
```
