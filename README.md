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

Install using uv (fast Python package manager):

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Python 3 and curl
sudo apt install -y python3 git curl

# Install uv (fast Python package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc

# Install Ansible
uv tool install ansible

# Install required Ansible collections
ansible-galaxy collection install -r requirements.yml
```

### Alternative Installation Methods

#### Using uv with virtual environment (alternative approach)
```bash
sudo apt install -y python3 git curl
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc
uv venv ansible-env
source ansible-env/bin/activate
uv pip install ansible
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

## Testing the playbook

You can test the playbook syntax without making changes:

```bash
# Check syntax
ansible-playbook --syntax-check local.yml

# Dry run (check mode)
sudo ansible-playbook --check local.yml

# Run specific tasks only
sudo ansible-playbook local.yml --list-tags
sudo ansible-playbook local.yml --tags facts,software
```

## Modern Features

This updated version includes:

- **Modern Ansible syntax**: Uses current YAML formatting and module parameters
- **Collection dependencies**: Automatically installs required Ansible collections
- **Improved error handling**: Better validation and failure messages
- **Handlers**: Automatic service restarts and system reboots when needed
- **Modular configuration**: Defaults, variables, and tasks properly separated
- **Better tagging**: Fine-grained control over which parts run
- **Security improvements**: Modern package management and SSH hardening

## Project Structure

```
.
├── ansible.cfg              # Ansible configuration
├── requirements.yml         # Required collections
├── local.yml               # Main playbook
├── hosts                   # Inventory file
└── roles/
    └── common/
        ├── defaults/       # Default variables
        ├── vars/          # Role variables  
        ├── tasks/         # Task files
        ├── handlers/      # Event handlers
        └── meta/         # Role metadata
```

## After running the playbook

This playbook purposely requires a few manual steps _after_ running the playbook
to ensure that it does not lock you out of your Raspberry Pi.

1. While logged in as pi, `sudo passwd <username>` (where username is the user you created)
   to set a password for that user. This is _not_ required to log in as that
   user with SSH keys, but it _is_ required to `sudo` as that user. You may also choose to
   set a password for the pi and/or root users.

2. `sudo usermod --lock pi` to ensure that the default user is completely disabled.
   **Warning**: Only do this after confirming you can log in with your new user account!

## Troubleshooting

### Common Issues

- **MAC address not found**: Add your Pi's MAC address to `roles/common/vars/main.yml`
- **Permission denied**: Make sure to run with `sudo` for system configuration
- **Collections not found**: Run `ansible-galaxy collection install -r requirements.yml`
- **SSH connection issues**: Verify the pi user SSH restrictions in `/etc/ssh/sshd_config`

### Debug mode

Run in verbose mode for troubleshooting:

```bash
sudo ansible-playbook local.yml -v     # verbose
sudo ansible-playbook local.yml -vv    # more verbose
sudo ansible-playbook local.yml -vvv   # debug level
```

### Validation

Use the validation playbook to check system compatibility:

```bash
ansible-playbook validate.yml
```

## Acknowledgment

I stole a lot of knowledge from https://github.com/giuaig/ansible-raspi-config/.

Original work by Glenn K. Lockwood, October 2018.  
Modernized for current Ansible best practices.
