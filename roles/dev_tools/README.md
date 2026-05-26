# Role: dev_tools

Installs developer tools distributed as vendor `curl | sh` scripts: Astral's
`uv` and `Ollama`.

## Design notes

- Both shell tasks use `set -o pipefail` (with `executable: /bin/bash`) so a
  failed `curl` is **not** masked by a successful `sh`.
- `uv` is installed/updated **as the target user** (`become_user`), into
  `~/.local/bin`.
- `Ollama` is installed **as root, intentionally** — its installer registers
  a systemd service and a dedicated `ollama` system user. This is an explicit
  `become: true` (no `become_user`) so the privilege level is deliberate
  rather than accidentally inherited.
- `changed_when` is derived from each installer's output as best it can be;
  these vendor scripts do not map cleanly onto Ansible's change model, so the
  detection is heuristic by nature.

## Variables

| Variable         | Default            | Description                         |
| ---------------- | ------------------ | ----------------------------------- |
| `dev_tools_user` | detected sudo user | User that owns the `uv` install.    |
| `dev_tools_home` | `/home/<user>`     | Home dir (used for the `uv` path).  |

## Usage

```yaml
roles:
  - role: dev_tools
    vars:
      dev_tools_user: "{{ target_user }}"
      dev_tools_home: "{{ target_home }}"
```
