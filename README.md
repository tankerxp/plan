# 家庭实验 / 开发平台 VM / CT 部署计划（带打钩）

目标：  
先搭建完整 VM / CT 架子，保证 NAS、iStore、Docker、CI/CD、监控、Home Assistant、K8s 学习等基础环境可用，软件内容可后续逐步填充。

---

# 主机环境

| 项目 | 配置 |
|-----|-----|
| 虚拟化平台 | Proxmox VE |
| CPU | Intel i3-12100T (4核8线程) |
| 内存 | 32GB |
| 系统盘 | NVMe 2TB |
| 网络 | 10Gb NIC |
| 交换机 | GS10 |
| 路由器 | MR60 |

---

# VM / CT 部署规划（可打钩）

| 完成 | 名称 | 类型 | CPU | 内存 | 磁盘 | 操作系统 | 版本 | VM/CT安装选项 | IP 地址 | 主要软件 & 版本 | 下载地址 | 备份策略 | 说明 |
|-------|---|---|---|---|---|---|---|---|---|---|---|---|---|
| [ ] | 飞牛 NAS | VM | 2 核 | 10GB | 32GB + RAID10 | Ubuntu Server | 24.04 LTS | BIOS: SeaBIOS, 机型: Q35, VirtIO网卡, SCSI VirtIO磁盘控制器, 桥接网络 | 10.0.0.10 | NFS 1.3+, SMB/CIFS | https://ubuntu.com/download/server | 快照每周一次，重要数据卷每日备份 | NAS 存储共享给其他 VM / CT |
| [ ] | iStore | VM | 1 核 | 2GB | 20GB | iStoreOS | 0.1.36 最新稳定 | BIOS: SeaBIOS, 机型: Q35, VirtIO网卡, SCSI VirtIO磁盘控制器, 桥接网络 | 10.0.0.11 | Tailscale Exit Node | https://istoreos.com | 快照每周一次 | 网络出口，手机/PC翻墙，ACL 配置控制流量 |
| [ ] | Docker 主机 | VM | 1 核 | 6GB | 50GB | Ubuntu Server | 24.04 LTS | BIOS: SeaBIOS, 机型: Q35, VirtIO网卡, SCSI VirtIO磁盘控制器, 桥接网络 | 10.0.0.12 | Docker 23.x + Docker Compose | https://docs.docker.com/get-docker/ | 每周快照，重要容器卷每日备份 | 容器中间件运行环境，挂载 NAS 数据卷 `/mnt/nas_data/docker` |
| [ ] | Gitea | VM | 1 核 | 2GB | 20GB | Ubuntu Server | 24.04 LTS | BIOS: SeaBIOS, 机型: Q35, VirtIO网卡, SCSI VirtIO磁盘控制器, 桥接网络 | 10.0.0.13 | Gitea 1.20.x | https://dl.gitea.io/gitea | 每周快照 | 代码仓库，数据库可用 Docker VM 内的 MySQL / PostgreSQL |
| [ ] | CI Runner | CT | 0.5 核 | 1GB | 10GB | Debian | 12 | LXC模板: Debian 12, 网络桥接, RootFS ext4 | 10.0.0.14 | Gitea Runner / Docker Runner 最新 | https://docs.gitea.io/en-us/ci-cd/ | 每周快照 | 自动化部署，挂载 NAS 卷用于构建产物 |
| [ ] | 监控系统 | CT | 1 核 | 2GB | 10GB | Debian | 12 | LXC模板: Debian 12, 网络桥接, RootFS ext4 | 10.0.0.15 | Grafana 10.x / Prometheus 2.48 | https://grafana.com/grafana/download / https://prometheus.io/download/ | 每周快照 | 系统监控，挂载 NAS 卷保存监控数据 |
| [ ] | Home Assistant | VM | 1 核 | 2GB | 20GB | HA OS | 2026.2.x | BIOS: SeaBIOS, 机型: Q35, VirtIO网卡, SCSI VirtIO磁盘控制器, 桥接网络 | 10.0.0.16 | Home Assistant 2026.2.x | https://www.home-assistant.io/installation/ | 每周快照 | 智能家居、摄像头接入，挂载 NAS 数据卷可存摄像头录像 |
| [ ] | OpenClaw | CT | 0.5 核 | 1GB | 10GB | Debian | 12 | LXC模板: Debian 12, 网络桥接, RootFS ext4 | 10.0.0.17 | Docker / OpenClaw 最新 | https://github.com/openclaw/openclaw | 每周快照 | 自动化执行 HA + CI/CD 任务，挂载 NAS 卷 |
| [ ] | Kubernetes 学习 | VM | 1 核 | 4GB | 20GB | Ubuntu Server | 24.04 LTS | BIOS: SeaBIOS, 机型: Q35, VirtIO网卡, SCSI VirtIO磁盘控制器, 桥接网络 | 10.0.0.18 | Minikube / Kind 最新 | https://minikube.sigs.k8s.io/docs/ | 每周快照 | 容器编排实验 |

---

# NAS 数据卷挂载规划

共享路径：`/mnt/nas_data`  

Docker / CI / HA 挂载示例：

```
/mnt/nas_data/docker/mysql
/mnt/nas_data/docker/redis
/mnt/nas_data/docker/es
/mnt/nas_data/docker/kafka
/mnt/nas_data/docker/oracle
/mnt/nas_data/docker/homeassistant
```

> 先挂 NAS，再部署 Docker / 中间件 / HA，保证数据持久化  

---

# 网络出口 / 防火墙规则

- iStore VM 作为 Tailscale Exit Node  
- ACL 控制每台设备访问权限  
- 出口流量可限制到特定服务（手机/PC翻墙、Docker 上网等）  
- VM/CT 之间桥接网络，主机防火墙可隔离必要端口  

---

# 各 VM / CT 内部软件安装概览

## 飞牛 NAS

- NFS Server 1.3+  
- SMB/CIFS 最新稳定  
- 用于共享给 Docker / Gitea / HA  

## iStore VM

- Tailscale Exit Node  
- ACL 配置控制访问  

## Docker 主机

- Docker 23.x + Docker Compose  
- NAS 数据卷挂载 `/mnt/nas_data/docker`  

## Gitea

- Gitea 1.20.x  
- 数据库可用 Docker VM 的 MySQL / PostgreSQL  
- 官方地址：https://dl.gitea.io/gitea  

## CI Runner

- Gitea Runner / Docker Runner 最新  
- 官方文档：https://docs.gitea.io/en-us/ci-cd/  

## 监控系统

- Prometheus 2.48  
- Grafana 10.x  
- 官方下载：  
  - Grafana：https://grafana.com/grafana/download  
  - Prometheus：https://prometheus.io/download/  

## Home Assistant

- HA OS 2026.2.x  
- 官方安装：https://www.home-assistant.io/installation/  
- NAS 卷挂载用于摄像头录像存储  

## OpenClaw

- Docker 镜像最新  
- 官方地址：https://github.com/openclaw/openclaw  

## Kubernetes 学习

- Minikube / Kind 最新  
- 官方地址：https://minikube.sigs.k8s.io/docs/  
- 单节点 VM，CPU 1 核，4GB 内存即可  

---

> ⚠️ 注意：
> 1. 先把 VM / CT 架子搭起来，内部软件安装可后续逐步补充  
> 2. 确保固定 IP、NAS 挂载、备份策略到位  
> 3. iStore Exit Node ACL 配置和防火墙策略要安装后检查  
> 4. 所有中间件数据库卷要挂 NAS 数据卷，保证持久化
