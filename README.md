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

### Troubleshooting Recap (Lessons Learned)

1. **Parsing Failures (Silent Errors)**: 
   - The `podman-user-generator` on Ubuntu/Debian is highly sensitive to hidden characters and formatting. Files missing a trailing newline or containing invisible BOM markers are silently ignored.
   - **Fix**: Use pure LF line endings and standard text editors to ensure file integrity.

2. **Service Naming Inconsistency**:
   - Quadlet on RHEL generates service names differently (e.g., `wp-stack-kube.service`) compared to Debian (e.g., `wp-stack.service`).
   - **Fix**: Used conditional Ansible task logic: `{{ 'wp-stack-kube.service' if ansible_os_family == 'RedHat' else 'wp-stack.service' }}`.

3. **Pod Association Logic**:
   - Rocky Linux Quadlet enforces strict parent-child relationships for `.container` files.
   - **Fix**: Implemented the naming convention `[PodName]-[ContainerName].container` to force the Quadlet generator to link container definitions to the primary Pod definition correctly.

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
