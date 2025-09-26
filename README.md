# Raspberry Pi Ansible

Glenn K. Lockwood, October 2018  
Updated for modern Ansible best practices

## Introduction

This is an Ansible configuration that configures a fresh Raspberry Pi OS installation
on Raspberry Pi. It is intended to be run in local (pull) mode, where ansible
is running on the same Raspberry Pi to be configured.

## Bootstrapping on Raspberry Pi OS

You will need Ansible installed on the Raspberry Pi being configured. This
playbook requires Ansible 2.9 or newer. 

### Modern Installation Method (Recommended)

Install using pip3 and collections:

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Python 3 and pip
sudo apt install -y python3-pip git

# Install Ansible
pip3 install --user ansible

# Add pip user bin to PATH if not already present
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Install required Ansible collections
ansible-galaxy collection install -r requirements.yml
```

### Alternative Installation Methods

#### Using pipx (preferred for isolation)
```bash
sudo apt install -y python3-pip git pipx
pipx install ansible-core
pipx inject ansible-core ansible
ansible-galaxy collection install -r requirements.yml
```

#### Using system packages (may be outdated)
```bash
sudo apt install -y ansible git
# Note: This may install an older version of Ansible
```

## Configuration

The `macaddrs` structure in _roles/common/vars/main.yml_ maps the MAC address of
a Raspberry Pi to its intended configuration state. Add your Raspberry Pi's MAC
address to that structure and set its configuration accordingly.

## Running the playbook

Then run the playbook:

```bash
sudo ansible-playbook local.yml
```

You can also run specific parts using tags:

```bash
# Run only software installation
sudo ansible-playbook local.yml --tags software

# Run only user configuration  
sudo ansible-playbook local.yml --tags users

# Run only firewall configuration
sudo ansible-playbook local.yml --tags firewall

# Skip firewall configuration
sudo ansible-playbook local.yml --skip-tags firewall
```

The playbook will self-discover its settings, then idempotently configure the
Raspberry Pi.

## After running the playbook

This playbook purposely requires a few manual steps _after_ running the playbook
to ensure that it does not lock you out of your Raspberry Pi.

1. While logged in as pi, `sudo passwd glock` (or whatever username you created)
   to set a password for that user.  This is _not_ required to log in as that
   user, but it _is_ required to `sudo` as that user.  You may also choose to
   set a password for the pi and/or root users.

2. `usermod --lock pi` to ensure that the default user is completely disabled.

## Acknowledgment

I stole a lot of knowledge from https://github.com/giuaig/ansible-raspi-config/.
