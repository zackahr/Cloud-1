# Cloud Infrastructure Deployment with Ansible

This project provides an automated deployment solution for a multi-service web application stack using Ansible. It deploys a WordPress site with MariaDB database, Nginx reverse proxy, and phpMyAdmin on multiple virtual machines.

## Project Overview

This Ansible automation project sets up a complete web hosting environment featuring:

- **WordPress**: Content management system with WP-CLI integration
- **MariaDB**: Database server for WordPress data storage
- **Nginx**: Web server with SSL/TLS termination and reverse proxy
- **phpMyAdmin**: Web-based database administration interface

All services are containerized using Docker and orchestrated with Docker Compose.

## Architecture

The deployment consists of:

- **Load Balancing**: Deploys across multiple VMs (defined in inventory.ini)
- **Containerization**: Each service runs in its own Docker container
- **SSL Security**: Self-signed certificates for HTTPS encryption
- **Volume Persistence**: Database and WordPress files are persistently stored
- **Service Discovery**: Internal networking between containers

## Project Structure

```
├── playbook.yml                    # Main Ansible playbook
├── inventory.ini                   # Target hosts configuration
└── rules/                          # Ansible role directory
    ├── tasks/main.yml              # Main deployment tasks
    ├── handlers/main.yml           # Service restart handlers
    ├── templates/data.j2           # Encrypted environment variables
    └── files/                      # Application files
        ├── docker-compose.yml      # Container orchestration
        ├── nginx/                  # Nginx configuration and Dockerfile
        ├── wordpress/              # WordPress setup and Dockerfile
        └── mariadb/               # MariaDB configuration and Dockerfile
```

## Prerequisites

- Ansible installed on control machine
- Target VMs with SSH access configured
- Root privileges on target machines
- SSH key authentication set up

## Configuration

### Environment Variables

The project uses Ansible Vault to encrypt sensitive environment variables in `rules/templates/data.j2`. This file contains:

- Database credentials
- WordPress admin credentials
- WordPress user credentials
- Application configuration

To edit the encrypted file:
```bash
ansible-vault edit rules/templates/data.j2
```

### Inventory Configuration

Update `inventory.ini` with your target hosts:

```ini
[my_vms]
vm1 ansible_host=YOUR_IP_1 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa
vm2 ansible_host=YOUR_IP_2 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa
```

## Deployment

### Running the Playbook

Execute the deployment with:

```bash
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass
```

### What Gets Deployed

The `playbook.yml` executes the following tasks (defined in rules/tasks/main.yml):

1. **System Setup**:
   - Updates and upgrades system packages
   - Installs Docker and Docker Compose

2. **Directory Structure**:
   - Creates `/etc/db` for database volume binding
   - Creates `/etc/files` for WordPress volume binding  
   - Creates `/home/application` for application files

3. **File Deployment**:
   - Deploys encrypted environment variables as `.env`
   - Copies all application files and configurations
   - Triggers Docker Compose deployment via handlers

4. **Container Orchestration** (via rules/handlers/main.yml):
   - Stops existing containers and removes images
   - Builds and starts new containers
   - Runs asynchronously with 10-minute timeout

## Services and Ports

| Service | Internal Port | External Port | Description |
|---------|--------------|---------------|-------------|
| Nginx | 80, 443 | 80, 443 | Web server with SSL |
| WordPress | 9000 | 8080 | PHP-FPM application |
| MariaDB | 3306 | - | Database server |
| phpMyAdmin | 80 | 8081 | Database admin interface |

## Service Details

### Nginx (rules/files/nginx/)
- Debian-based container with SSL certificates
- Redirects HTTP to HTTPS
- Proxies PHP requests to WordPress container
- Configured for domain: `https://domain.name`

### WordPress (rules/files/wordpress/)
- Debian-based with PHP-FPM and WP-CLI
- Automated WordPress installation and configuration
- Creates admin user and additional author user
- Connects to MariaDB for data persistence

### MariaDB (rules/files/mariadb/)
- Debian-based database server
- Automated database and user creation
- Configured for external connections
- Persistent storage via volume binding

## Security Features

- **SSL/TLS Encryption**: Self-signed certificates for HTTPS
- **Ansible Vault**: Encrypted storage of sensitive variables
- **Container Isolation**: Services run in separate containers
- **Database Security**: Dedicated database user with limited privileges

## Accessing the Application

After deployment:

- **WordPress Site**: `https://domain.name` (or your configured domain)
- **phpMyAdmin**: `http://YOUR_SERVER_IP:8081`
- **Direct WordPress**: `http://YOUR_SERVER_IP:8080`

## Maintenance

### Updating the Application

To redeploy with changes:
1. Modify files in the `rules/files` directory
2. Re-run the playbook
3. The handler will automatically rebuild and restart containers

### Managing Secrets

To update environment variables:
```bash
ansible-vault edit rules/templates/data.j2
```

## Troubleshooting

- **Container Logs**: Check logs on target hosts in `/home/application`
- **Service Status**: Verify Docker containers are running with `docker ps`
- **Volume Mounts**: Ensure `/etc/db` and `/etc/files` directories exist and have correct permissions
- **Network Connectivity**: Verify containers can communicate on the `Shared` network

## Development Notes

This project demonstrates:
- Ansible role structure and best practices
- Docker multi-container applications
- Automated WordPress deployment
- SSL certificate generation
- Database initialization automation
- Volume persistence strategies