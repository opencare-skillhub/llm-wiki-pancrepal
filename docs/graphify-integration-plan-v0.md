# Graphify 集成与医学 AST 迁移计划 v0

> 目的：把 graphify 的结构化抽取思想迁移为 PancrePal-Wiki 的“医学 AST 派生层”。
> 说明：本文件是专项参考，主设计仍以 `docs/architecture-v1.1.md` 为准。

## 1) 结论（先给结果）

- 可以吸收 graphify 的“分层结构化抽取”方法。
- 不能让 graph 覆盖 wiki 主事实层；graph 只能是派生层。
- 医疗 AST 的最小落地对象：`patient_case -> diagnosis/treatment/adverse_event/follow_up/outcome`。

## 2) 建议的数据模型（最小）

### Node
- `node_id`
- `node_type`（patient_case/diagnosis/treatment/...）
- `label`
- `attributes`（JSON）
- `provenance`（extracted/inferred/ambiguous）
- `source_id`
- `evidence_span`

### Edge
- `edge_id`
- `edge_type`（has_diagnosis/received_treatment/...）
- `from_node_id`
- `to_node_id`
- `confidence`
- `source_id`
- `evidence_span`

## 3) 与治理系统的强耦合点

- 入库前分级：`source_authority + evidence_grade`
- 图谱节点继承来源等级，不允许“低等级来源高置信覆盖”
- retract/restrict 后，受影响 node/edge 必须标记并可重编译

## 4) 本地执行状态（当前）

尝试执行：
- `git submodule add https://github.com/safishamsi/graphify.git reference_repo/graphify`

当前环境报错（网络连通性）：
- `Failed to connect to github.com port 443`

因此本轮未完成 graphify 子模块落库与本地部署验证。

## 5) 环境恢复后的一键步骤

```bash
git submodule add https://github.com/safishamsi/graphify.git reference_repo/graphify
git submodule update --init --recursive

# 进入参考仓库阅读其 README / 示例
cd reference_repo/graphify
```

## 6) POC 验证目标（不进入正式开发）

1. 选 1 份结构化程度较高的病历样本（脱敏）  
2. 抽取 20~40 个节点、30~60 条关系  
3. 生成 `graph/medical_ast/snapshots/<date>.json`  
4. 抽样验证 10 条边是否可回溯到 `source_id + evidence_span`

