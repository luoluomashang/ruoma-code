---
name: xushikj-xiezuo
description: |
  叙事空间创作系统·写作模块。执行步骤10：逐章创作。
  采用 orchestrator + 2 sub-agents 架构，集成帮回辅助系统、双保险机制、质量评估。
metadata:
  version: 2.0.0
  parent: xushikj-chuangzuo
  step: 10
  triggers:
    - 写第N章
    - 开始写作
    - 继续写
---

# 叙事空间创作系统 - 写作模块

## 概述

执行宏观十二工作流的第十步：逐章创作。采用 **orchestrator + 2 sub-agents** 架构：

- **Orchestrator**（主进程，~5-6K tokens）：调度、KB 切片、diff 应用、质量评估、帮回系统
- **Chapter Writer Sub-agent**（~10-15K tokens）：章节正文创作 + KB diff 生成
- **Summary Sub-agent**（model="haiku"，~4K tokens）：每章概括

此架构确保 orchestrator 上下文**不随章节增长膨胀**，通过 KB 切片控制 sub-agent 上下文上限。

## 架构流程

```
orchestrator
  ├─ 读 state.json + scene_plan
  ├─ 构建 KB 切片（仅相关实体）
  ├─ 组装参数 → 启动 chapter-writer sub-agent
  │     ├─ 写 chapters/chapter_XX.md
  │     ├─ 写 kb_diffs/chapter_XX_diff.json
  │     └─ 返回 ≤150 tokens 确认
  |-─ 启动 judge sub-agent 进行 AI 味审查
  │     ├─ 读 chapters/chapter_XX.md
  │     ├─ 写 reviews/chapter_XX_judge_report.md
  │     └─ 返回裁决结果（PASS/REJECT）
  ├─ 验证 + 应用 KB diff → 增量更新 knowledge_base.json
  ├─ 启动 summary sub-agent（model="haiku"）
  │     ├─ 写 summaries/chapter_XX_summary.md
  │     ├─ 更新 summary_index.md
  │     └─ 返回 ≤50 tokens 确认
  ├─ 质量评估（读取章节文件）
  ├─ 更新 state.json + pending.md
  └─ 向用户报告
```

## 前置依赖

| 依赖 | 必要性 | 说明 |
|------|--------|------|
| 步骤8-9（场景规划） | 必须 | 逐章创作基于场景规划 |
| `knowledge_base.json` | 必须 | KB 切片和一致性校验 |

必须确认以下文件存在：
- `.xushikj/scenes/scene_list.md` - 场景清单
- `.xushikj/scenes/scene_plans/` - 各场景规划
- `.xushikj/knowledge_base.json` - 动态实体知识库
- `.xushikj/outline/volume_{V}_four_pages` - 四页大纲（宏观参考）

## 产出文件

| 文件路径 | 说明 |
|----------|------|
| `.xushikj/chapters/chapter_XX.md` | 各章正文（sub-agent 产出） |
| `.xushikj/reviews/chapter_XX_judge_report.md` | AI 味审查报告（judge sub-agent 产出） |
| `.xushikj/kb_diffs/chapter_XX_diff.json` | KB 变更记录（sub-agent 产出） |
| `.xushikj/summaries/chapter_XX_summary.md` | 各章概括（summary sub-agent 产出） |
| `.xushikj/summaries/summary_index.md` | 概括索引（summary sub-agent 维护） |
| `.xushikj/quality_reports/chapter_XX_quality.md` | 质量评估报告（orchestrator 产出） |
| `.xushikj/knowledge_base.json` | 每章完成后增量更新 |
| `.xushikj/state.json` | 更新当前章节进度 |

## 依赖文件（references）

| 文件 | 用途 |
|------|------|
| `references/chapter-writer-sub-agent-prompt.md` | 章节写作 sub-agent 提示词模板 |
| `references/summary-sub-agent-prompt.md` | 章节概括 sub-agent 提示词模板 |
| `references/kb-diff-schema.md` | KB diff 格式规范与操作语义 |

## Orchestrator 加载的配置

| 配置文件 | 用途 | 加载时机 |
|----------|------|----------|
| `config/bangui_modes.yaml` | 帮回系统模式定义 | 帮回系统触发时 |
| `config/quality_dimensions.yaml` | 八维度质量评估标准 | 章节完成后评估时 |

**以下配置由 sub-agent 自行读取**（不进入 orchestrator 上下文）：
- `config/writing_rules.yaml`、`config/style_rules.yaml`、`config/content_limits.yaml`
- `config/meta_rules.yaml`、`config/declarations.yaml`、`config/golden_opening.yaml`
- `config/safety_guard.yaml`

## 章节生成流程

### 1. 准备阶段（orchestrator）

