# Role: docker

Configures the Docker Engine after the packages role has installed it.

## What it does

1. Enables and starts the `docker` systemd service.
2. Adds the target user to the `docker` group (log out / back in, or reboot,
   for the group membership to take effect).

## Variables

| Variable      | Default            | Description                          |
| ------------- | ------------------ | ------------------------------------ |
| `docker_user` | detected sudo user | User to add to the `docker` group.   |

## Requirements

- The `packages` role must have installed `docker-ce` and friends first.
- `become: true` (both tasks are root operations).

## Usage

```yaml
roles:
  - role: packages
  - role: docker
    vars:
      docker_user: "{{ target_user }}"
```
