# Ansible DevOps Project

This repository contains Ansible playbooks and roles for configuring servers and deploying Docker containers, including an NGINX web server, Node Exporter, LEMP, etc for RedHat & Debian based linux distributions.

## Prerequisites

### Control Node Setup

1. **Create and activate a Python virtual environment**:
   It is recommended to run Ansible from within an isolated Python virtual environment.
   ```bash
   # Create the virtual environment
   python3 -m venv ansible-venv
   
   # Activate the virtual environment
   source ansible-venv/bin/activate
   
   # Install ansible and its dependencies
   pip install ansible
   ```

### Target Machine Setup

Before running the playbooks, you must ensure the following prerequisites are met on your target machines:

1. **Create the `ansible` user and grant sudo access**:
   The playbooks are designed to operate using a dedicated `ansible` user on the target machines.
   ```bash
   sudo useradd -m -s /bin/bash ansible
   sudo usermod -aG wheel ansible (for centos and other rpm based distros)
   sudo usermod -aG sudo ansible (for debian and other deb based distros)
   ```

2. **Configure SSH key-based authentication**:
   Generate an SSH key pair on your Ansible control node (if you haven't already) and copy the public key to the target machine for the `ansible` user.
   ```bash
   ssh-keygen -t ed25519 -C "ansible"
   ssh-copy-id ansible@<target_ip>
   # <or>
   #copy the public key and add it to the target machine '.ssh/authorized_keys' file for the 'ansible' user.
   ```

3. **Configure `ansible` user for passwordless sudo access**:
   The `ansible` user requires passwordless sudo privileges to perform system-level tasks (such as installing packages and creating directories).
   ```bash
   echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
   ```

## Guide for Running Playbooks

The main entry point for deploying these services is the `docker-playbook.yaml` playbook. It includes several roles that are logically separated and can be run independently using tags.

### Roles and Tags

- `common`: Basic system configurations, updating package cache and upgrading the system, creating the additional sysadmin user and group (Tag: `common`)
- `docker`: Installs and configures Docker and Docker Compose (Tag: `docker`)
- `nodeExporter`: Deploys the Prometheus Node Exporter for system metrics (Tag: `nodeExporter`)
- `nginx`: Deploy nginx docker container using docker-compose 
- `nginx-fpm`: LEMP stack + Static site deployments for Debian & RPM base OSes.
- `php-fpm`: Install php and configure php-fpm pools

### Execution Commands

**1. Run the entire playbook:**
This will execute all roles in sequence on your target hosts.
```bash
ansible-playbook docker-playbook.yaml -e "@vars.yaml"
# inventory and remote user are already defined in ansible.cfg
```

**2. Run specific roles using tags:**
If you only want to execute a specific component, use the `--tags` argument:

- To run only the **NGINX** role:
  ```bash
  ansible-playbook docker-playbook.yaml  --tags "nginx"  -e "@vars.yaml"
  ```
- To run only the **Docker** installation:
  ```bash
  ansible-playbook docker-playbook.yaml  --tags "docker"  -e "@vars.yaml"
  ```

### Customization
Variables are defined in the `defaults/main.yml` within each role (e.g., `roles/nginx/defaults/main.yml`). You can override them inside `docker-playbook.yaml or var-file vars.yaml with -e option` (as demonstrated with the `docker_users` variable) or by using `host_vars` and `group_vars` in your inventory.
