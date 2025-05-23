# git Role

This role configures global Git settings for the user on the target system. It ensures that the Git `user.name` and `user.email` are set automatically, which is essential for proper commit authorship.

## Purpose

The primary goal of this role is to:
* Set the global `user.name` in Git configuration.
* Set the global `user.email` in Git configuration.

These settings are crucial for identifying the author of commits when working with Git repositories.

## Configuration

The `user.name` and `user.email` values are directly defined within the `tasks/main.yml` file of this role.

**Current Configuration:**

* **`user.name`**: `MichaelSandilands`
* **`user.email`**: `michael.b.sandilands@gmail.com`

If these values need to be changed or made dynamic (e.g., passed as variables), you would modify `roles/git/tasks/main.yml` or introduce a `vars/main.yml` file within this role.

## Usage

To use this role, include it in your main playbook (`local.yml` in this project) under the `roles` section:

```yaml
# local.yml
# ...
  roles:
    # ... other roles
    - role: git
# ...
```

This role is designed to be idempotent. It will only update the Git configuration if the `user.name` or `user.email` values differ from what's already set, ensuring efficiency and avoiding unnecessary changes.
