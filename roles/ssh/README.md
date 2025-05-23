# ssh Role

This role is responsible for setting up SSH keys and configuring SSH for Git interactions, specifically with GitHub, on the target system. It ensures that the necessary SSH credentials are in place and correctly configured for secure communication.

## Purpose

The main goal of this role is to:
* Create and secure the `~/.ssh` directory.
* Copy the SSH private and public keys (generated for `ansible_tuxedo`) to the user's `~/.ssh` directory.
* Set the correct, restrictive permissions for the SSH keys and directory.
* Add a configuration entry to `~/.ssh/config` for `github.com` to use the specified SSH key.

## Included Files

This role utilizes the following files from its `files/` directory:

* **`ansible_tuxedo`**: This is your **encrypted** private SSH key. It must be decrypted by Ansible Vault during the playbook run.
* **`ansible_tuxedo.pub`**: This is your unencrypted public SSH key.

## Usage

To use this role, include it in your main playbook (`local.yml` in this project) under the `roles` section:

```yaml
# local.yml
# ...
  roles:
    - role: ssh
    # ... other roles
```

This role is designed to be idempotent, ensuring that SSH configurations are only applied or updated as necessary.

## Sensitive Data Handling (Ansible Vault)

The `ansible_tuxedo` private key is stored encrypted within this role's `files/` directory using Ansible Vault. This is a critical security measure to prevent sensitive data from being exposed in your Git repository.

### Running Playbooks with Vault Passphrase

When running your Ansible playbook, you **must** provide the Vault passphrase to decrypt the private key.

#### For `ansible-playbook`:

Use the --ask-vault-pass (or its shorthand -J) argument:

```bash
ansible-playbook local.yml --ask-vault-pass
# Or, using the shorthand:
ansible-playbook local.yml -J
```


#### For `ansible-pull`:

```bash
ansible-pull -U https://github.com/MichaelSandilands/ansible_tuxedo.git --ask-vault-pass
# Or, using the shorthand:
ansible-pull -U https://github.com/MichaelSandilands/ansible_tuxedo.git -J
```
