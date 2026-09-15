# EES Ansible Setup

This repository contains Ansible playbooks to provision and deploy the EES application (a PHP/Laravel application).

## Prerequisites
- Ansible installed on your control node.
- SSH access to the target servers.
- Proper inventory configuration.

## Playbooks

The setup is divided into several sub-playbooks, all orchestrated by the master `playbook.yml`.

### `playbook.yml`
The master playbook that runs all necessary sub-playbooks in the correct order for a complete full-stack deployment.

### Sub-playbooks (in `playbooks/`)
- `app-setup.yml`: Configures the application environment, including installing PHP, Composer, Git, Nginx, setting up the application user, SSH keys, cloning the repository, and setting up Laravel dependencies.
- `nginx-setup.yml`: Configures Nginx for the application.
- `update-codebase.yml`: Handles updating the application codebase.

## Usage

To run the complete setup:
```bash
ansible-playbook -i <inventory-file> playbook.yml --ask-become-pass
```

You can also run individual playbooks if you only need to perform specific tasks:
```bash
ansible-playbook -i <inventory-file> playbooks/app-setup.yml --ask-become-pass
```

*Note: Ensure your inventory variables and target hosts are correctly defined before running the playbooks.*
