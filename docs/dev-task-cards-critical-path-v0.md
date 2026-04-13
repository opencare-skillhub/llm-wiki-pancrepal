# Critical Path 开发任务卡 v0

> 用于直接执行的最小任务卡。每张卡包含：目标、输入、产出、验收标准。

---

## Card 01 — 定义核心 Schema

**目标**
- 固化 source 生命周期对象与治理枚举。

**输入**
- `docs/architecture-v1.1.md`
- `docs/source-authority-and-evidence-grade-v0.md`
- `docs/governance-retraction-principles.md`

**产出**
- `schemas/source_input.schema.json`
- `schemas/captured_source.schema.json`
- `schemas/normalized_source.schema.json`
- `schemas/enriched_source.schema.json`
- `schemas/governed_source.schema.json`
- `schemas/governance.enums.json`

**验收**
- schema 可校验通过
- 枚举包含 authority/grade/policy/restriction/review/retention
- 对象字段满足四阶段最小必填约束

---

## Card 02 — 初始化目录骨架

**目标**
- 建立 raw/wiki/cards 的最小结构与治理分区。

**输入**
- `docs/architecture-v1.1.md`

**产出**
- `_raw/`
- `raw/`, `raw/_restricted/`, `raw/_archive/`
- `wiki/`, `wiki/draft/`, `wiki/syntheses/`
- `cards/`, `cards/draft/`
- `logs/`, `evolution-log/`

**验收**
- 目录一次性创建成功
- 结构与设计文档一致
- 后续 ingest 无需再动态补关键目录

---

## Card 03 — 建立 Manifest 最小账本

**目标**
- 支持 source_id 主键、hash 变更检测、状态追踪。

**输入**
- Card 01 schema
- Card 02 目录

**产出**
- `.manifest.json` 初始化模板
- `manifest` 读写模块（可用脚本或模块）

**验收**
- 可新增 source 记录
- 可更新 `capture/normalize/enrich/govern/compiler` 状态
- 可保存 `raw_hash/derived_hash/govern_hash`

---

## Card 04 — 实现 Capture Adapter（URL/File/Text）

**目标**
- 将输入统一落入 raw 层并生成 `CapturedSource`。

**输入**
- Card 01/03

**产出**
- `capture_adapter`（url/file/pasted_text）
- raw 文件/原件落盘
- `source_id` 与 `hash_sha256`

**验收**
- URL/File/Text 三种输入均可 capture
- `.manifest.json` 写入成功
- 失败场景有明确错误与日志

---

## Card 05 — 实现 Normalize Engine（先文本）

**目标**
- 从 `CapturedSource` 生成 `NormalizedSource`。

**输入**
- Card 04 输出

**产出**
- `normalize_engine`（先支持 markdown/txt/url 抽取）
- `content_type/modality/language/source_type` 初判

**验收**
- 能产出 `NormalizedSource` 有效对象
- 至少一个主内容字段存在（extracted_text/ocr/transcript/visual_notes）
- 初判字段可写回 manifest

---

## Card 06 — 实现 Enrich + Govern（最小规则）

**目标**
- 产出知识候选与治理决策。

**输入**
- Card 05 输出

**产出**
- `enrich_engine`：entities/concepts/claims/tags
- `govern_engine`：authority/grade/risk/policy/restriction

**验收**
- `EnrichedSource`、`GovernedSource` 均可生成
- 未分级内容不能标记 publish_direct
- 高风险低等级默认进入受限路径

---

## Card 07 — 实现 Publish Engine

**目标**
- 将治理后的内容写入 wiki/cards，并更新索引日志。

**输入**
- Card 06 输出

**产出**
- `publish_engine`
- `wiki/index.md` / `wiki/log.md` / `wiki/overview.md` 更新
- `cards/*` 写入

**验收**
- 支持 `draft/publish/restricted` 路径
- 每次写入有 `source_id/trace_id` 追踪信息
- manifest 的 pages/cards 更新列表正确

---

## Card 08 — 最小 Query（带引用）

**目标**
- 基于 wiki 返回回答，并提供可追溯 citations。

**输入**
- Card 07 结果

**产出**
- `query` 最小能力
- `citation_id -> source_id` 可回溯

**验收**
- 回答包含 citations
- 可从 citation 追到 raw source
- retract/restricted 来源不会默认进入主回答

---

## Card 09 — 端到端验收用例

**目标**
- 跑通 “URL ingest → compile → query → trace” 全链路。

**输入**
- Card 01-08

**产出**
- E2E 测试脚本/手工步骤文档

**验收**
- 单条 URL 可完成全链路
- 输出可追溯
- 治理约束生效（未分级/高风险低等级不会直出）

