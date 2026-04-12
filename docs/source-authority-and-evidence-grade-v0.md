# Source Authority Taxonomy v0 & Evidence Grade v0

> 用于 PancrePal-Wiki 及后续可复用医疗 LLM-Wiki 项目的来源治理基线。  
> 目标：在来源正式进入 wiki/card 层之前，完成信源类别、信息质量、入库策略与展示限制的统一判断。

---

## 1. 为什么需要这套分级

医疗知识系统会混合接收：
- 指南与专家共识
- 协会 / 学会 / 官方专业组织发布
- 同行评议论文
- 临床试验注册信息
- 医院官方内容
- 专业医生公开表达
- 新闻媒体与公开搜索结果
- 患者经验与社区帖子
- 图片 / 截图 / 视频 / 扫描 PDF 转录内容

如果不在入库前做分级与标识，这些来源会被错误地拉平成同一种知识，导致：
- 指南与患者经验混权
- 二手转述冒充原始证据
- 图像/视频解释结果被误当作确定事实
- 患者问答时无法区分证据强弱

因此，`source_authority` 与 `evidence_grade` 必须在正式入库前完成，而不是等 query 时临时判断。

---

## 2. Source Authority Taxonomy v0

该字段回答：**这条信息首先来自什么类型的信源？**

### 2.1 高权威原始信源

#### `guideline_consensus`
- 指南
- 专家共识
- 标准路径
- NCCN / CSCO / ESMO / ASCO / 中华医学会等

#### `society_association`
- 学会 / 协会 / 官方专业组织发布
- 不一定是正式指南，但具机构权威性

#### `clinical_trial_registry`
- ClinicalTrials.gov
- ChiCTR
- 官方试验注册平台

#### `peer_reviewed_paper`
- 同行评议论文
- 正式期刊文章

### 2.2 专业机构 / 专业个人信源

#### `hospital_official`
- 医院官网
- 科室公告
- 医院科普
- MDT 公告

#### `professional_doctor`
- 专业医生公开表达
- 医生讲座
- 医生公众号 / 视频号 / 专栏
- 但未上升为正式指南 / 机构共识

### 2.3 二级传播与大众传播

#### `news_media`
- 新闻媒体
- 医疗资讯媒体
- 行业报道
- 科技 / 健康媒体

#### `public_search`
- 公开搜索结果页
- 搜索聚合页
- 普通网页整理页

#### `commercial_material`
- 商业宣传
- 产品页
- 药企营销材料
- 医疗服务推广页

### 2.4 经验与社区层

#### `patient_report`
- 患者经验
- 家属记录
- 病友分享
- 患者旅程记录

#### `community_post`
- 非专业社区讨论
- 论坛、评论、问答区
- 群聊摘要

### 2.5 未知 / 待定

#### `unknown`
- 暂时无法识别
- 来源链不完整
- 截图转述无法确认原始出处

---

## 3. Source Authority Flags（辅助标记）

主类型之外，建议增加一个多值辅助字段：`source_authority_flags`。

可选值示例：
- `official`
- `peer_reviewed`
- `primary_source`
- `secondary_interpretation`
- `expert_opinion`
- `patient_experience`
- `commercial_bias_risk`
- `anonymous`
- `screenshot_only`
- `transcript_derived`

### 为什么需要 flags
同一个来源主类型不一定足够表达风险。例如：
- 一个医生讲课视频，主类型可能是 `professional_doctor`
- 但它还可能带：
  - `expert_opinion`
  - `transcript_derived`
  - `secondary_interpretation`

因此，主类型 + flags 的组合比单一枚举更实用。

---

## 4. Evidence Grade v0

该字段回答：**这条信息在当前系统中，可采信到什么程度？**

> 注：这里先采用项目内部实用分级，不直接等同于完整医学循证分级体系。

### `A1`
- 高权威 + 高稳定 + 直接证据
- 例如：
  - 最新正式指南
  - 高质量共识
  - 高可信原始规范文件

### `A2`
- 高权威，但存在范围限制或更新滞后
- 例如：
  - 较旧但仍有效指南
  - 高质量机构说明
  - 严肃综述 / 系统综述

