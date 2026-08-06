# 数据录入流程

本说明用于指导 agent 如何将已确认的原始资料映射到本地数据库。它是记录规范，
不是医疗建议。

## 一条事实只放一个位置

| 已确认的信息 | 写入位置 |
| --- | --- |
| 稳定的疾病类型、最早症状日期、确诊记录链接 | `disease_profile` |
| 一次真实输注 | `injections` |
| 一次真实抽血、门诊、操作或影像检查 | `checkups` |
| 该次检查中的一个原子化结果 | `checkup_results` |
| 指标本身的固定定义 | `metric_definitions` |
| 报告原文或私有附件引用 | `checkup_reports` |
| 医生明确确认的结论 | `checkup_assessments` |
| 一次大型复查及其子检查 | `review_episodes` + `review_components` |
| 日常症状 | `symptom_logs` |
| 饮食、睡眠、压力等可能因素 | `factor_terms` + `factor_logs` |

同一次抽血即使同时服务于输注与大型复查，也不能复制两条检查记录；只创建一条
`checkups(kind='lab')`，再按需关联到输注和/或复查。

## 化验单录入流程

1. 确认原始报告，并创建或找到对应的唯一一条 `lab` 检查记录。
2. 只有在真实采样/完成时间明确时，才把该检查标记为完成。
3. 每一项需要保留的数值，先确认已有指标定义；没有时先由用户明确采用后注册。
4. 将各项结果写到同一条检查记录，保留报告原始单位、参考范围、异常标记和必要来源备注。
5. 叙述性所见写入带版本的 `checkup_reports`，不能改写成模型生成的医学结论。

## 自定义指标

仅在用户根据真实报告或照护计划明确采用某个指标后，才使用 `metric-add`：

```bash
python3 scripts/ibd_db.py metric-add \
  --code infliximab_level \
  --name "英夫利西单抗浓度" \
  --value-type numeric \
  --default-unit "μg/mL"
```

- `code` 必须是稳定的小写 `snake_case`，创建后不改名。
- 测量数值使用 `numeric`，文本型结果使用 `qualitative`。
- `default_unit` 可选，只是录入提示；不得换算或覆盖化验单上的实际单位。
- 不得自行填写参考范围或异常标记，只保留报告提供的内容。
- 不再需要新录入时使用 `metric-deactivate --code <code>`；历史结果保留，且该指标不能继续写入新结果。

## 大型复查和医生评估

多项检查构成的大型复查应创建一个 `review_episodes` 父记录，再关联实际发生的抽血、
内镜、影像、病理或门诊检查。医生评估只能记录在对应已完成、承担评估的检查上，且必须
来自明确确认的原始措辞；症状、化验数值和 agent 推断均不得自动生成医生评估。
