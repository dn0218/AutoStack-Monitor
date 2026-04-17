# Role: NFS Storage (Server & Client)

## 中文说明
### 功能概述
实现跨主机的持久化存储，解决 WordPress 附件同步问题。
**服务端**: 在 `node1` 上安装并配置 `nfs-kernel-server` 。
**安全性**: 针对 RHEL 系统自动配置 `firewalld` 放行 NFS 流量 。
**挂载策略**: 客户端使用 `rw,sync,hard` 参数挂载，确保数据写入的可靠性 。

### 为什么这样写？
**no_root_squash**: 在 exports 中使用此参数是为了简化 rootless 容器挂载时的权限冲突问题。
**条件触发**: 通过 `is_nfs_server` 变量灵活控制节点角色，避免重复安装 。

---

## English Description
### Overview
Implements cross-host persistent storage to resolve WordPress attachment synchronization issues.
**Server**: Installs and configures `nfs-kernel-server` on `node1`.
**Security**: Automatically configures `firewalld` to allow NFS traffic on RHEL systems.
**Mount Strategy**: Clients use `rw,sync,hard` parameters for mounting to ensure data reliability.

### Rationale
**no_root_squash**: This parameter is used in exports to simplify permission conflicts during rootless container mounting.
**Conditional Execution**: Uses the `is_nfs_server` variable to flexibly control node roles and prevent redundant installations.
