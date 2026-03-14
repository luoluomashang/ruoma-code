---
name: xushikj-hudong
description: |
  叙事空间创作系统·互动式写作模块。
  用户实时引导剧情方向，帮回系统为核心操控手段。
  复用 sub-agent 架构，章节级粒度输出。
metadata:
  version: 1.0.0
  parent: xushikj-chuangzuo
  step: 10
  triggers:
    - 互动写作
    - 开始互动
    - 帮回
---

# 叙事空间创作系统·互动式写作模块

与流水线写作（xushikj-xiezuo）并行的写作范式。用户实时引导剧情方向，帮回系统为核心操控手段，每章写完由用户确认落盘。

## 架构

```
orchestrator ←→ 用户交互循环：
  ├─ 接收用户指令（自然语言 / 帮回指令）
  ├─ 解析为结构化场景参数
  ├─ KB 切片 → writer-agent（整章，3000-4000字）→ 呈现给用户
  ├─ 用户审阅："改XX" / "重写" / "OK落盘"
  └─ 落盘 → maintenance-agent（KB diff + 概括 + 质量评估）
```

## 前置依赖

| 步骤 | 产出 | 必须 |
|------|------|------|
| 1 | outline/one_sentence.md | 是 |
| 2 | outline/one_paragraph.md | 是 |
| 3 | outline/characters.md | 是 |
| 5 | outline/character_arcs.md | 是 |
| 7 | knowledge_base.json（精简初始化） | 是 |

**不需要**：四页大纲（步骤6）、场景清单（步骤8）、场景规划（步骤9）

## 产出文件

与 xushikj-xiezuo 相同目录结构：

```
.xushikj/
├── chapters/chapter_*.md        # 落盘后的正式章节
├── kb_diffs/chapter_*_diff.json # KB 变更记录
├── summaries/
│   ├── chapter_*_summary.md     # 章节概括
│   └── summary_index.md         # 全书概要索引
└── quality_reports/chapter_*_quality.md
```

## 共享资源（不重复建设）

| 资源 | 路径 | 说明 |
|------|------|------|
| writer sub-agent 提示词 | xushikj-xiezuo/references/chapter-writer-sub-agent-prompt.md | 直接复用，仅参数不同 |
| KB diff schema | xushikj-xiezuo/references/kb-diff-schema.md | 直接复用 |
| KB diff 应用脚本 | xushikj-xiezuo/scripts/apply_kb_diff.py | 直接复用 |
| summary sub-agent 提示词 | xushikj-xiezuo/references/summary-sub-agent-prompt.md | 直接复用 |
| 全部 config 文件 | xushikj-chuangzuo/config/*.yaml | 直接复用 |

## 落盘机制

- **章节写完 = 草稿态**，仅呈现给用户，不触发任何后处理
- **用户说"OK落盘"**才触发 maintenance agent，一次性完成：
  1. KB diff 验证与应用
  2. 章节概括 + summary_index.md 更新
  3. 八维度质量评估
  4. state.json 章节号 +1

## 帮回系统集成

帮回系统在互动模式中是**核心交互手段**：

- **甲类指令**（8个即时指令）：转化为 bangui_context 注入 writer sub-agent
- **甲类·选项模式**：加 `[选项]` 后缀，orchestrator 先构思方向让用户选择
- **乙类**（章节规划）：orchestrator 在主进程内生成规划方案
- **丙类**（分析诊断）：orchestrator 在主进程内执行分析

## 概要注入策略

| 阶段 | 条件 | 注入方式 |
|------|------|---------|
| 早期 | 已落盘章节 ≤ 3 | 仅前章末尾 500 字衔接 |
| 中期 | 已落盘章节 > 3 且概要总字数 < 4000 | 完整概要 + 前章末尾 |
| 后期 | 概要总字数 ≥ 4000 | 完整概要注入 sub-agent，由其自行压缩理解 |

**关键约束**：orchestrator 绝不压缩概要——前文剧情已存在于主进程对话历史，随 compact 自动压缩。

## 与流水线模式的关键差异

| | 流水线 (xushikj-xiezuo) | 互动 (xushikj-hudong) |
|--|--|--|
| 场景规划来源 | 预写文件 scene_plans/*.md | 用户即时指令 + 帮回 |
| 四页大纲 | 必须 | 不需要 |
| writer sub-agent | 相同 | 相同（复用） |
| 落盘时机 | 自动（章节完成即落盘） | 用户触发 |
| 概要注入 | 固定最近 3 章 | 阶梯式 |
| 帮回系统 | 挂载但无自然触发点 | 核心交互手段 |
| orchestrator 状态 | 无状态 | 有状态（累积本章草稿 + 对话） |

## Notes

- 互动式写作与流水线写作地位平等，在步骤 5 完成后由用户选择
- state.json 中 `writing_mode: "interactive"` 标记互动模式，断点续做时自动路由
- 步骤 7 知识库为精简初始化：只初始化人物实体 + 基本关系，地点/物品在写作中由 KB diff 动态创建
