---
title: "StarRocks 学习笔记 Day4"
description: 本文涵盖了集群的部署、升级回滚、扩缩容以及参数调整等核心运维操作。
date: 2025-06-05
image: sr.png
categories:
    - StarRocks
    - 数据库
---

## 01. 集群部署

 StarRocks 集群的规范化部署遵循六个主要步骤 。

### ① 检查部署要求

**硬件要求**
*  **CPU**: FE 推荐 8 核以上，BE 推荐 16 核以上。x86 架构下 BE 必须支持 AVX2 指令集才能启动 。
*  **内存**: 生产环境 FE 推荐 16GB+，BE 推荐 64GB+ 。
*  **存储**: 推荐使用 SSD   。FE 预留 100GB 以上独立存储   。BE 磁盘容量根据数据量（默认三副本和 LZ4 压缩）、节点数和 20% 以上的冗余空间来评估 。
*  **网络**: 强烈推荐万兆（10GE）或以上网络 。

**软件与系统要求**
*  **操作系统**: 推荐 CentOS 7.9 或 Ubuntu 22.04 。
* **软件环境**:
    *  所有节点需安装 JDK   。StarRocks 2.5 及以上版本推荐 JDK 11（3.3 版本起不再支持 JDK 8）。
    *  推荐安装 MySQL 客户端便于运维 。

### ② 规划集群架构

*  **FE (Frontend)**: 生产环境推荐部署 3 个 Follower 角色的 FE 节点以实现高可用   。不允许在同一 IP 的机器上部署多个 FE 实例 。
*  **BE (Backend)**: 生产环境推荐至少部署 3 个 BE 节点以实现三副本高可用   。查询性能与 BE 数量正相关，推荐服务器同构 。
*  **CN (Compute Node)**: 存算一体架构下的可选组件，用于扩展计算能力 。
*  **Broker**: 可选组件，按需部署 。

### ③ 配置系统环境

 部署前需对所有服务器进行系统参数配置，以保证集群稳定性和性能   。带 `*` 的为必调项 。

| 配置项 | 描述与要求 |
| --- | --- |
|  **CPU AVX2 指令集检查**\* | x86 架构下 BE 依赖 AVX2   。检查命令：`cat /proc/cpuinfo | grep avx2` 。 |
| **操作系统检查**\* |  内核版本不低于 3.10，glibc 版本不低于 2.17 。 |
| **端口冲突检查**\* | 确认 FE/BE/CN 的默认端口未被占用 。 |
| **JDK 配置**\* |  推荐 JDK 11，并配置好 `JAVA_HOME` 环境变量 。 |
| **Overcommit 参数设置**\* |  要求 `overcommit_memory=1`   。修改命令：`echo 1 > /proc/sys/vm/overcommit_memory` 。 |
| **mmap 参数设置**\* |  推荐调整到 200万以上   。修改命令：`echo 2000000 > /proc/sys/vm/max_map_count` 。 |
| **透明大页参数设置**\* |  推荐禁用   。修改命令：`echo never > /sys/kernel/mm/transparent_hugepage/enabled` 。 |
| **Swap 分区设置**\* |  要求禁用交换分区，`swapoff -a`   。同时设置 `swappiness=0` 。 |
| **磁盘调度算法**\* | HDD 建议使用 `deadline`，SSD 建议使用 `noop` 。 |
| **SELinux 设置**\* |  推荐禁用   。修改命令：`setenforce 0` 。 |
| **防火墙设置**\* | 内网环境推荐禁用防火墙 `systemctl stop firewalld` 。 |
| **ulimit 设置**\* |  最大句柄数(nofile)和用户进程数(nproc)要求大于 65535 。 |
| **网络参数设置**\* |  推荐调整 `tcp_abort_on_overflow=1` 和 `somaxconn=1024` 。 |
| **时钟同步**\* |  各 FE 间时钟差大于 5 秒服务将无法启动   。推荐使用 ntp 服务进行同步 。 |
| **文件系统设置**\* |  推荐使用 ext4 或 xfs 文件系统 。 |

### ④ 准备部署文件

 可从 StarRocks 社区官网或镜舟科技官网获取二进制安装包   。版本号格式为 `Major.Minor.Patch` 。

### ⑤ 部署 StarRocks

 以部署 3 个 FE 和 3 个 BE 节点为例 ：
1.   **前置准备**: 创建用户、授权目录、配置 SSH 免密、解压并分发安装包 。
2.  **FE 部署**:
    *  **创建元数据目录**: `mkdir meta` 。
    * **修改配置 `fe.conf`**:
        *  配置 JVM 堆内存（生产推荐 20G+）。
        *  配置元数据目录 `meta_dir` 。
        *  通过 `priority_networks` 绑定服务 IP 。
    * **启动与添加节点**:
        *  在第一个节点上执行 `./start_fe.sh --daemon` 启动服务 。
        *  使用 MySQL 客户端连接该 FE，添加其他 FE 节点：`ALTER SYSTEM ADD FOLLOWER 'ip:9010';` 。
        *  在其他节点上以 `--helper` 方式启动 FE：`./start_fe.sh --helper <node1_ip>:9010 --daemon` 。
