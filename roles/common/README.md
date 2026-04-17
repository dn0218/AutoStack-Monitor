# Role: common

## 中文说明
### 功能概述
此 Role 负责初始化目标主机的基础环境，确保 rootless 容器能够正常运行。
* [cite_start]**基础包安装**: 安装 `podman`, `acl`, `sudo` 等必要组件 [cite: 109]。
* [cite_start]**用户环境**: 创建部署用户 `deploy_user` 并配置无密码 sudo 权限 [cite: 110, 111]。
* [cite_start]**持久化配置**: 通过 `loginctl enable-linger` 确保用户退出登录后容器继续运行 [cite: 112]。
* [cite_start]**权限映射**: 配置 `/etc/subuid` 和 `/etc/subgid` 以支持 rootless 模式下的 UID 映射 [cite: 112]。
* [cite_start]**API 激活**: 开启用户级别的 Podman Socket [cite: 112, 113]。

### 为什么这样写？
* **ACL & Sudo**: Rootless 模式下，Ansible 切换用户执行任务需要 ACL 支持，否则会出现权限报错。
* **Linger**: 默认情况下，用户注销后其 systemd 进程会被清理，`linger` 是保证容器服务化运行的前提。

---

## English Description
### Overview
This role initializes the base environment of the target host to ensure rootless containers can run correctly.
* [cite_start]**Base Packages**: Installs `podman`, `acl`, `sudo`, and other essential components[cite: 109].
* [cite_start]**User Setup**: Creates the `deploy_user` and configures passwordless sudo privileges[cite: 110, 111].
* [cite_start]**Persistence**: Uses `loginctl enable-linger` to ensure containers remain running after the user logs out[cite: 112].
* [cite_start]**Namespace Mapping**: Configures `/etc/subuid` and `/etc/subgid` to support UID mapping in rootless mode[cite: 112].
* [cite_start]**API Activation**: Enables the user-level Podman Socket[cite: 112, 113].

### Rationale
* **ACL & Sudo**: In rootless mode, Ansible requires ACL support to switch users during task execution; otherwise, permission errors occur.
* **Linger**: By default, systemd user processes are terminated upon logout. `linger` is a prerequisite for running containers as persistent services.
  
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
