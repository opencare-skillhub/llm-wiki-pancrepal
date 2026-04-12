# Development Tasklist v0（从设计树导出）

> 目标：先实现可审计、可演化的 LLM-Wiki 内核，再扩展到多入口与智能体生态。

## M1：基础内核（先打通）

- [ ] 定义 Schema
  - [ ] `source_input`
  - [ ] `CapturedSource`
  - [ ] `NormalizedSource`
  - [ ] `EnrichedSource`
  - [ ] `GovernedSource`
  - [ ] `source_authority / evidence_grade / policy` 枚举
- [ ] 落地目录骨架
  - [ ] `_raw/`
  - [ ] `raw/`
  - [ ] `raw/_restricted/`
  - [ ] `raw/_archive/`
  - [ ] `wiki/`
  - [ ] `cards/`
- [ ] 实现 `capture_adapter`（URL / file / pasted text）
- [ ] 实现 `.manifest.json` 最小账本（`source_id + hash + status`）
- [ ] 落地 `index.md / log.md / overview.md` 自动更新

### M1 验收标准
- [ ] 一条 URL 可完成 capture
- [ ] raw + manifest + index/log 正确更新
- [ ] `source_id` 可追踪

---

## M2：编译与治理

- [ ] 实现 `normalize_engine`（文本/OCR/转录分支）
- [ ] 实现 `enrich_engine`（entities/concepts/claims/contradictions）
- [ ] 实现 `govern_engine`（authority/grade/risk/policy）
- [ ] 实现 `publish_engine`（draft/publish/restricted 路径）
- [ ] 实现 `query` 最小版（带 citations）
- [ ] 实现 `lint` 双层（deterministic + semantic）

### M2 验收标准
- [ ] source 从 raw 编译到 wiki/card
- [ ] query 可返回 citations
- [ ] citation 可追溯到 `source_id`

---

## M3：长期演化能力

- [ ] 实现周期 review（weekly/monthly）
- [ ] 实现 retract pipeline（blast radius + cleanup + optional recompile）
- [ ] 实现 restrict/archive/delete 策略联动
- [ ] 实现 traceability chain 全链追踪
- [ ] 接入 OpenClaw/Hermes 最小 skill contract

### M3 验收标准
- [ ] 高风险低等级来源可 review → retract → restrict/archive/delete
- [ ] 全链审计可回放

---

## 本周执行建议（可并行）

- Day 1: Schema + 目录 + Manifest
- Day 2: Capture Adapter
- Day 3: Normalize（先文本）
- Day 4: Enrich + Govern（最小规则）
- Day 5: Publish + Index/Log + 最小 Query

