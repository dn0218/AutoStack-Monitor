# Role: podman_rootless

## 中文说明
### 功能概述
这是项目的核心，负责利用 Quadlet 编排整个 WordPress 应用栈（WordPress, MariaDB, Redis）。
* **跨平台适配**: 
    **RedHat 族**: 部署原生的 `.pod` 和 `.container` 文件 。
    **Debian 族**: 利用 Quadlet 托管 `Kube YAML` 方案，增强稳定性 。
**网络隔离**: 自动创建 `wp-network` 虚拟网络，确保 Pod 内外通讯隔离 。
**自动化管理**: 任务包含清理旧容器、重载 systemd 生成器以及启动服务 。

### 为什么这样写？
* **Quadlet vs Docker Compose**: Quadlet 将容器直接转化为 systemd 服务，支持自动重启、依赖检查，且完全遵循 rootless 规范。
**Kube Way (Ubuntu)**: Ubuntu 的旧版 Podman 对多容器 Quadlet 依赖解析存在 Bug，采用 Kube YAML 更加健壮 。

---

## English Description
### Overview
The core of the project, responsible for orchestrating the entire WordPress stack (WordPress, MariaDB, Redis) using Quadlet.
* **Cross-Platform Adaptation**:
    **RedHat Family**: Deploys native `.pod` and `.container` files.
    **Debian Family**: Utilizes the `Kube YAML` strategy managed by Quadlet for enhanced stability.
**Network Isolation**: Automatically creates the `wp-network` virtual network to isolate internal and external communications.
**Automated Management**: Tasks include cleaning up old containers, reloading systemd generators, and starting services.

### Rationale
* **Quadlet vs Docker Compose**: Quadlet converts containers directly into systemd services, supporting automatic restarts and dependency checks while fully complying with rootless standards.
**Kube Way (Ubuntu)**: Older versions of Podman on Ubuntu have bugs in resolving multi-container Quadlet dependencies; using Kube YAML is more robust.
