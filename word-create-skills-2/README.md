# 叙事空间创作系统

> 三位一体·叙事空间创作系统 v3.0 — 集"顶级商业网文策划师"、"资深代笔人"与"高智能交互式创作AI"于一身的 Claude Code 创作工具包。

## 系统组成

| Skill | 职责 | 触发词 |
|-------|------|--------|
| **xushikj-chuangzuo** | 主入口，智能路由与流程调度 | 叙事空间、网文创作、商业小说 |
| **xushikj-duibiao** | 步骤0：对标作品学习与风格解析 | 对标学习、风格分析 |
| **xushikj-guihua** | 步骤1-6+11：从一句话到四页大纲 + 书名简介 | 写大纲、人物设定、规划情节 |
| **xushikj-zhishiku** | 步骤7：动态实体知识库管理 | 初始化知识库、更新知识库 |
| **xushikj-changjing** | 步骤8-9：场景清单与场景规划 | 场景清单、规划场景 |
| **xushikj-xiezuo** | 步骤10（A）：流水线逐章创作 | 写第N章、开始写作 |
| **xushikj-hudong** | 步骤10（B）：互动式写作 | 互动写作、帮回 |
| **humanizer-xiaoshuo** | 后处理：去 AI 痕迹 | 小说去AI、去AI味 |

## 创作流程

```
对标(可选) → 一句话 → 一段式 → 人物 → 一页大纲 → 人物大纲 → 四页大纲
                                                                    ↓
                                                              知识库初始化
                                                                    ↓
                                                         场景清单 → 规划场景
                                                                    ↓
                                                    ┌─ 流水线模式(A) ──→ 书名简介
                                                    └─ 互动模式(B) ───→ 书名简介
                                                                    ↓
                                                          humanizer 去AI味
```

## 安装（重要：必须完整安装）

> **8 个 Skill 必须全部安装到同一个 `.claude/skills/` 目录下。** 不可以只装部分。
> Skill 之间存在大量跨引用（见下方「跨 Skill 引用地图」），缺失任何一个都会导致断链。

```bash
# 将全部 8 个目录复制到项目的 .claude/skills/ 下
cp -R xushikj-chuangzuo xushikj-guihua xushikj-duibiao \
      xushikj-changjing xushikj-xiezuo xushikj-hudong \
      xushikj-zhishiku humanizer-xiaoshuo \
      /your-project/.claude/skills/
```

安装后的目录结构应该是：

```
/your-project/.claude/skills/
├── xushikj-chuangzuo/    ← 主入口
├── xushikj-guihua/
├── xushikj-duibiao/
├── xushikj-changjing/
├── xushikj-xiezuo/
├── xushikj-hudong/
├── xushikj-zhishiku/
└── humanizer-xiaoshuo/
```

重启 Claude Code（或开新对话）即可使用。

## 快速开始

```
你：写一部网文
系统：自动路由到 xushikj-guihua，从步骤1开始引导

你：对标学习《斗破苍穹》
系统：路由到 xushikj-duibiao，分析对标作品

你：写第3章
系统：路由到 xushikj-xiezuo，执行逐章创作

你：互动写作
系统：路由到 xushikj-hudong，开启互动模式

你：去AI味
系统：路由到 humanizer-xiaoshuo，执行后处理
```

## 两种写作模式

- **流水线模式（A）**：系统按场景规划自动逐章创作，用户在章间审阅。适合有完整大纲、追求效率的场景。
- **互动模式（B）**：用户实时引导剧情方向，帮回系统为核心操控手段。适合探索性创作、想掌控每个转折点的场景。

## 核心特性

- **十二步工作流**：从对标分析到成稿的完整商业小说创作管线
- **动态知识库**：基于 Ex3 论文，持续追踪角色/物品/地点/势力状态变化
- **帮回辅助系统**：干预叙事走向的核心机制
- **双保险质量控制**：写作模块内置质量评估
- **Sub-agent 架构**：写作和去AI味均支持并行处理，长篇不爆上下文
- **可定制配置**：`xushikj-chuangzuo/config/` 下的 12 个 YAML 可调节写作风格、节奏、规则

