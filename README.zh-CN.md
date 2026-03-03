# pve-automation

[English](./README.md) | 简体中文

Proxmox VE (PVE) 自动化仓库，包含两部分：

1. 可直接运行的 Python API Client/CLI（`scripts/pve_client.py`）
2. 面向 Codex 的 PVE 自动化技能说明（`SKILL.md`）

这个仓库不只是“几个命令示例”，而是一个可扩展的 PVE 自动化基础层 + 一整套运维工作流参考。

## 仓库结构

- `scripts/pve_client.py`：可执行客户端与 CLI
- `SKILL.md`：PVE 自动化能力说明、API 工作流与最佳实践
- `evals/evals.json`：评估配置

## 已实现能力（代码层）

`PVEClient` 已封装以下 API 能力：

- 节点管理
  - `list_nodes`
  - `get_node_status`
- VM（QEMU）管理
  - `list_vms`
  - `get_vm_status`
  - `start_vm`
  - `stop_vm`
  - `shutdown_vm`
  - `reset_vm`
  - `create_vm`
  - `delete_vm`
  - `clone_vm`
- LXC 容器管理
  - `list_containers`
  - `get_container_status`
  - `start_container`
  - `stop_container`
  - `create_container`
  - `delete_container`
- 任务管理
  - `get_task_status`
  - `wait_for_task`
- 存储管理
  - `list_storage`
  - `get_storage_content`
- 集群管理
  - `get_cluster_resources`
  - `get_cluster_status`
- 快照管理
  - `list_snapshots`
  - `create_snapshot`
  - `rollback_snapshot`

### 已实现 CLI 命令

当前 CLI 直接支持：

- `list-nodes`
- `list-vms <node>`
- `list-containers <node>`
- `start-vm <node> <vmid>`
- `stop-vm <node> <vmid>`
- `create-vm <node> --vmid ... --name ... [--memory] [--cores] [--storage] [--disk] [--net] [--iso]`
- `wait-task <node> <upid> [--timeout]`

## SKILL 支持范围（文档/工作流层）

`SKILL.md` 覆盖了完整的 PVE 自动化方法论与 API 操作路径，包括：

- 认证与连接
  - API Token 认证
  - Ticket/CSRF 认证
  - 自定义 API 端口
- 核心资源操作
  - 节点、VM、LXC、集群、任务查询
- 模板与镜像
  - 模板列表、下载、aplinfo 模板源使用
- Cloud-Init
  - `ciuser`/`cipassword`/`ipconfig`/`cicustom` 等配置策略
- 快照与备份
  - VM 快照创建/回滚
  - 备份创建、恢复、清理、备份查询
- 存储与网络
  - 存储内容管理、上传路径说明、网络参数规范
- 安全与稳定性
  - 资源校验（容量/配额）
  - 批量操作 dry-run 思路
  - 常见错误模式与处理建议
- 高级运维能力（API 工作流）
  - HA（高可用）
  - 在线迁移（Live Migration）
  - PCI/GPU 直通参数配置
  - 定时任务（如备份计划）
  - 权限管理（用户/角色/API Token）
  - 通知系统（Webhook/通知目标/匹配器）
  - 复制（Replication）
  - 防火墙（节点级/VM级）
  - 存储配置
  - 证书与 ACME 插件管理
  - 节点维护流程

说明：上述“SKILL 支持范围”中的部分能力属于工作流与接口指南；如需 CLI 一键化，可在 `scripts/pve_client.py` 继续扩展对应子命令。

## 运行环境

- Python 3.9+
- `requests`
- `urllib3`

安装依赖：

```bash
pip install requests urllib3
```

## 认证参数

可用环境变量：

- `PVE_HOST`：PVE 主机或 IP
- `PVE_USER`：PVE 用户（默认 `root@pam`）
- `PVE_TOKEN_ID`：API Token ID
- `PVE_SECRET`：API Token Secret

示例：

```bash
export PVE_HOST=192.168.1.10
export PVE_USER=root@pam
export PVE_TOKEN_ID=automation
export PVE_SECRET=your-token-secret
```

也可使用全局参数覆盖：

```bash
python scripts/pve_client.py --host 192.168.1.10 --user root@pam --token-id automation --token-secret 'xxx' list-nodes
```

## 快速示例

```bash
# 1) 查询节点
python scripts/pve_client.py list-nodes

# 2) 查询某节点 VM
python scripts/pve_client.py list-vms pve-node-1

# 3) 创建 VM
python scripts/pve_client.py create-vm pve-node-1 \
  --vmid 120 \
  --name test-vm \
  --memory 4096 \
  --cores 2 \
  --storage local-lvm \
  --disk 32 \
  --net 'virtio,bridge=vmbr0'

# 4) 启动 VM
python scripts/pve_client.py start-vm pve-node-1 120
```

## 作为库使用（Python）

```python
from scripts.pve_client import PVEClient

client = PVEClient()

# 集群资源
resources = client.get_cluster_resources()

# VM 快照
client.create_snapshot('pve-node-1', 120, 'before-upgrade')
```

## 生产使用建议

- 当前实现默认 `verify=False`（关闭 SSL 校验），仅建议在可信内网使用
- 生产环境建议启用有效证书并开启 SSL 校验
- 为 API Token 设置最小权限原则，避免使用过大权限的 root token
