# Dollar Universe 6.10 - Automated Deployment

Simple automation for deploying Dollar Universe Application Server (DUAS) 6.10 on Oracle Linux using GitLab CI/CD and Ansible.

## Overview

This project automates:

- Download DUAS installer from JFrog Artifactory (using Vault credentials)
- Silent installation on Oracle Linux
- UVMS registration
- Auto-start configuration via systemd

## Prerequisites

### Infrastructure

- Oracle Linux 7.x or 8.x target server
- GitLab instance with CI/CD runners
- HashiCorp Vault server
- JFrog Artifactory with DUAS installer

### Access

- SSH access to target server
- Vault access for credentials
- UVMS server connectivity

## Quick Setup

### 1. Configure GitLab CI Variables

In GitLab project: **Settings → CI/CD → Variables**

| Variable      | Value                     | Type     |
| ------------- | ------------------------- | -------- |
| `VAULT_ADDR`  | https://vault.example.com | Variable |
| `TARGET_HOST` | target-server.example.com | Variable |

**Note**: All credentials (Artifactory credentials and SSH private key) are stored in Vault and will be fetched automatically by Ansible using the `community.hashi_vault` collection.

### 2. Update Configuration

Edit `ansible/inventory.ini`:

```ini
[duas_servers]
your-server.example.com ansible_user=root
```

Edit `ansible/roles/duas_install/defaults/main.yml`:

```yaml
duas_company: "MYCOMPANY"
duas_node: "NODE01"
duas_area: "APP" # APP, INT, or SIM
duas_install_dir: "/var/opt/AUTOMIC/DUAS" # Official default
uvms_hostname: "uvms-server.example.com"
artifactory_url: "https://your-artifactory.jfrog.io/artifactory"
```

### 3. Deploy

```bash
git add .
git commit -m "Configure DUAS deployment"
git push origin main
```

In GitLab: **CI/CD → Pipelines** → Click play button on `deploy_duas`

## Vault Secrets Structure

The following secrets must be stored in Vault:

**Artifactory credentials** (`secret/artifactory/duas`):

```json
{
  "username": "artifactory-user",
  "password": "artifactory-password",
  "url": "https://your-artifactory.jfrog.io/artifactory",
  "repository": "duas-installers"
}
```

**SSH private key** (`secret/ssh/duas`):

```json
{
  "private_key": "-----BEGIN RSA PRIVATE KEY-----\n...\n-----END RSA PRIVATE KEY-----"
}
```

**Note**: Ansible uses the `community.hashi_vault` collection to fetch these secrets. Install it with:

```bash
ansible-galaxy collection install community.hashi_vault
```

## Project Structure

```
duas_deploy/
├── .gitlab-ci.yml              # GitLab CI/CD pipeline
├── README.md                   # This file
└── ansible/
    ├── ansible.cfg            # Ansible configuration
    ├── inventory.ini          # Target servers
    ├── playbooks/
    │   └── duas.yml          # Main deployment playbook
    └── roles/
        └── duas_install/
            ├── defaults/main.yml      # Variables
            ├── tasks/
            │   ├── main.yml          # Task orchestration
            │   ├── prerequisites.yml  # System setup
            │   ├── install.yml        # DUAS installation
            │   └── configure.yml      # Post-install config
            ├── handlers/main.yml      # Systemd handlers
            └── templates/
                ├── install.file.j2    # Silent install response file
                └── duas.service.j2    # Systemd service
```

## Service Management

After deployment, manage DUAS on the target server:

```bash
# Using systemd
systemctl start duas_MYCOMPANY.service
systemctl stop duas_MYCOMPANY.service
systemctl restart duas_MYCOMPANY.service
systemctl status duas_MYCOMPANY.service

# Using DUAS commands directly
su - duas
source /var/opt/AUTOMIC/DUAS/MYCOMPANY_<hostname>/unienv.ksh
unidlt start
unidlt stop
unidlt status
```

