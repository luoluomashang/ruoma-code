---
name: xushikj-chuangzuo
description: |
  叙事空间创作系统主入口。当用户需要写网文、商业小说、叙事空间创作时使用。
  基于三位一体·叙事空间创作系统 v3.0，十二步工作流 + 动态知识库 + 帮回辅助系统。
  自动路由到对标/规划/知识库/场景/写作/互动六个子Skill。
metadata:
  version: 1.0.0
  triggers:
    - 叙事空间
    - 网文创作
    - 商业小说
    - 爆款小说
    - 叙事空间创作
---

# 叙事空间创作系统

集"顶级商业网文策划师"、"资深代笔人"与"高智能交互式创作AI"于一身的"三位一体"创作系统。主入口负责智能路由和流程调度。

## 创作流程

```
对标(可选) → 一句话 → 一段式 → 人物 → 一页大纲 → 人物大纲 ─┬─→ 四页大纲
                                                              │         ↓
                                                              │   知识库初始化
                                                              │         ↓
                                                              │  场景清单 → 规划场景
                                                              │         ↓
                                                              │  逐章创作(×N) → 书名简介
                                                              │  [流水线模式 A]
                                                              │
                                                              └─→ 知识库(精简)
                                                                        ↓
                                                                  互动式写作(×N) → 书名简介
                                                                  [互动模式 B]
```

## 步骤映射表（十二步 → 六子Skill）

