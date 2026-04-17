# AutoStack-Monitor

[![Ansible](https://img.shields.io/badge/Ansible-2.15+-red.svg)](https://www.ansible.com/)
[![Podman](https://img.shields.io/badge/Podman-Rootless-blue.svg)](https://podman.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)

## 🏗 项目概述 (Project Overview)
**AutoStack-Monitor** 是一个企业级自动化运维方案，旨在 RHEL/Rocky Linux 9 和 Ubuntu 24.04 环境下，通过 Ansible 实现 WordPress 高可用容器栈的“一键式”全自动部署。

该项目核心展示了如何利用 **Podman Quadlet** 技术规避 Root 权限风险，同时解决跨发行版在容器编排上的差异。

## 🚀 架构方案 (Architecture)
* **容器编排**: 使用 Systemd Quadlet（RHEL 原生）和 Kube YAML（Ubuntu 兼容模式）。
* **安全模型**: 采用 **Rootless Podman**，容器以普通用户权限运行，极大降低宿主机被提权的风险。
* **存储方案**: 基于 NFS 的分布式存储，实现 WordPress `wp-content` 跨节点挂载与数据持久化。
* **监控体系**: Zabbix Agent 2 (监控容器指标) + Grafana (可视化面板)，监控数据通过 Podman Socket 采集。

## 📋 环境准备 (Prerequisites)
* **操作系统**: Rocky Linux 9.x / Ubuntu 24.04 LTS (建议最小化安装)。
* **硬件建议**: 至少 2GB RAM, 20GB 磁盘空间。
* **网络要求**: 确保控制端（Ansible 主机）可通过 SSH 免密登录被控节点。

## 🛠 快速上手 (Quick Start)

1. **克隆项目**:
   ```bash
   git clone <your-repo-url>
   cd AutoStack-Monitor
   ```
2. **配置清单 (Inventory)**:
编辑 inventory.ini，指定目标主机的 IP 地址和 SSH 用户。

3. **执行部署**:
   ```bash
   # 检查语法与逻辑
ansible-playbook -i inventory.ini playbooks/site.yml --check

# 执行部署
ansible-playbook -i inventory.ini playbooks/site.yml
    ```
4. **验证部署**:
- WordPress: http://<node_ip>:8081

- Grafana: http://<node_ip>:3000 (User: admin / Pass: admin)

## 📁 项目结构 (Project Structure)
```bash
AutoStack-Monitor/
├── inventory.ini           # 节点清单
├── playbooks/              # 入口 Playbooks
├── roles/                  # 核心角色逻辑
│   ├── common/             # 系统基础初始化
│   ├── podman_rootless/    # Quadlet 容器编排
│   ├── nfs_server/         # 存储服务端配置
│   └── monitoring/         # Zabbix/Grafana 监控部署
└── uninstall.yml           # 环境清理剧本
```

## 💡 设计决策 (Design Decisions)
**为什么使用 Rootless Podman?**

- 符合“最小特权原则”（Least Privilege），即便容器被攻破，攻击者也无法直接获得宿主机 Root 权限。

**为什么 Quadlet 对比 Docker Compose？**

- Quadlet 是红帽主推的容器集成方式，它直接调用 Systemd 管理容器生命周期，实现“容器即服务”，支持自动重启和依赖编排。

**针对跨 OS 的处理**:

- 针对 RHEL 9，利用 .pod 文件实现原生 Quadlet 支持；针对 Ubuntu，通过 podman kube 指令封装，屏蔽了发行版在底层容器运行时的微小差异。

## 🛠 故障排查 (Troubleshooting)
检查容器状态: systemctl --user status wordpress.service

查看日志: journalctl --user -u wordpress.service -f

存储问题: 检查 /etc/exports 是否已成功更新，并确认 nfs-server 服务在被控机已启动。

## 🗺 路线图 (Roadmap)
[x] RHEL 9 & Ubuntu 24.04 跨平台部署

[x] Rootless Podman + Quadlet 编排

[x] Zabbix + Grafana 监控集成

[ ] Windows Server 2019 Core 节点适配 (In Progress)

