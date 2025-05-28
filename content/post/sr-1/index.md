---
title: "StarRocks 学习笔记 Day1"
description:  
date: 2025-05-28
image: sr.png
categories:
    - StarRocks
    - 数据库
---

## 数据架构演进
| 代际 | 特点 | 典型技术 | 数据类型 | 资源架构 |
|------|------|----------|----------|----------|
| 第1代 | 关系型数据库 | Oracle/MySQL | 结构化 | 专业设备共享存储 |
| 第2代 | MPP数仓 | Greenplum/Vertica | 结构化 | X86通用服务器MPP |
| 第3代 | Hadoop平台 | Hive/Hudi/Iceberg | 结构化/半结构化 | 通用服务器 |
| 第4代 | 智能湖仓一体 | StarRocks | 结构化/半结构化/非结构化 | 存算分离云原生 |

## StarRocks 核心优势
1. **极致性能**
   - 向量化执行引擎
   - CBO优化器+Runtime Filter
   - 单节点100亿行/秒处理速度
   - 查询延迟亚秒级

2. **实时能力**
   - 秒级数据更新可见
   - 单节点100MB/s写入速度
   - 支持主键更新模型

3. **灵活架构**
   - 多模型支持：星型/雪花/宽表模型
   - 存算分离架构（云原生）
   - 多表物化视图加速查询

4. **生态整合**
   - 外部数据目录：Hive/ES/MySQL/Hudi/Iceberg/Delta Lake
   - 计算集群隔离：多租户资源组
   - 无缝对接BI工具（Tableau/FineBI等）

## 核心场景
1. **实时分析**
   - 实时数据看板
   - A/B测试分析
   - 金融风控即时决策

2. **混合负载**
   - 高并发点查（TP99<1s）
   - Ad-hoc查询
   - 批量报表生成

3. **湖仓一体**
   - 直接查询数据湖（Iceberg/Hudi/Delta Lake）
   - 无需ETL预处理
   - 统一数据源（Single Source of Truth）

## 数据仓库 vs 数据湖
| 维度         | 数据仓库                | 数据湖                  |
|--------------|-------------------------|-------------------------|
| 数据格式     | 特有格式（优化性能）    | 通用格式（Parquet/ORC） |
| 存储成本     | 高（本地SSD）           | 低（S3/对象存储）       |
| 典型引擎     | StarRocks/ClickHouse    | Trino/Presto            |
| 优势场景     | 实时BI分析              | 机器学习/AI训练         |

## 部署能力
- **云原生支持**
  - AWS/GCP/Azure全平台兼容
  - Kubernetes/Helm自动化部署
- **扩展生态**
  - 数据集成：Flink/Kafka/Spark Connector
  - 权限管理：Kerberos/Apache Ranger
  - 监控体系：Prometheus/Grafana/Datadog

## 性能指标
- 综合查询速度比同类产品快3-5倍
- 支持数千用户并发访问
- 高并发场景QPS>10,000+
- 存储成本降低80%（对比传统MPP）