| 步骤 | 名称 | 子Skill | 产出 |
|------|------|---------|------|
| 0 | 对标作品学习（可选） | xushikj-duibiao | benchmark/style_report.md |
| 1 | 一句话概括 | xushikj-guihua | outline/one_sentence.md |
| 2 | 一段式概括 | xushikj-guihua | outline/one_paragraph.md |
| 2.5 | 世界观与力量体系设定 | xushikj-guihua | outline/worldview_and_system.md |
| 3 | 一页人物介绍 | xushikj-guihua | outline/characters.md |
| 4 | 一页大纲 | xushikj-guihua | outline/volume_{V}_one_page.md |
| 5 | 人物大纲 | xushikj-guihua | outline/character_arcs.md |
| 6 | 四页大纲 | xushikj-guihua | outline/volume_{V}_four_pages |
| 7 | 动态实体知识库 | xushikj-zhishiku | knowledge_base.json |
| 8 | 场景清单 | xushikj-changjing | scenes/scene_list.md |
| 9 | 规划场景 | xushikj-changjing | scenes/scene_plans/*.md |
| 10 | 逐章创作（流水线） | xushikj-xiezuo | chapters/chapter_*.md |
| 10-互动 | 互动式写作 | xushikj-hudong | chapters/chapter_*.md |
| 11 | 书名与简介 | xushikj-guihua | outline/title_and_synopsis.md |

## 路由逻辑

### 自动识别阶段

根据用户意图和 `.xushikj/state.json` 状态自动路由：

| 用户意图 | 路由目标 |
|----------|----------|
| "对标学习"、"风格分析"、"学习作品风格" | → xushikj-duibiao |
| "写网文"、"开始创作"、无 state.json | → xushikj-guihua（从步骤1开始） |
| "写大纲"、"人物设定"、"规划情节" | → xushikj-guihua |
| "初始化知识库"、"建立知识库" | → xushikj-zhishiku |
| "场景清单"、"规划场景" | → xushikj-changjing |
| "写第N章"、"开始写作"、"继续写" | → xushikj-xiezuo（流水线）或 xushikj-hudong（互动） |
| "互动写作"、"开始互动"、"帮回" | → xushikj-hudong |
| "书名"、"简介" | → xushikj-guihua（步骤11） |

### 项目状态检查

执行前检查 `.xushikj/` 目录的文件状态：

```
检查 .xushikj/state.json         → 是否存在项目
检查 .xushikj/outline/           → 规划阶段完成度
检查 .xushikj/knowledge_base.json → 知识库是否初始化
检查 .xushikj/scenes/            → 场景规划完成度
检查 .xushikj/chapters/          → 已完成章节数
```

## 执行流程

### Step 1: 确认项目目录

询问用户项目位置，或使用当前目录：
- 默认：当前工作目录
- 可指定：`小说写作台/{项目名}/`
- 确保目录存在

### Step 2: 检查项目状态

读取 `.xushikj/state.json`，判断当前进度：

**情况A：无 state.json（新项目）**
→ 执行初始设置流程：
  1. 询问启动模式（原创 / 风格学习）
  2. 收集核心设定（类型标签、主角困境、金手指等）
  3. 确认默认配置（回复长度、互动选项等）
  4. 创建 `.xushikj/` 目录和 `state.json`
  5. 路由到对应子Skill

**情况B：state.json 存在（断点续做）**
→ 读取状态，校验文件完整性：
  1. 读取 state.json，获取 current_step 和 completed_steps
  2. 读取 memory.md（如存在），获取任务进度和用户叮嘱
  3. 校验已完成步骤的产出文件是否完整
  4. 若文件缺失则回退到最后完整步骤
  5. 报告恢复状态，询问是否继续

### Step 3: 智能路由

根据 `current_step` 和用户意图，路由到对应子Skill：

| current_step | 子Skill | 传递上下文 |
|-------------|---------|-----------|
| 0 | xushikj-duibiao | 对标作品名 |
| 1-6, 11 | xushikj-guihua | 前序产出文件 |
| 7 | xushikj-zhishiku | outline/* 全部文件 |
| 8-9 | xushikj-changjing | volume_{V}_four_pages + knowledge_base.json |
| 10 | xushikj-xiezuo | 场景规划 + 知识库 + 前章 |
| 10-互动 | xushikj-hudong | 知识库 + 前章 + 用户指令 |

### Step 4: 执行子Skill

调用对应子Skill，传递必要上下文。
子Skill 完成后更新 `state.json`。

### Step 5: 阶段完成后

询问用户是否继续下一阶段，循环 Step 3-4。

## 快速命令

| 用户说 | 触发行为 |
|--------|----------|
| "写网文" / "叙事空间" | 进入主流程 |
| "继续写" | 自动判断下一步 |
| "对标学习 {作品名}" | 进入对标分析 |
| "写第N章" | 直接进入逐章创作 |
| "帮回{指令}" | 触发帮回系统 |
| "项目状态" | 显示当前进度 |

## 项目运行时目录结构

```
.xushikj/
├── state.json                  # 状态机
├── memory.md                   # 项目记忆（进度 + 叮嘱 + 反思）
├── knowledge_base.json         # 动态实体知识库
├── pending.md                  # 伏笔追踪
├── config/
│   └── user_settings.yaml      # 用户配置
├── outline/
│   ├── one_sentence.md         # 步骤1
│   ├── one_paragraph.md        # 步骤2
│   ├── characters.md           # 步骤3
│   ├── volume_{V}_one_page.md  # 步骤4
│   ├── character_arcs.md       # 步骤5
│   ├── volume_{V}_four_pages  # 步骤6
│   └── title_and_synopsis.md   # 步骤11
├── scenes/
│   ├── scene_list.md           # 步骤8
│   └── scene_plans/            # 步骤9
├── chapters/                   # 步骤10
├── summaries/                  # 章节概括（每3章自动生成）
│   ├── summary_index.md        # 全书概要索引
│   ├── group_XX_chNN-MM.md     # 分组概括（500字/组）
│   └── _progress.json          # 概括进度追踪
├── quality_reports/            # 质量评估报告
├── quarantine/                 # 双保险隔离区
└── benchmark/                  # 步骤0
```

## 配置文件索引

| 配置文件 | 用途 | 加载时机 |
|---------|------|---------|
| config/meta_rules.yaml | 元指令（最高优先级） | 全步骤 |
| config/genre_tags.yaml | 小说类型参考列表 | 初始化 + 步骤1 |
| config/methodology.yaml | 六大创作法则 | 全步骤 |
| config/writing_rules.yaml | 描写规范核心要求 | 步骤10 + 帮回 |
| config/bangui_modes.yaml | 帮回系统配置 | 帮回触发时 |
| config/quality_dimensions.yaml | 质量评估维度 | 步骤10完成后 |
| config/workflow.yaml | 工作流定义 | 路由时 |
| config/style_rules.yaml | 语言风格自然化 | 步骤10 |
| config/content_limits.yaml | 通用内容限制 | 步骤10 |
| config/golden_opening.yaml | 黄金开篇规则 | 前三章 |
| config/safety_guard.yaml | 双保险配置 | 步骤8-10 |
| config/declarations.yaml | 三级声明模板 | 按敏感度注入 |
| templates/memory_template.md | 项目记忆初始模板 | 新项目初始化 |
| templates/summary_index_template.md | 章节概括索引模板 | 首次概括触发时 |

## Notes

- 子Skill 可独立调用，会自动检查前置依赖
- 每步必须得到用户"满意"确认后方可继续
- 配置文件按需加载，避免上下文膨胀
- 原提示词写作规则逐字保留在配置文件中，不做任何修改
- humanizer 为可选后置模块，不内嵌到创作流程中
