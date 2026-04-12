# 参考项目深度分析（4个 LLM Wiki 相关仓库）

> 本文是工程参考分析，不替代主设计文档。唯一主设计文档仍是 `docs/architecture-v1.1.md`。

## 1. nvk/llm-wiki

### 1.1 真正的优势不只是“多 Agent”
这个项目最强的地方不是“5/8/10 个 agent 并行”，而是它把整个 wiki 工作流做成了**命令化、可恢复、可递进研究**的系统。

### 1.2 关键工程优点

#### A. Hub-and-Spoke 不是口号，而是工作编排结构
- hub 只做注册与调度，不存具体知识内容
- 每个 topic wiki 独立，避免全局 wiki 污染
- coordinator 负责：问题分解、轮次推进、gap 选择、结果汇编
- 子 agent 负责：不同研究角度 / 子命题 / 反证链路

#### B. `--min-time` 多轮研究机制非常重要
不是“一轮搜完就结束”，而是：
- 第一轮找主题与缺口
- 反思后选 top gaps
- 第二轮及以后继续钻取
- 用 progress score 决定是否提前终止

这个机制非常适合医疗领域的“争议问题”和“持续追踪”。

#### C. `resume` 设计成熟
通过 `.research-session.json` / 日志 / 最近更新文章恢复工作现场，而不是靠模型记忆。

#### D. `lint --fix` 变成迁移系统
它不是单纯报错，而是承担：
- frontmatter alias rewrite
- canonical placement 修复
- 老布局迁移
- 项目结构修复

这意味着 lint 既是健康检查，也是**架构演进通道**。

#### E. thesis mode 是医疗场景强需求
它把研究对象从“泛主题”收缩成“可证伪命题”：
- 变量
- 预测
- 证伪条件
- 支持证据
- 反对证据
- verdict

对于胰腺癌知识系统，这比普通 research 更适合处理：
- 某疗法是否有效
- 某营养支持是否有争议
- 某副作用处理策略是否可靠

#### F. query 的深度层次很清楚
- quick：只读 index
- standard：index → 相关页 → grep
- deep：全 index + 全文 + raw + sibling wiki

这种查询深度分层值得直接吸收。

### 1.3 应吸收到主设计中的点
- topic wiki 隔离
- 研究 session 持久化
- thesis-driven investigation
- lint 兼具迁移职责
- query depth levels
- append-only log
- `_index.md` 优先导航

---

## 2. SamurAIGPT/llm-wiki-agent

### 2.1 优势在“最小闭环很完整”
这个项目虽然不如 nvk 复杂，但它把最基础的 LLM Wiki 闭环做得很清楚：
- `raw/`
- `wiki/index.md`
- `wiki/log.md`
- `wiki/overview.md`
- `entities/`
- `concepts/`
- `syntheses/`
- `graph/`

### 2.2 关键优点

#### A. `overview.md` 很有价值
除了 index/log 外，它有一个持续更新的总览页，作为“全局活体综述”。
这对医疗 wiki 很重要，因为患者/家属很多时候不是查一个点，而是需要先看全貌。

#### B. ingest prompt 的 JSON 输出约束明确
`tools/ingest.py` 要求模型返回结构化 JSON：
- source page
- index entry
- overview update
- entity pages
- concept pages
- contradictions
- log entry

这说明它把 ingest 拆成了多目标写入，而不是只生成一页。这个思想值得保留。

#### C. deterministic lint + semantic lint 的双层设计
`tools/lint.py` 里分成：
- 确定性检查（orphan / broken link / missing entity）
- 语义性检查（contradiction / stale / data gaps）

这是一种非常实用的结构：先低成本硬规则，再高成本语义检查。

#### D. syntheses 作为 query 写回区
把高价值问答归档到 `wiki/syntheses/`，而不是直接混入 concepts/entities，很适合保留对话产物。

#### E. graph 输出独立
图谱不是核心真相层，而是派生物，放到 `graph/`。这个边界清晰。

### 2.3 应吸收到主设计中的点
- `overview.md` / 医疗总览页
- ingest 的多目标结构化输出
- deterministic lint + semantic lint 分层
- `syntheses/` 或 query archive 区
- graph 作为派生层而非知识主层