3.  **BE 部署**:
    *  **创建数据目录**: `mkdir storage` 。
    * **修改配置 `be.conf`**:
        *  配置数据存储路径 `storage_root_path`，支持多目录配置 。
        *  通过 `priority_networks` 绑定服务 IP 。
    * **添加与启动节点**:
        *  使用 MySQL 客户端连接 FE，添加 BE 节点：`ALTER SYSTEM ADD BACKEND 'ip:9050';` 。
        *  在所有 BE 节点上执行 `./start_be.sh --daemon` 启动服务 。
4.  **服务启停与密码修改**:
    *  启停脚本位于各组件的 `bin` 目录下 (`start_xx.sh`, `stop_xx.sh`) 。
    *  初始 root 用户密码为空，可通过 `SET PASSWORD = PASSWORD('new_password');` 修改 。

### ⑥ 部署后参数调整

*  **`fe.conf` 中 Xmx 堆内存**: 生产环境建议调整至 20GB 以上 。
*  **用户连接数**: 默认 100，可按需调整 `max_user_connections` 。
* **BE 内存配置**:
    *  通过 `mem_limit` 限制 BE 内存使用，默认为服务器内存的 90% 。
    *  若使用 UDF 等功能，需配置 BE 的 JVM 内存上限 `JAVA_OPTS="-Xmx4096m ..."` 。

## 02. 升级与回滚

### 集群升级

*  **升级分类**: 分为小版本升级（如 3.1.10 -> 3.1.11）和大版本升级（如 3.0 -> 3.1）[cite: 95, 96] 。大版本升级不建议跨版本进行 。
*  **升级顺序**: 先升级 BE/CN，再升级 FE   。FE 内部顺序为 Observer -> Follower -> Leader 。
*  **升级操作**: 本质是替换 `bin` 和 `lib` 等程序文件目录   。为减少停机时间，可先将新版本文件放入部署目录，停止服务后，通过重命名目录的方式快速切换，然后启动新服务 。

### 集群降级/回滚

*  **降级分类**: 同样分为小版本和大版本降级 [cite: 112, 113] 。大版本降级不建议跨版本，且只推荐降级到目标大版本的最新小版本 。
*  **降级顺序**: 与升级相反，先降级 FE，再降级 BE/CN   。FE 内部顺序为 Observer -> Follower -> Leader 。
*  **降级操作**: 与升级操作类似，通过替换程序文件目录，将版本回滚至升级前的备份版本 。

## 03. 集群扩缩容

### 集群扩容

*  **FE 扩容**: 当 FE 负载高时，可扩容 Observer 节点   。新节点 `http_port` 必须与现有集群一致 。首次启动需以 `--helper` 方式加入集群 。
*  **BE 扩容**: 当存储或计算资源出现瓶颈时，可扩容 BE   。新节点推荐与现有 BE 同构   。扩容后数据会自动均衡 。

### 集群缩容

*  **FE 缩容**: 使用 `ALTER SYSTEM DROP OBSERVER/FOLLOWER 'ip:9010';` 命令   。不允许直接缩容 Leader 节点 。
* **BE 缩容**:
    *  `DECOMMISSION`（推荐）: `ALTER SYSTEM DECOMMISSION BACKEND 'ip:9050';`   。此方式会先将数据安全迁移至其他 BE 节点，再下线该节点 。
    *  `DROP`: `ALTER SYSTEM DROP BACKEND 'ip:9050';`。此方式会立刻删除 BE，可能导致数据丢失 。

## 04. 参数调整

### 参数分类

1.  **系统变量 (System Variables)**:
    *  通过 `SHOW VARIABLES;` 查看 。
    *  支持 SQL 级 (`/*+ SET_VAR(...) */`)、会话级 (`SET SESSION ...`) 和全局级 (`SET GLOBAL ...`) 三种生效范围 。

2.  **FE 参数**:
    *  分为动态参数和静态参数 。
    *  **动态参数**: 支持在线修改 (`ADMIN SET FRONTEND CONFIG ('key'='value');`)，立即生效但重启后失效 。
    *  **静态参数**: 只能通过修改 `fe.conf` 文件并重启 FE 生效 。

3.  **BE 参数**:
    *  同样分为动态和静态参数 。
    *  **动态参数**: 支持通过 HTTP 请求临时修改 (`curl -XPOST http://be_ip:be_http_port/api/update_config?key=value`)，立即生效但重启后失效 。
    *  **静态参数**: 只能通过修改 `be.conf` 文件并重启 BE 生效 。# StarRocks 基础运维学习笔记