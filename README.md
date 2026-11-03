# EMMS Ansible Setup

This repository contains Ansible playbooks to provision and deploy the EMMS application (a PHP/Laravel application running on Laravel Octane with Swoole, Supervisor, and Nginx reverse proxy).

## Architecture
- **PHP 8.2** with Swoole extension (replaces PHP-FPM).
- **Supervisor Process Management** with separate configuration files per component:
  - `emms-octane`: Laravel Octane application server running on `127.0.0.1:8000`.
  - `emms-worker`: Laravel Queue Worker (`php artisan queue:work --sleep=3 --tries=3 --max-time=3600`).
  - `emms-scheduler`: Laravel Task Scheduler (`php artisan schedule:work`).
- **Nginx**: High-performance reverse proxy with WebSocket support and direct static file serving.
- **App User**: `www-data` (no login shell) with home directory `/home/www-data`.
- **Application Path**: `/home/www-data/emms`.

## Prerequisites
- Ansible installed on your control node.
- SSH access to the target servers.
- Proper inventory configuration (`inventory.ini`).

## Playbooks

The setup is orchestrated by the master `playbook.yml` or can be run via individual playbooks in `playbooks/`:

### `playbook.yml`
Master playbook linking and executing all sub-playbooks in order for a full-stack deployment.

### Sub-playbooks (in `playbooks/`)
- `app-setup.yml`: Installs PHP, Swoole, Supervisor, Composer, Git, creates `www-data` user with no login shell (`/home/www-data`), clones `snbhattacharjya/emms.git` to `/home/www-data/emms`, configures Octane with Swoole, sets up separate Supervisor configuration files for Octane, Queue Worker, and Scheduler, and starts all services.
- `nginx-setup.yml`: Configures Nginx reverse proxy for Octane, manages firewall (UFW/Firewalld) and SELinux policies.
- `update-codebase.yml`: Pulls latest code from Git, runs composer optimization, restarts queue workers, and reloads Laravel Octane.

## Supervisor Commands

Each component has its own dedicated supervisor configuration file:
- `emms-octane.conf`: Octane server
- `emms-worker.conf`: Queue workers
- `emms-scheduler.conf`: Task scheduler

Useful commands:
```bash
# Check status of all components
supervisorctl status

# Restart individual components
supervisorctl restart emms-octane:*
supervisorctl restart emms-worker:*
supervisorctl restart emms-scheduler:*

# Restart all at once
supervisorctl restart emms-octane:* emms-worker:* emms-scheduler:*
```

## Usage

To run the complete setup:
```bash
ansible-playbook -i inventory.ini playbook.yml --ask-become-pass
```

You can also run individual playbooks:
```bash
ansible-playbook -i inventory.ini playbooks/app-setup.yml --ask-become-pass
ansible-playbook -i inventory.ini playbooks/nginx-setup.yml --ask-become-pass
ansible-playbook -i inventory.ini playbooks/update-codebase.yml --ask-become-pass
```