---

## 3. astro-han/karpathy-llm-wiki

### 3.1 优势在“极简而不失真”
这个项目的价值不是功能多，而是它非常贴近 Karpathy 原始思路，避免过度工程化。

### 3.2 关键优点

#### A. 只有三层：raw / wiki / skill(schema)
它提醒我们：
- `raw/` 是原料
- `wiki/` 是成品知识层
- `SKILL.md` 是行为规范层

#### B. topic 只有一级目录
`wiki/<topic>/<article>.md`，不鼓励过深嵌套。对长期维护有利。

#### C. query 默认不写文件
只有明确要求 archive 时才写入。这个边界值得保留，避免 query 污染 wiki。

#### D. lint 分为 auto-fix 与 report-only
- 索引不一致、路径错误、缺失项 → 可修
- 事实冲突、知识过期、判断性问题 → 只报告

这种边界非常重要，特别适合医疗场景。

#### E. 引用与链接路径规范清晰
它对 wiki 内相对路径、对话中 project-root-relative 路径做了明确区分。这个细节对 agent 可用性很重要。

### 3.3 应吸收到主设计中的点
- 保持目录层级克制
- query 默认不落盘
- archive 写入单独区域
- lint 的 auto-fix/report-only 严格分层
- schema 文件承担行为协议职责

---

## 4. Ar9av/obsidian-wiki

### 4.1 优势在“知识治理工程化”
这个项目最强的不是某个单点功能，而是把 wiki 当作一个需要长期治理的系统。

### 4.2 关键优点

#### A. `.manifest.json` 增量跟踪很强
它不是简单比较 mtime，而是：
- path
- modified_at
- content_hash
- pages_created
- pages_updated
- source_type
- stats

这让 append ingest 更可靠，也适合医疗资料持续更新。

#### B. provenance tracking 非常适合医疗
它明确区分：
- extracted
- inferred
- ambiguous

并要求页面写 `provenance:` frontmatter block。这个对于医疗知识极其关键，应该强吸收。

#### C. `summary:` frontmatter 是低成本检索关键
不是每次 query 都开全文，而是优先读 summary/index。这个非常适合大规模 wiki。

#### D. cross-linker 是独立 skill
它把“补链接”从 ingest / lint 中拆出来，变成独立维护动作。这个拆分很成熟。

#### E. tag taxonomy 作为显式治理对象
标签不是随便写，而是有 canonical taxonomy，并可被 audit。医疗 wiki 非常需要这个能力。

#### F. status / delta / insights 三件套
它不只看“有没有问题”，还看：
- 还有哪些未 ingest
- vault 当前状态如何
- 图谱中心页、bridge pages、fragmented tag clusters

这比普通 lint 更像运营仪表盘。

#### G. `_raw/` staging 思路很好
把草稿和正式知识分离，降低脏数据直接污染 wiki 的风险。

### 4.3 应吸收到主设计中的点
- `.manifest.json` 增量账本
- provenance block
- `summary:` frontmatter
- 独立 `cross-linker`
- taxonomy 治理
- status/delta/insights 模块
- `_raw/` staging 目录

---

## 5. 四个项目合并后的设计建议

### 5.1 最应吸收的“共同稳定能力”
1. Raw immutable
2. index/log 优先导航
3. ingest/query/lint 主闭环
4. append-only log
5. query depth levels
6. thesis-driven research
7. query archive / syntheses
8. deterministic + semantic lint
9. provenance tracking
10. manifest-based incremental ingest
11. structural guardian / canonical placement
12. dedicated cross-linker
13. weekly/monthly self-improvement with regression sets

### 5.2 不应直接照搬的部分
1. 不要一开始把 PancrePal 做成大而全的通用 hub
2. 不要过早引入过多输出形态（slides/report/playbook 全上）
3. 不要把患者入口直接等同于 research pipeline
4. 不要让 query 默认污染正式 wiki

### 5.3 适合 PancrePal 的吸收顺序
- 第一优先：raw/wiki/schema/log/index/manifest/provenance
- 第二优先：thesis mode / query depth / deterministic+semantic lint
- 第三优先：cross-linker / status / insights / project output
- 第四优先：multi-agent research orchestration