## 配置文件

`xushikj-chuangzuo/config/` 目录包含 12 个 YAML 配置文件，控制创作的各个维度：

- `writing_rules.yaml` — 写作规则
- `style_rules.yaml` — 风格规则
- `genre_themes.yaml` — 类型与题材
- `rhythm_beats.yaml` — 节奏与节拍
- 等（详见目录内文件）

可根据个人风格偏好自定义修改。

## 文件产出结构

创作过程中系统会在工作目录生成：

```
.xushikj/
├── state.json           # 创作进度状态
├── benchmark/           # 对标分析报告
├── outline/             # 大纲系列文件
├── knowledge_base.json  # 动态知识库
├── scenes/              # 场景清单与规划
└── chapters/            # 章节正文
```

## 跨 Skill 引用地图

8 个 Skill 之间存在以下文件级跨引用。这些路径使用 `skill名/子路径` 格式，在 `.claude/skills/` 目录下作为平级目录时**天然可用，无需修改**。

### 引用关系总览

```
xushikj-hudong ──复用──→ xushikj-xiezuo/references/chapter-writer-sub-agent-prompt.md
                ──复用──→ xushikj-xiezuo/references/kb-diff-schema.md
                ──复用──→ xushikj-xiezuo/references/summary-sub-agent-prompt.md
                ──复用──→ xushikj-xiezuo/scripts/apply_kb_diff.py
                ──复用──→ xushikj-chuangzuo/config/*.yaml

xushikj-xiezuo ──引用──→ xushikj-chuangzuo/templates/summary_index_template.md

xushikj-zhishiku ─引用──→ xushikj-xiezuo/references/kb-diff-schema.md

humanizer-xiaoshuo ─协作──→ xushikj-chuangzuo/config/writing_rules.yaml（不冲突声明）
                   ─协作──→ xushikj-chuangzuo/config/style_rules.yaml（不冲突声明）
```

### 路由引用（YAML metadata + 文本提及）

所有 6 个子 Skill 的 SKILL.md 包含 `parent: xushikj-chuangzuo`。
各模块在步骤衔接时会提示用户进入下一模块（如 xushikj-guihua → xushikj-zhishiku → xushikj-changjing → xushikj-xiezuo）。

### 路径解析规则

当 AI 在 Skill 文件中读到 `xushikj-xiezuo/references/kb-diff-schema.md` 这样的路径时，应解析为：

```
.claude/skills/xushikj-xiezuo/references/kb-diff-schema.md
```

即：`.claude/skills/` + `路径原文`。所有跨引用路径都遵循此规则。

---

## 部署验证（供 AI 执行）

安装完成后，AI 可执行以下验证确认系统完整性：

```bash
# 1. 确认 8 个目录全部存在
for skill in xushikj-chuangzuo xushikj-guihua xushikj-duibiao \
  xushikj-changjing xushikj-xiezuo xushikj-hudong \
  xushikj-zhishiku humanizer-xiaoshuo; do
  [ -d ".claude/skills/$skill" ] && echo "✓ $skill" || echo "✗ MISSING: $skill"
done

# 2. 验证关键跨引用文件存在
for f in \
  xushikj-xiezuo/references/chapter-writer-sub-agent-prompt.md \
  xushikj-xiezuo/references/kb-diff-schema.md \
  xushikj-xiezuo/references/summary-sub-agent-prompt.md \
  xushikj-xiezuo/scripts/apply_kb_diff.py \
  xushikj-chuangzuo/config/writing_rules.yaml \
  xushikj-chuangzuo/config/style_rules.yaml \
  xushikj-chuangzuo/templates/summary_index_template.md; do
  [ -f ".claude/skills/$f" ] && echo "✓ $f" || echo "✗ MISSING: $f"
done

# 3. 验证 parent 引用完整
grep -rl "parent: xushikj-chuangzuo" .claude/skills/xushikj-*/SKILL.md | wc -l
# 期望结果：6（guihua, duibiao, changjing, xiezuo, hudong, zhishiku）
```

如果所有检查通过，系统即可正常工作。

## 许可

本工具包供个人创作使用。
