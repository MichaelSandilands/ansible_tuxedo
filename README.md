# Personal Tuxedo OS Environment

To run this playbook on a completely fress install of Tuxedo OS:

```bash
# Update apt repository cache and upgrade machine
sudo apt update && sudo apt upgrade -y

# Install ansible
sudo apt install ansible

# Install ansible-galaxy roles
curl -L -o requirements.yml "https://raw.githubusercontent.com/MichaelSandilands/ansible_tuxedo/refs/heads/main/requirements.yml"
ansible-galaxy install -r requirements.yml
rm requirements.yml

# Provision Machine
ansible-pull -U https://github.com/MichaelSandilands/ansible_tuxedo.git -K -J
```
