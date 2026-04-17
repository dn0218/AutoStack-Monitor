# AutoStack-Monitor

Automated deployment of a WordPress stack (WordPress + MariaDB + Redis) on **RHEL/Rocky Linux** and **Debian/Ubuntu** using Podman Rootless and Ansible.

## Architecture Overview
This project provides a robust, cross-platform deployment strategy for containerized applications. 
- **RHEL/Rocky Linux**: Leverages native **Quadlet (.pod + .container)** definitions for fine-grained systemd integration.
- **Debian/Ubuntu**: Utilizes **Kube YAML (podman kube play)** managed by Quadlet for maximum compatibility and reliability.



## Key Technical Challenges & Solutions

### Cross-Distro Deployment Summary

| Feature | Rocky Linux (RHEL family) | Ubuntu (Debian family) |
| :--- | :--- | :--- |
| **Deployment Mode** | Quadlet (.pod + .container) | Kube YAML (Play Kube) |
| **Parsing Logic** | Strict, requires file naming convention | Sensitive to file formatting/BOM |
| **Firewall** | Requires `firewalld` port openings | No additional restrictions |


## Deployment Guide

### Prerequisites
- Ansible installed on the control node.
- SSH access to target nodes.

### Execution
1. Clone this repository.
2. Configure your `inventory.ini` with target node IPs and `deploy_user`.
3. Execute the deployment:
   ```bash
   ansible-playbook -i inventory.ini playbooks/site.yml
