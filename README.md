# MU Game Server Provision or Rebuild Instructions

## Overview

This repository provisions and configures Ubuntu 24.04 game servers using Ansible.

It provides a consistent and repeatable deployment process for new MMORPG zone servers with the following features:

* System baseline configuration
* User management
* SSH hardening
* Operating system hardening
* Firewall configuration (UFW)
* Node Exporter installation
* Game server directory preparation
* Deployment validation

---

# Repository Structure

```text
ansible/
├── inventories/
│   ├── production/
│   └── staging/
├── playbooks/
├── roles/
│   ├── common/
│   ├── users/
│   ├── ssh/
│   ├── hardening/
│   ├── firewall/
│   ├── node_exporter/
│   ├── game_server/
│   └── validation/
```

---

# Prerequisites

Controller machine:

* macOS or Linux
* Ansible 2.16+
* Python 3
* SSH access to target servers

Target servers:

* Ubuntu Server 24.04 LTS
* Network connectivity from the Ansible controller
* SSH access using the deployment user

---

# Initial Setup

Clone the repository.

```bash
git clone <repository-url>
cd <repository>
```

Install the required Ansible collection.

```bash
ansible-galaxy collection install community.general
```

Verify Ansible installation.

```bash
ansible --version
```

---

# Configure Inventory

Edit the inventory file.

```
ansible/inventories/production/hosts.yml
```

Example:

```yaml
game_servers:
  hosts:
    zone01:
      ansible_host: 10.0.1.10
    zone02:
      ansible_host: 10.0.1.11
```

Update variables if required.

```
ansible/inventories/production/group_vars/all.yml
```

Typical variables include:

* deployment user
* timezone
* game directories
* Node Exporter port

---

# Validate Configuration

Before deployment, run a syntax check.

```bash
cd ansible

ansible-playbook --syntax-check playbooks/site.yml
```

Validate the inventory.

```bash
ansible-inventory -i inventories/production/hosts.yml --list
```

---

# Provision a New Game Server

Run the complete playbook.

```bash
ansible-playbook \
-i inventories/production/hosts.yml \
playbooks/site.yml
```

This will:

1. Configure the operating system
2. Create required users
3. Harden SSH
4. Apply OS security baseline
5. Configure firewall
6. Install Node Exporter
7. Prepare game directories
8. Run validation checks

---

# Deploy to Staging

```bash
ansible-playbook \
-i inventories/staging/hosts.yml \
playbooks/site.yml
```

---

# Add a New Game Server

1. Provision a new Ubuntu 24.04 server.
2. Add the server IP to the appropriate inventory.
3. Verify SSH connectivity.
4. Run the deployment playbook.
5. Confirm the validation tasks complete successfully.

No changes to playbooks or roles are required for adding additional servers.

---

# Rollback

If deployment fails:

1. Fix the configuration issue.
2. Re-run the playbook.

All roles are designed to be idempotent, allowing the deployment to be safely executed multiple times.

---

# Security

This repository applies several baseline security controls:

* SSH key authentication
* Root login disabled
* Password authentication disabled
* UFW firewall enabled
* Linux kernel hardening
* Dedicated service account for the game server

---

# Validation

After deployment, verify:

* SSH access is available.
* Firewall is enabled.
* Node Exporter service is running.
* Game directories exist.
* Ansible validation tasks complete successfully.

---

# Notes

This repository focuses on infrastructure provisioning and operating system configuration. Game binaries and application deployment are intentionally outside the scope of this repository.


---

# Future Improvements (From this assignment)

The current implementation focuses on providing a clean, repeatable, and secure baseline for provisioning Ubuntu-based MMORPG game servers. The following enhancements could be implemented as the infrastructure evolves.

## Infrastructure Automation

* Integrate with cloud provisioning tools (Terraform or OpenTofu) to automate VM and networking creation before Ansible configuration.
* Support multiple Linux distributions through OS-specific variables and tasks.
* Separate environment-specific configuration into dedicated inventory repositories.

## Security

* Store sensitive information using Ansible Vault or an external secrets manager (e.g., HashiCorp Vault, AWS Secrets Manager).
* Implement automated operating system patch management and scheduled maintenance windows.
* Add file integrity monitoring and security auditing.
* Apply CIS Benchmark hardening profiles where appropriate.

## CI/CD

* Add GitHub Actions or GitLab CI pipelines to automatically execute:

  * YAML validation
  * Ansible syntax checks
  * ansible-lint
  * Molecule tests
* Require pull request approval before merging infrastructure changes.

## Testing

* Add Molecule test scenarios for every Ansible role.
* Include automated verification of idempotency.
* Perform integration testing using disposable virtual machines or containers.

## Monitoring & Observability

* Extend monitoring beyond Node Exporter by deploying:

  * Prometheus
  * Grafana
  * Centralized log collection
* Add health checks for game server processes and critical services.
* Configure alerting for infrastructure failures and resource utilization.

## Game Server Deployment

* Automate deployment of game server binaries from an artifact repository.
* Manage application configuration using templates.
* Deploy and manage the game server as a systemd service.
* Support rolling updates and versioned releases with rollback capability.

## Operational Improvements

* Support zero-downtime deployments where applicable.
* Implement automatic backup and restore procedures.
* Add maintenance mode workflows for game server updates.
* Generate deployment reports after each execution.