## Troubleshooting

### Pipeline fails

- Check Vault connectivity: `curl -s $VAULT_ADDR/v1/sys/health`
- Verify credentials in Vault: `vault kv get secret/artifactory/duas`
- Check SSH access: `ssh root@target-server`

### Installation fails

```bash
# On target server - check installation logs
tail -f /tmp/duas_installer/*.log
journalctl -xe
df -h /var/opt/AUTOMIC
```

### Service won't start

```bash
systemctl status duas_MYCOMPANY.service -l
journalctl -u duas_MYCOMPANY.service -n 100
ps -ef | grep duas
# Check DUAS logs
tail -f /var/opt/AUTOMIC/DUAS/MYCOMPANY_*/log/*.log
```

### UVMS registration fails

```bash
telnet uvms-server.example.com 4184
# Manual registration
su - duas
source /u00/app/automic/MYCOMPANY/NODE01/bin/unienv.ksh
unisetup -register -uvms uvms-server.example.com -port 4184
```

## Configuration Variables

Key variables in `ansible/roles/duas_install/defaults/main.yml`:

```yaml
duas_company: "MYCOMPANY" # Company code
duas_node: "NODE01" # Node name
duas_area: "APP" # Area designation
duas_install_dir: "/u00/app/automic" # Installation path
uvms_hostname: "uvms-server.example.com"
uvms_port: "4184"
artifactory_url: "https://your-artifactory.jfrog.io/artifactory"
artifactory_repo: "duas-installers"
duas_installer_archive: "Dollar.Universe_Application.Server_linux.64_6_10_111+build.0039.tar.gz"
```

## Support

- Check logs: `/tmp/duas_installer/*.log`, `journalctl -u duas_*`
- Review DUAS documentation: https://docs.automic.com/
- Contact DevOps team

---

**Status:** Production Ready  
**Version:** 1.0.0

Complete DevOps automation solution for deploying Dollar Universe Application Server (DUAS) 6.10 on Oracle Linux using GitLab CI/CD and Ansible.

## 🎯 Features

- ✅ Automated download from JFrog Artifactory using Vault-stored credentials
- ✅ Silent/unattended installation on Oracle Linux
- ✅ Automatic UVMS registration
- ✅ Systemd service configuration for auto-start on boot
- ✅ Service management (start/stop/restart/status)
- ✅ Complete prerequisite setup (packages, users, kernel parameters)
- ✅ GitLab CI/CD pipeline integration

## 📋 Prerequisites

### Infrastructure

- **Target Server:** Oracle Linux 7.x or 8.x
- **GitLab:** GitLab instance with CI/CD runners
- **Vault:** HashiCorp Vault server
- **Artifactory:** JFrog Artifactory with DUAS installer

### Required Access

- SSH access to target server
- Vault access for credential retrieval
- Artifactory credentials
- UVMS server details

## 🏗️ Project Structure

```
duas_deploy/
├── .gitlab-ci.yml              # GitLab CI/CD pipeline
├── README.md                    # This file
├── VAULT_SETUP.md              # Vault configuration guide
├── Dollar.Universe_Application.md
└── ansible/
    ├── ansible.cfg             # Ansible configuration
    ├── inventory.ini           # Target servers inventory
    ├── playbooks/
    │   ├── duas.yml            # Main deployment playbook
    │   └── manage_duas.yml     # Service management playbook
    └── roles/
        └── duas_install/
            ├── defaults/
            │   └── main.yml    # Default variables
            ├── files/
            ├── handlers/
            │   └── main.yml    # Service handlers
            ├── tasks/
            │   ├── main.yml            # Main task orchestration
            │   ├── prerequisites.yml   # System preparation
            │   ├── install.yml         # DUAS installation
            │   └── configure.yml       # Post-install configuration
            └── templates/
                ├── duas.service.j2     # Systemd service
                ├── unattended.txt.j2   # Silent install config
                ├── kshrc.j2            # User environment
                ├── duas_start.j2       # Start script
                ├── duas_stop.j2        # Stop script
                ├── duas_restart.j2     # Restart script
                └── duas_status.j2      # Status script
```