```
1.1 读取 state.json + memory.md
1.2 读取场景规划，提取涉及角色 IDs
1.3 构建 KB 切片（仅相关实体，不持有 KB 全量）
1.4 提取最近 3 章概括、活跃伏笔清单
1.5 组装 instruction package
```

### 2. 章节创作（chapter-writer sub-agent）

Sub-agent 接收 instruction package 后：
```
2.1 读取场景规划 + 配置文件 + 前章末尾 500 字
2.2 声明注入（根据 sensitivity）
2.3 执行写作（writing_rules > style_rules > content_limits）
2.4 HC1-HC5 自检
2.5 生成 KB diff → 写入 kb_diffs/chapter_XX_diff.json
2.6 写入 chapters/chapter_XX.md
2.7 返回确认（≤150 tokens）
```
### 2.5. 启动裁判 Sub-agent 进行 AI 味审查（judge sub-agent）

Sub-agent 接收 chapters/chapter_XX.md 后：
```
2.5.1 读取chapters/chapter_XX.md
2.5.2 执行ai味审查（基于审查矩阵，逐段扫描）
2.5.3 生成诊断报告（高亮触雷点，整体诊断，最终裁决）
2.5.4 写入 reviews/chapter_XX_judge_report.md
2.5.5 返回裁决结果（PASS/REJECT）
```

### 3. KB Diff 应用（orchestrator）

```
3.1 读取 diff 文件
3.2 读取 knowledge_base.json（内存中操作）
3.3 按操作语义逐项应用（update/append/create/evolve/plant/resolve）
3.4 执行一致性校验（死亡复活/物品冲突/地点矛盾/能力超纲/ID冲突）
3.5 一次性 Write 保存
3.6 同步更新 pending.md（如有伏笔变更）
```

### 4. 章节概括（summary sub-agent, model="haiku"）

**每章触发**，产出本章独立概括。

### 5. 质量评估（orchestrator）

读取章节文件，执行八维度自评，保存报告。

### 6. 状态更新与用户确认（orchestrator）

更新 state.json，向用户展示进度和质量摘要。

## 帮回辅助系统

保留在 orchestrator 中（交互式，需用户输入）。

### 即时行动/对话辅助（类别甲）

| 指令 | 模式 | 风格 |
|------|------|------|
| `帮回主动1` | 直接行动 | 果断、目标明确、情感外放 |
| `帮回主动2` | 策略行动 | 巧妙、引导性、体现智慧 |
| `帮回被动1` | 外部压力 | 犹豫、退让、策略性隐忍 |
| `帮回被动2` | 内在情感 | 依赖、脆弱、寻求慰藉 |
| `帮回黑暗1` | 心理暗黑 | 掌控欲、操控、冷漠、病态执念 |
| `帮回黑暗2` | 行动暗黑 | 侵略性、占有性、挑战禁忌 |
| `帮回推进1` | 宏观旁白 | 戏剧化、关键转折、渲染氛围 |
| `帮回推进2` | 微观旁白 | 细腻、感官细节、心理活动 |

### 章节规划（类别乙）/ 分析与诊断（类别丙）

同前版本，功能不变。

## 双保险机制

### 保险一：前置声明

三级声明体系（L1/L2/L3），由 sub-agent 根据 sensitivity 自动注入。重试时自动升级。

### 保险二：输出完整性验证

五项健康检查（HC1-HC5），由 sub-agent 自检。三级响应（PASS/WARN/HALT）。

HALT 时 orchestrator 向用户提供四个选项（重试/调整敏感度/跳过/修改指令）。

**绝不执行**：自动降低描写尺度、删除用户描写规范、建议换表达、用模糊替代具体。

## 向后兼容

- **旧项目无 snapshot 字段**：orchestrator 首次运行时自动为所有角色补空值
- **旧项目无 summary_state 新字段**：根据 chapter_length 自动推算 summary_word_limit 并补充
- **旧格式概括文件**（`group_XX_chNN-MM.md`）：保留不动，新概括按 `chapter_XX_summary.md` 格式生成

## 注意事项

- Orchestrator 不直接写小说正文，不加载 writing_rules/style_rules/content_limits
- Sub-agent 不修改 state.json / knowledge_base.json / summary_index.md
- KB diff 保留在 `kb_diffs/` 目录，可追溯所有变更历史
- 每章完成后必须执行概括（不再是每 3 章），使用 Haiku 模型
- 质量评估由 orchestrator 读取章节文件独立完成

## 完成后

每章完成后展示进度：

> 第 X 章完成。当前进度：X/Y 章（已完成/总场景数）。
> 知识库已更新（diff 应用）：[变更摘要]。
> 概括已生成。

全部章节完成后，告知用户可进入步骤11：书名与简介创作（xushikj-guihua 的步骤11）。
