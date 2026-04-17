# Troubleshooting Recap (Lessons Learned)

1. **Parsing Failures (Silent Errors)**: 
   - The `podman-user-generator` on Ubuntu/Debian is highly sensitive to hidden characters and formatting. Files missing a trailing newline or containing invisible BOM markers are silently ignored.
   - **Fix**: Use pure LF line endings and standard text editors to ensure file integrity.

2. **Service Naming Inconsistency**:
   - Quadlet on RHEL generates service names differently (e.g., `wp-stack-kube.service`) compared to Debian (e.g., `wp-stack.service`).
   - **Fix**: Used conditional Ansible task logic: `{{ 'wp-stack-kube.service' if ansible_os_family == 'RedHat' else 'wp-stack.service' }}`.

3. **Pod Association Logic**:
   - Rocky Linux Quadlet enforces strict parent-child relationships for `.container` files.
   - **Fix**: Implemented the naming convention `[PodName]-[ContainerName].container` to force the Quadlet generator to link container definitions to the primary Pod definition correctly.