## 🚀 Quick Start

### 1. Configure Vault

Follow the instructions in [VAULT_SETUP.md](VAULT_SETUP.md) to configure Vault secrets.

```bash
vault kv put secret/artifactory/duas \
  username="artifactory-user" \
  password="your-password" \
  url="https://your-artifactory.jfrog.io/artifactory" \
  repository="duas-installers" \
  installer_file="Dollar.Universe_Application.Server_linux.64_6_10_111+build.0039.tar.gz"
```

### 2. Configure GitLab CI Variables

In your GitLab project, navigate to **Settings > CI/CD > Variables** and add:

| Variable          | Value                     | Type     |
| ----------------- | ------------------------- | -------- |
| `VAULT_ADDR`      | https://vault.example.com | Variable |
| `SSH_PRIVATE_KEY` | Your SSH private key      | File     |
| `TARGET_HOST`     | target-server.example.com | Variable |

### 3. Update Inventory

Edit `ansible/inventory.ini`:

```ini
[duas_servers]
target-server.example.com ansible_user=root

[duas_servers:vars]
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
```

### 4. Customize Variables

Edit `ansible/roles/duas_install/defaults/main.yml`:

```yaml
duas_company: "MYCOMPANY"
duas_node: "NODE01"
duas_area: "APP"
uvms_hostname: "uvms-server.example.com"
uvms_port: "4184"
```

### 5. Run Deployment

Push to GitLab and manually trigger the `deploy_duas` job:

```bash
git add .
git commit -m "Configure DUAS deployment"
git push origin main
```

In GitLab CI/CD > Pipelines, click the play button on `deploy_duas`.

## 🎮 Pipeline Stages

### Stage 1: Download

**Job:** `download_installer`

- Fetches credentials from Vault
- Downloads DUAS installer from Artifactory
- Creates artifact for next stage

### Stage 2: Deploy

**Job:** `deploy_duas`

- Installs system prerequisites
- Performs silent DUAS installation
- Registers with UVMS
- Configures systemd service
- Enables auto-start on boot

### Stage 3: Manage

**Jobs:** `start_duas`, `stop_duas`, `restart_duas`, `status_duas`

- Manual service management operations
- Available after initial deployment

## 📝 Usage Examples

### Deploy DUAS

```bash
# Via GitLab CI (manual job)
# Click "deploy_duas" in pipeline

# Via Ansible directly
cd ansible
ansible-playbook -i inventory.ini playbooks/duas.yml
```

### Manage DUAS Service

#### Via GitLab CI

Click the appropriate manual job in the pipeline:

- `start_duas` - Start the service
- `stop_duas` - Stop the service
- `restart_duas` - Restart the service
- `status_duas` - Check service status

#### Via Ansible

```bash
cd ansible

# Start
ansible-playbook -i inventory.ini playbooks/manage_duas.yml --tags start

# Stop
ansible-playbook -i inventory.ini playbooks/manage_duas.yml --tags stop

# Restart
ansible-playbook -i inventory.ini playbooks/manage_duas.yml --tags restart

# Status
ansible-playbook -i inventory.ini playbooks/manage_duas.yml --tags status
```

#### Via Management Scripts (on target server)

```bash
# After deployment, these scripts are available:
duas_start          # Start DUAS
duas_stop           # Stop DUAS
duas_restart        # Restart DUAS
duas_status         # Check status
```

### Check Service Status on Server

```bash
# Systemd status
systemctl status duas_MYCOMPANY_NODE01.service

# Process check
ps -ef | grep -E "(unidlt|unistart)" | grep -v grep

# User environment
su - duas
source ~/.kshrc
unidlt status
```

