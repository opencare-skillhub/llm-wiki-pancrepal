# 患者目录模板 → MedicalCaseProfile Schema 映射 v0

> 目的：把 `templates/my_records/` 中的患者资料目录，与 `schemas/medical_case_profile.schema.json` 的字段建立稳定映射，供后续抽取器、编译器、报告生成器共用。

---

## 1. 映射原则

1. **目录决定主题语义**  
   文件所在目录优先决定其归属字段。

2. **文件内容决定字段细节**  
   例如病理报告在 `02_确诊信息/病理报告/`，但具体字段是否落入 `diagnosis` 或 `pathology_and_genomics.pathology`，由内容决定。

3. **一个文件可映射多个字段**  
   例如 NGS 报告可同时写入：
   - `pathology_and_genomics.genomic_findings`
   - `pathology_and_genomics.drug_response_genes`
   - `diagnosis`

4. **目录只是强提示，不是硬编码真相**  
   如果内容与目录冲突，以内容为准，但应保留目录路径作为 provenance。

---

## 2. 目录到字段的主映射

| 目录 | 主字段 | 次字段 |
|---|---|---|
| `01_基础信息/身份信息/` | `basic_info` | `patient_label` |
| `01_基础信息/医保与商业保险/` | `basic_info.insurance` |  |
| `01_基础信息/过敏史_既往史_家族史/` | `basic_info.allergies` / `basic_info.past_history` / `basic_info.family_history` |  |
| `02_确诊信息/首诊资料/` | `diagnosis` | `citations` |
| `02_确诊信息/病理报告/` | `pathology_and_genomics.pathology` | `diagnosis` |
| `02_确诊信息/分期评估/` | `diagnosis.stage` / `diagnosis.tnm` |  |
| `02_确诊信息/MDT结论/` | `diagnosis.mdt_summary` | `treatment_history` |
| `03_基因与病理详情/NGS报告/` | `pathology_and_genomics.genomic_findings` | `pathology_and_genomics.drug_response_genes` |
| `03_基因与病理详情/药敏性与药物代谢基因/UGT1A1/` | `pathology_and_genomics.drug_response_genes` | `medication_and_safety.safety_reminders` |
| `03_基因与病理详情/药敏性与药物代谢基因/DPYD/` | `pathology_and_genomics.drug_response_genes` | `medication_and_safety.safety_reminders` |
| `03_基因与病理详情/药敏性与药物代谢基因/TPMT_NUDT15/` | `pathology_and_genomics.drug_response_genes` | `medication_and_safety.safety_reminders` |
| `03_基因与病理详情/药敏性与药物代谢基因/CYP相关/` | `pathology_and_genomics.drug_response_genes` | `medication_and_safety.interaction_alerts` |
| `03_基因与病理详情/免疫与分子标志物/` | `pathology_and_genomics.pathology.ihc_markers` | `pathology_and_genomics.genomic_findings` |
| `04_治疗记录/手术/` | `treatment_history[]` |  |
| `04_治疗记录/化疗/` | `treatment_history[]` | `medication_and_safety.adverse_effects` |
| `04_治疗记录/放疗/` | `treatment_history[]` |  |
| `04_治疗记录/靶向/` | `treatment_history[]` | `medication_and_safety.current_medications` |
| `04_治疗记录/免疫治疗/` | `treatment_history[]` | `medication_and_safety.current_medications` |
| `04_治疗记录/临床试验/` | `treatment_history[]` | `follow_up_and_surveillance` |
| `05_影像资料/CT/` | `imaging_tracking.imaging_records[]` |  |
| `05_影像资料/MRI/` | `imaging_tracking.imaging_records[]` |  |
| `05_影像资料/PET-CT/` | `imaging_tracking.imaging_records[]` |  |
| `05_影像资料/影像光盘与原始DICOM说明/` | `imaging_tracking.dicom_metadata[]` | `imaging_tracking.imaging_records[]` |
| `06_检验指标与曲线/肿瘤标志物/` | `lab_and_marker_tracking.tumor_markers[]` |  |
| `06_检验指标与曲线/血常规/` | `lab_and_marker_tracking.blood_routine[]` |  |
| `06_检验指标与曲线/生化_肝肾功能/` | `lab_and_marker_tracking.liver_kidney_function[]` |  |
| `06_检验指标与曲线/炎症与凝血/` | `lab_and_marker_tracking.infection_and_coagulation[]` | `complications_and_risk` |
| `06_检验指标与曲线/趋势曲线导出/` | `lab_and_marker_tracking.*` | `report charts` |
| `07_用药方案与提醒/当前用药清单/` | `medication_and_safety.current_medications[]` |  |
| `07_用药方案与提醒/历史用药/` | `medication_and_safety.historical_medications[]` |  |
| `07_用药方案与提醒/不良反应与处理/` | `medication_and_safety.adverse_effects[]` | `complications_and_risk` |
| `07_用药方案与提醒/给药日历_复诊提醒/` | `medication_and_safety.safety_reminders[]` | `follow_up_and_surveillance.next_visit_date` |
| `08_并发症预防与风险管理/血栓_感染_出血风险/` | `complications_and_risk.known_risks` | `complications_and_risk.warning_signs` |
| `08_并发症预防与风险管理/胰外分泌不足与血糖管理/` | `complications_and_risk.prevention_plan` | `lab_and_marker_tracking.glucose_and_metabolism[]` |
| `08_并发症预防与风险管理/急症预警卡/` | `complications_and_risk.warning_signs` |  |
| `09_营养评估/体重与BMI变化/` | `nutrition.weight_history[]` / `nutrition.bmi` |  |
| `09_营养评估/PG-SGA_营养筛查/` | `nutrition.pg_sga_score` |  |
| `09_营养评估/营养干预记录/` | `nutrition.nutrition_plan` |  |
| `10_心理评估/HADS/` | `psychology.scales[]` |  |
| `10_心理评估/PHQ-9_GAD-7/` | `psychology.scales[]` |  |
| `10_心理评估/心理干预记录/` | `psychology.psychological_support_notes` |  |
| `11_随访与复发监测/随访计划/` | `follow_up_and_surveillance.follow_up_plan` |  |
| `11_随访与复发监测/复查记录/` | `follow_up_and_surveillance.surveillance_notes` | `imaging_tracking` / `lab_and_marker_tracking` |
| `11_随访与复发监测/复发_转移评估/` | `diagnosis.recurrence_status` / `diagnosis.metastasis_sites` | `follow_up_and_surveillance` |

---

## 3. 特殊规则

### 3.1 药物毒性基因
- `UGT1A1`：优先关联 `伊立替康`
- `DPYD`：优先关联 `5-FU / 卡培他滨`
- `TPMT / NUDT15`：优先关联 `硫嘌呤类`
- `CYP相关`：优先写入 `interaction_alerts`

### 3.2 标志物与趋势
- 同一指标多次记录时，合并为同一个 `measurement_series`
- `values[]` 按日期升序
- 超参考范围时填 `flag=high/low`

### 3.3 影像
- PDF/截图中的影像结论 → `imaging_records`
- DICOM 文件元数据 → `dicom_metadata`
- 若同次检查同时有 DICOM 与报告，应在后续 compiler 中做同次 study 合并

### 3.4 心理量表
- HADS / PHQ-9 / GAD-7 默认写入 `psychology.scales[]`
- 若量表结果带解释结论，也写入 `interpretation`

---

## 4. 后续编译器要求

后续 `capture -> normalize -> enrich -> govern -> publish/report` 应至少支持：

1. 目录路径映射
2. 文件类型映射（pdf/image/dicom/docx/xlsx）
3. 内容字段抽取
4. 多文件合并到同一病例 profile
5. 每个字段附带 `citations`

