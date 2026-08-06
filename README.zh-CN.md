# IBD Companion

[English](README.md) | [简体中文](README.zh-CN.md)

IBD Companion 是一个本地优先的 OpenClaw skill，用于私密整理个人
IBD（炎症性肠病）/克罗恩病记录。它在一个 SQLite 数据库中，将原始观察、
治疗事件、检查、大型复查、医生明确结论和个人症状基线分别保存，避免混淆。

它是个人记录与计划辅助工具，不用于诊断、开药或替代紧急医疗服务。

## 主要功能

- 以追加方式保存症状观察，并保留更正历史
- 从原始记录实时生成每日概览，不重复存储第二份记录
- 规范化饮食、睡眠、压力等可能因素
- 管理输注排期、改期、完成状态和计划来源
- 单项检查可同时关联输注与大型复查
- 保存结构化检查指标和带版本历史的叙述性报告
- 保存带版本历史、且经过明确确认的医生评估
- 支持草稿、候选、已确认和历史个人症状基线
- 使用自然月生成复查计划，并避免重复创建复查
- 可选外部提醒，默认保持关闭
- 通过增量迁移支持至数据库 schema V6

## 隐私模型

SQLite 数据库是唯一事实来源，默认保存在仓库之外：

```text
${IBD_DB_PATH:-${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/private/ibd/ibd.sqlite3}
```

不要向 Git 提交数据库、附件、导出的报告、真实病历或可识别个人身份的健康信息。
仓库提供的 `.gitignore` 会拦截常见数据库文件和私有数据目录，但每次提交前仍应
人工检查待提交内容。

## 环境要求

- Python 3.10 或更高版本
- Python 标准库中的 SQLite 支持
- 使用 skill 时需要 OpenClaw；命令行脚本也可以独立运行

不需要安装第三方 Python 包。

## 安装

将仓库克隆到 OpenClaw workspace 的 skills 目录：

```bash
git clone https://github.com/denniscriss/ibd-companion.git \
  ~/.openclaw/workspace/skills/ibd-companion
```

初始化或迁移私有数据库：

```bash
python3 scripts/ibd_care.py init
```

如需使用其他数据库位置，可为命令设置 `IBD_DB_PATH`：

```bash
IBD_DB_PATH=/path/to/private/ibd.sqlite3 python3 scripts/ibd_care.py init
```

## 命令概览

基础跟踪脚本负责症状、因素、输注、检查、检查结果和内部提醒记录：

```bash
python3 scripts/ibd_db.py --help
```

照护上下文扩展负责疾病档案、治疗与复查计划、复查层级、报告、已确认评估和症状基线：

```bash
python3 scripts/ibd_care.py --help
```

使用命令前请先阅读 `SKILL.md`。其中规定了安全边界、唯一事实来源和明确确认要求，
用于防止不同医学含义的数据被混在一起。

## 测试

测试只使用临时数据库，不会接触默认的私有数据库：

```bash
python3 -m unittest discover -s scripts -p 'test_ibd_*.py' -v
```

## 仓库结构

```text
SKILL.md                 OpenClaw 工作流程与安全规则
references/schema.md     数据模型与迁移说明
scripts/ibd_db.py        基础 SQLite 跟踪脚本和命令行入口
scripts/ibd_care.py      照护上下文扩展和命令行入口
scripts/test_ibd_*.py    使用临时数据库的自动化测试
```

## 安全边界

- 保留来源原文和未知值，不编造医学事实。
- 不根据症状、检查指标或因素比较推断疾病活动或因果关系。
- 不依据自动分析改变药物或治疗周期。
- 只有在明确、已确认的来源支持时，才记录医生评估。
- 出现紧急或危险信号时应寻求专业医疗帮助，而不是继续常规趋势分析。
- 外部通知必须由用户另行批准并配置。

## 许可证

MIT，详见 `LICENSE`。