## 🔧 Configuration

### DUAS Installation Variables

Key variables in `ansible/roles/duas_install/defaults/main.yml`:

```yaml
# DUAS Instance
duas_company: "MYCOMPANY" # Company code
duas_node: "NODE01" # Node name
duas_area: "APP" # Area designation
duas_install_dir: "/u00/app/automic" # Base install path

# UVMS Registration
uvms_hostname: "uvms-server.example.com"
uvms_port: "4184"
duas_register_with_uvms: true

# User/Group
duas_user: "duas"
duas_group: "duas"
```

### Required System Packages

Automatically installed by Ansible:

- ksh
- libnsl
- libxcrypt-compat
- glibc
- libstdc++
- compat-libstdc++-33
- make, gcc, gcc-c++

### Kernel Parameters

Automatically configured:

```yaml
kernel.msgmnb: 65536
kernel.msgmax: 65536
kernel.shmmax: 68719476736
fs.file-max: 6815744
```

## 🛡️ Security

- ✅ Credentials stored in HashiCorp Vault
- ✅ No hardcoded passwords
- ✅ JWT authentication for CI/CD
- ✅ SSH key-based authentication
- ✅ Firewall configuration included
- ✅ SELinux compatible

## 🐛 Troubleshooting

### Download Fails

```bash
# Check Vault connectivity
curl -s $VAULT_ADDR/v1/sys/health | jq

# Verify Artifactory credentials
vault kv get secret/artifactory/duas
```

### Installation Fails

```bash
# Check logs on target server
tail -f /tmp/duas_installer/*.log
tail -f /var/log/messages

# Verify prerequisites
ansible-playbook -i inventory.ini playbooks/duas.yml --tags prerequisites -v
```

### Service Won't Start

```bash
# Check systemd status
systemctl status duas_MYCOMPANY_NODE01.service -l

# Check journal logs
journalctl -u duas_MYCOMPANY_NODE01.service -n 50

# Verify installation
ls -la /u00/app/automic/MYCOMPANY/NODE01/bin/

# Check permissions
ps aux | grep duas
ls -la /u00/app/automic/MYCOMPANY/NODE01/
```

### UVMS Registration Fails

```bash
# Test connectivity
telnet uvms-server.example.com 4184

# Check firewall
firewall-cmd --list-all

# Manual registration
su - duas
source /u00/app/automic/MYCOMPANY/NODE01/bin/unienv.ksh
unisetup -register -uvms uvms-server.example.com -port 4184
```

## 📚 Dollar Universe 6.10 Specific Notes

### Silent Installation

The unattended installation uses a response file with these key parameters:

- `INSTALL_MODE=SILENT` - No interactive prompts
- `ACCEPT_LICENSE=YES` - Automatic license acceptance
- `REGISTER_WITH_UVMS=Y` - Auto UVMS registration
- `AUTO_START=N` - Managed by systemd instead

### Service Management

DUAS 6.10 uses `unidlt` command:

- `unidlt start` - Start all DUAS components
- `unidlt stop` - Stop all DUAS components
- `unidlt status` - Check component status

### Components

Installation includes:

- **IO Server** (`INSTALL_IO=Y`)
- **Manager** (`INSTALL_MANAGER=Y`)
- **Commands** (`INSTALL_CMD=Y`)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## 📄 License

Internal use only - proprietary software deployment automation.

## 👥 Support

For issues and questions:

- Create an issue in GitLab
- Contact DevOps team
- Check Dollar Universe documentation

## 🔗 References

- [Dollar Universe Documentation](https://docs.automic.com/)
- [Ansible Documentation](https://docs.ansible.com/)
- [GitLab CI/CD](https://docs.gitlab.com/ee/ci/)
- [HashiCorp Vault](https://www.vaultproject.io/docs)

---

**Version:** 1.0.0  
**Last Updated:** January 2026  
**Maintained by:** DevOps Team
