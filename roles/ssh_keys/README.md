# Role: ssh_keys

Idempotent role for managing SSH keys, SSH config, and the global git
identity. Private keys are encrypted with Ansible Vault so they can be safely
committed to a Git repository. The role loops over every entry in
`ssh_keys_pairs`, so adding a key is just adding a list item.

## Directory Structure

```
roles/ssh_keys/
├── defaults/
│   └── main.yml          # Default (role-prefixed) variables
├── files/
│   ├── config            # SSH config file
│   ├── github_key        # Vault-encrypted private key
│   └── github_key.pub    # Public key
└── tasks/
    └── main.yml          # Role tasks
```

## Variables

All variables are prefixed with the role name (`ssh_keys_`) per Ansible
convention.

| Variable                  | Description                           | Default    |
| ------------------------- | ------------------------------------- | ---------- |
| `ssh_keys_user`           | Target username                       | detected   |
| `ssh_keys_git_user_name`  | Git global `user.name`                | `""`       |
| `ssh_keys_git_user_email` | Git global `user.email`               | `""`       |
| `ssh_keys_pairs`          | List of key pairs to deploy           | `[]`       |
| `ssh_keys_config_files`   | List of config files to deploy        | `[config]` |
| `ssh_keys_git_signing`    | Optional list of git signing settings | undefined  |

Git identity defaults are empty on purpose: the values live once in the
playbook (single source of truth) and the tasks apply them only when set.

## Usage

```yaml
roles:
  - role: ssh_keys
    vars:
      ssh_keys_user: "{{ target_user }}"
      ssh_keys_git_user_name: "Michael"
      ssh_keys_git_user_email: "michael@example.com"
      ssh_keys_pairs:
        - name: github_key
          public: github_key.pub
          private: github_key
```

## What Gets Deployed

| File                 | Permissions | Notes                             |
| -------------------- | ----------- | --------------------------------- |
| `~/.ssh/`            | `0700`      | Directory                         |
| `~/.ssh/config`      | `0600`      | SSH host configuration            |
| `~/.ssh/*.pub`       | `0644`      | Public keys                       |
| `~/.ssh/*` (private) | `0600`      | Private keys (owner-only)         |
| `~/.gitconfig`       | n/a         | Managed via the git_config module |

When Ansible encounters a vault-encrypted file in `src`, it decrypts it in
memory and writes the plaintext to `dest`. The private-key task sets
`no_log: true` so key material never appears in terminal output, and
`loop_control.label` keeps the loop output to filenames only.

## Adding a New SSH Key

### 1. Generate the key pair

```bash
ssh-keygen -t ed25519 -C "michael@example.com" -f ~/.ssh/id_ed25519_ec2
```

### 2. Register the public key with the service

GitHub: Settings → SSH and GPG keys → New SSH key. For another server, append
the `.pub` to its `~/.ssh/authorized_keys`.

### 3. Copy the public key into the role (plaintext is fine)

```bash
cp ~/.ssh/id_ed25519_ec2.pub roles/ssh_keys/files/
```

### 4. Encrypt the private key with Ansible Vault

```bash
ansible-vault encrypt ~/.ssh/id_ed25519_ec2 \
  --output roles/ssh_keys/files/id_ed25519_ec2

head -1 roles/ssh_keys/files/id_ed25519_ec2   # -> $ANSIBLE_VAULT;1.1;AES256
```

### 5. Add a host block to `roles/ssh_keys/files/config`

```
Host ec2
    HostName your-ec2-host
    User ec2-user
    IdentityFile ~/.ssh/id_ed25519_ec2
    IdentitiesOnly yes
```

### 6. Add the pair to `ssh_keys_pairs` in the playbook

```yaml
ssh_keys_pairs:
  - name: github_key
    public: github_key.pub
    private: github_key
  - name: id_ed25519_ec2
    public: id_ed25519_ec2.pub
    private: id_ed25519_ec2
```

### 7. Commit the encrypted key and provision

```bash
git add roles/ssh_keys/files/id_ed25519_ec2 \
        roles/ssh_keys/files/id_ed25519_ec2.pub \
        roles/ssh_keys/files/config
git commit -m "Add EC2 SSH key"

ansible-pull -U "https://github.com/MichaelSandilands/ansible_tuxedo.git" -K
```

## Managing Existing Keys

```bash
# View without writing plaintext to disk
ansible-vault view roles/ssh_keys/files/github_key

# Rotate the vault password on a file
ansible-vault rekey roles/ssh_keys/files/github_key
```

## Commit Signing (optional)

```yaml
ssh_keys_git_signing:
  - { key: gpg.format, value: ssh }
  - { key: user.signingKey, value: "~/.ssh/github_key.pub" }
  - { key: commit.gpgSign, value: "true" }
  - { key: tag.gpgSign, value: "true" }
```

## Security Notes

- **Never commit unencrypted private keys.** Always `ansible-vault encrypt`
  before copying into `files/`.
- **Never commit a `.vault_pass` file** (the repo `.gitignore` covers it).
- Vault-encrypted files use AES-256 and are safe in a public repo.
- Public keys are public by design and committed in plaintext.
- All keys can share one vault password so a single prompt decrypts them all;
  use `ansible-vault rekey` on every file if you rotate it.
