# Personal Tuxedo OS Environment

To run this playbook on a completely fresh install of Tuxedo OS:

```bash
# Update apt repository cache and upgrade machine
sudo apt update && sudo apt upgrade -y

# Install ansible
sudo apt install ansible -y

# Install ansible-galaxy roles
curl -L -o requirements.yml "https://raw.githubusercontent.com/MichaelSandilands/ansible_tuxedo/refs/heads/main/requirements.yml"
ansible-galaxy install -r requirements.yml
rm requirements.yml

# Provision Machine
ansible-pull -U "https://github.com/MichaelSandilands/ansible_tuxedo.git" -K
# The ansible.cfg file will ensure the user is prompted for a vault password. 
```

## `ansible-galaxy install`

Install ansible galaxy roles on the local system.

- `-r`: A file containing a list of roles to be installed.

## `ansible-pull`

Pulls playbooks from a version control system and executes them on target host.

- `-U`: URL of the playbook repository
- `-K`: ask for privilege escalation password

The ansible.cfg file will ensure the user is prompted for a vault password. 
