# Personal Tuxedo OS Environment

This playbook is my personal setup for Ubuntu 24.04 based distributions. Specifically, my Tuxedo OS distribution.

##  To run this playbook on a completely fresh install of Tuxedo OS:

```bash
# Update apt repository cache and upgrade machine
sudo apt update && sudo apt upgrade -y

# Install ansible
sudo apt install ansible -y

# Install ansible-galaxy roles
curl -L -o temp_requirements.yml "https://raw.githubusercontent.com/MichaelSandilands/ansible_tuxedo/refs/heads/main/requirements.yml"
ansible-galaxy install -r temp_requirements.yml
rm temp_requirements.yml

# Provision Machine
ansible-pull -U "https://github.com/MichaelSandilands/ansible_tuxedo.git" -K
# The ansible.cfg file will ensure the user is prompted for a vault password. 

# Reboot after Provision
# reboot
```

### `ansible-galaxy install`

Install ansible galaxy roles on the local system.

- `-r`: A file containing a list of roles to be installed.

### `ansible-pull`

Pulls playbooks from a version control system and executes them on target host.

- `-U`: URL of the playbook repository
- `-K`: ask for privilege escalation password

The ansible.cfg file will ensure the user is prompted for a vault password. 

## Post Install Operations

### Add Tmux Plugins

Enter tmux

```bash
tmux
```
Press prefix + I (capital i, as in Install) to fetch the plugin.

### Initialise Conda

```{bash}
conda init
```

After restarting the terminal you can then activate the datascience_stack conda environment.

```{bash}
conda activate datascience_stack
```

### Neovim & Molten Issues

If you encounter issues with the molten or quarto plugins run the this command in neovim and restart nvim `:UpdateRemotePlugins`

If you encounter kernel issues run jupyter kernel --kernel=python3 in the bash terminal.