### `B1`
- 较强证据，但不应单独作为最高级主结论
- 例如：
  - 同行评议论文
  - 高质量临床研究
  - 试验注册 + 初步结果
  - 专业医生系统性解读

### `B2`
- 中等可信，适合作为补充证据
- 例如：
  - 单中心研究
  - 回顾性研究
  - 医院官方科普
  - 医生公开视频 / 文章

### `C`
- 弱证据 / 经验性信息 / 二手整理
- 例如：
  - 新闻报道
  - 搜索结果页
  - 非正式科普整理
  - 社区总结帖

### `D`
- 仅经验 / 讨论 / 线索，不可直接作为主结论
- 例如：
  - 患者经验
  - 群聊转述
  - 未证实截图
  - 模糊来源视频切片

### `U`
- Unknown
- 暂时无法判断

---

## 5. `source_authority` 与 `evidence_grade` 的关系

这两个字段不是一回事。

### 示例 1
- 某医生公众号文章
- `source_authority = professional_doctor`
- `evidence_grade = B2`

### 示例 2
- 患者群截图转述某指南
- `source_authority = patient_report` 或 `community_post`
- `evidence_grade = D`
- 即便它声称来源是指南，也不能直接升格

### 示例 3
- ClinicalTrials.gov 条目
- `source_authority = clinical_trial_registry`
- `evidence_grade = B1` 或 `A2`
- 取决于只是注册信息，还是已有明确结果支持

---

## 6. 默认映射规则（v0）

这是初判映射，不代表最终裁决。

- `guideline_consensus` → `A1 / A2`
- `society_association` → `A2 / B1`
- `clinical_trial_registry` → `A2 / B1 / B2`
- `peer_reviewed_paper` → `B1`
- `hospital_official` → `B1 / B2`
- `professional_doctor` → `B2`
- `news_media` → `C`
- `public_search` → `C`
- `commercial_material` → `C / D`
- `patient_report` → `D`
- `community_post` → `D`
- `unknown` → `U`

---

## 7. 入库策略建议（与等级绑定）

### 可直接进入正式层
- `A1`
- `A2`

### 可进入正式层，但需提示 / 交叉验证
- `B1`
- `B2`

### 默认进入受限区
- `C`
- `D`
- `U`

也就是说：
- `C/D/U` 默认先进：
  - `draft/`
  - `experience/`
  - `controversy/`
  - `internal notes`
- 不应直接写入主结论页

---

## 8. 面向患者的显示策略建议

### 默认优先显示顺序
1. `A1`
2. `A2`
3. `B1`
4. `B2`

### `C / D / U` 的显示原则
- 不默认置顶
- 必须带限制语
- 不能伪装成标准建议
- 更适合作为：
  - 经验补充
  - 争议观点
  - 待核实信息

---

## 9. 与四阶段对象模型的关系

### `NormalizedSource`
记录：
- `source_authority_initial`
- `evidence_grade_initial`
- `grading_basis`

### `GovernedSource`
记录：
- `source_authority_final`
- `evidence_grade_final`
- `grade_reason`
- `publication_policy`
- `display_restriction`

也就是说：
- Normalize：初判
- Govern：定级

---

## 10. 设计原则（必须遵守）

1. 低质量来源不能直接覆盖高质量来源结论。  
2. 患者经验、社区帖子、公开搜索结果，默认不能直接成为主结论。  
3. 图片、截图、视频、扫描 PDF 等高推断风险来源，必须显式提高 `provenance_risk`。  
4. Query 层必须消费这些分级结果，默认优先高等级来源。  
5. 正式 wiki/card 页面至少应能追溯：
   - `source_authority`
   - `evidence_grade`
   - `provenance`
   - `review_status`

---

## 11. 可复用建议

这份分类与分级体系不应只用于 PancrePal-Wiki，也适合作为：
- 肿瘤知识系统
- 罕见病知识系统
- 慢病管理知识系统
- 医疗 Agent / Skill 的知识治理底层规范

