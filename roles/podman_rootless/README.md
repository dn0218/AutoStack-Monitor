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

## 排错与修复小结 (中文)

### 1. 解析失败 (静默错误)
* **现象**: 在 Debian/Ubuntu 上，如果文件包含特殊编码（如 BOM）或行尾符不标准，生成器会直接跳过文件。
* **修复**: 确保文件使用纯净的 LF 换行符，并使用 `/usr/lib/systemd/user-generators/podman-user-generator --dryrun` 进行实时诊断。

### 2. 服务命名不一致
* **现象**: 不同系统生成的 Systemd 服务名不统一，导致 Ansible 无法找到服务。
* **修复**: 通过 Ansible 的 `ansible_os_family` 变量动态指定服务名称，确保启动逻辑在跨平台时具备自适应性。

### 3. Pod 关联错误
* **现象**: Rocky 系统提示 "pod is not Quadlet based"，导致 Pod 启动但内部没有容器。
* **修复**: 严格遵循 Quadlet 的文件命名约定：容器文件必须以 `[Pod名称]-[容器名].container` 格式命名，以强制生成器建立父子关联。

### 4. 网络与权限 (Rocky 特有)
* **现象**: 服务启动但无法访问，报 `No route to host` 或 `Connection reset`。
* **修复**: 检查 `firewalld` 规则是否放行端口，并使用 `chcon` 修正映射目录的 SELinux 安全上下文。

* ## Troubleshooting Recap / 排错与修复记录

### 1. Parsing Failures (Silent Errors)
* **Issue**: On Debian/Ubuntu, `podman-user-generator` may silently ignore Quadlet files if they contain hidden characters (BOM) or lack proper Unix line endings (LF).
* **Fix**: Ensure all template files are saved with pure LF line endings. Use `printf` or standard text editors to create the files. Verify with `/usr/lib/systemd/user-generators/podman-user-generator --dryrun`.

### 2. Service Naming Inconsistency
* **Issue**: Quadlet generates different service names based on the OS. RHEL/Rocky may produce `wp-stack-kube.service`, while Debian/Ubuntu often truncates it to `wp-stack.service`.
* **Fix**: Used conditional Ansible logic for service startup:
    `name: "{{ 'wp-stack-kube.service' if ansible_os_family == 'RedHat' else 'wp-stack.service' }}"`

### 3. Pod Association Logic
* **Issue**: Rocky Linux Quadlet enforces strict parent-child relationships for `.container` files, failing with "pod wp-stack is not Quadlet based" if not correctly linked.
* **Fix**: Implemented the naming convention `[PodName]-[ContainerName].container` (e.g., `wp-stack-mariadb.container`) to force the Quadlet generator to link container definitions to the primary Pod definition correctly.

### 4. Firewall & Permissions (Rocky Specific)
* **Issue**: `curl` failing with `No route to host` or `Connection reset by peer`.
* **Fix**: 
    1. Open firewall ports: `firewall-cmd --permanent --add-port=8081/tcp && firewall-cmd --reload`.
    2. Set SELinux contexts for persistent storage: `chcon -Rt svirt_sandbox_file_t /home/{{ deploy_user }}/data/`.
