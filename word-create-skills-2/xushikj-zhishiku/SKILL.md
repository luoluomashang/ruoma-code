---
name: xushikj-zhishiku
description: |
  叙事空间创作系统·知识库模块。执行步骤7：动态实体知识库初始化与管理。
  基于Ex3论文，建立角色/物品/地点/势力的动态追踪系统。
metadata:
  version: 1.0.0
  parent: xushikj-chuangzuo
  step: 7
  triggers:
    - 初始化知识库
    - 建立知识库
    - 更新知识库
---

# 叙事空间创作系统 - 知识库模块

## 概述

执行宏观十二工作流的第七步：动态实体知识库初始化与管理。源自《Ex3》论文的设计理念，替代传统静态「人物宝典」。建立一个在创作过程中持续更新的动态实体知识库，追踪角色、物品、地点、势力的全部状态变化，确保长篇创作的逻辑一致性。

## 前置依赖

| 依赖 | 必要性 | 说明 |
|------|--------|------|
| 步骤1-6（规划阶段） | 必须 | 需要完整的大纲和人物设定作为初始化数据源 |

必须确认以下文件存在：
- `.xushikj/outline/characters.md` - 人物介绍
- `.xushikj/outline/character_arcs.md` - 人物大纲
- `.xushikj/outline/volume_{V}_four_pages` - 四页大纲

## 产出文件

| 文件路径 | 说明 |
|----------|------|
| `.xushikj/knowledge_base.json` | 动态实体知识库主文件 |
| `.xushikj/state.json` | 更新当前步骤为 8（完成后） |

## 模板引用

知识库的数据结构遵循以下模板：

```json
{
  "meta": {
    "version": "1.0.0",
    "created_at": "YYYY-MM-DD",
    "last_updated": "YYYY-MM-DD",
    "last_updated_chapter": 0
  },
  "characters": {
    "char_001": {
      "id": "char_001",
      "name": "角色名",
      "aliases": ["外号", "绰号"],
      "role": "主角/女主/反派/配角",
      "age": 25,
      "status": "alive/dead/missing/unknown",
      "location": "当前所在地",
      "personality": ["性格特征1", "性格特征2"],
      "core_logic": "行为底层逻辑/不妥协的执念",
      "abilities": ["能力1", "能力2"],
      "relationships": {
        "char_002": {"type": "朋友", "note": "补充说明"}
      },
      "items": ["item_001"],
      "arc_stage": "当前弧光阶段",
      "history": [
        {"chapter": 0, "event": "初始状态", "timestamp": "YYYY-MM-DD"}
      ]
    }
  },
  "items": {
    "item_001": {
      "id": "item_001",
      "name": "物品名",
      "type": "武器/道具/金手指/信物",
      "owner": "char_001",
      "location": "随身携带/某地",
      "status": "active/destroyed/lost/sealed",
      "description": "物品描述",
      "history": []
    }
  },
  "locations": {
    "loc_001": {
      "id": "loc_001",
      "name": "地点名",
      "type": "城市/门派/秘境/战场",
      "status": "normal/destroyed/sealed/occupied",
      "characters_present": ["char_001"],
      "description": "地点描述",
      "history": []
    }
  },
  "factions": {
    "fac_001": {
      "id": "fac_001",
      "name": "势力名",
      "type": "门派/国家/组织/家族",
      "status": "active/destroyed/merged",
      "leader": "char_003",
      "members": ["char_001", "char_002"],
      "allies": ["fac_002"],
      "enemies": ["fac_003"],
      "description": "势力描述",
      "history": []
    }
  }
}
```

## 执行流程

### Step 1: 初始化

读取规划阶段的全部产出，提取所有实体信息：

1. 从 `characters.md` 和 `character_arcs.md` 提取角色信息
2. 从 `volume_{V}_four_pages` 提取物品、地点、势力信息
3. 为每个实体生成唯一 ID（`char_xxx`、`item_xxx`、`loc_xxx`、`fac_xxx`）
4. 填充初始状态

### Step 2: 录入与补全

向用户展示提取结果，逐类确认：

> 以下是从大纲中提取的角色信息，请确认是否完整：
> [角色列表]
> 是否需要补充或修改？

依次确认：角色 → 物品 → 地点 → 势力

### Step 3: 校验

对初始化后的知识库进行一致性校验：

- **关系对称性**：如果 A 是 B 的朋友，B 也应记录 A 为朋友
- **位置一致性**：角色所在地点的 `characters_present` 是否包含该角色
- **物品归属**：物品 owner 与角色 items 是否对应
- **势力成员**：势力 members 与角色信息是否一致

校验不通过时，报告冲突并要求用户确认修正。

### Step 4: 保存

将完整的知识库保存为 `.xushikj/knowledge_base.json`。

## 更新机制

知识库在后续创作过程中持续更新：

### 自动更新（步骤10逐章创作时触发）

每章创作完成后，写作模块（xushikj-xiezuo）会：

1. 从章节内容中提取实体状态变化
2. 生成变更记录（JSON diff）
3. 应用变更到知识库
4. 记录 history 条目（章节号 + 事件描述）
5. 更新 `meta.last_updated` 和 `meta.last_updated_chapter`

### 手动更新

用户可随时通过以下方式触发更新：

- 「更新知识库」：手动录入新的实体或状态变化
- 「查询知识库」：查看某个实体的当前状态和历史
- 「知识库一致性检查」：重新运行校验流程

### 冲突检测

在生成新内容前，必须查询知识库进行一致性校验：

| 冲突类型 | 检测方式 | 处理 |
|----------|----------|------|
| 已死亡角色出场 | 检查 `status == "dead"` | 阻断并警告 |
| 已销毁物品使用 | 检查 `status == "destroyed"` | 阻断并警告 |
| 角色位置矛盾 | 对比 location 与场景地点 | 警告并提供修正建议 |
| 关系状态矛盾 | 对比 relationships 记录 | 警告并提供修正建议 |
| 能力使用矛盾 | 对比 abilities 列表 | 警告并提供修正建议 |

## 注意事项

- 知识库是整个创作系统的「记忆核心」，所有后续模块都依赖它
- 每 10 轮对话后的内部剧情总结必须与知识库交叉验证
- 知识库更新应保留完整历史，不删除旧记录
- 如果检测到逻辑冲突，必须在继续创作前解决

## 完成后

告知用户知识库已初始化完毕，展示实体统计（角色数/物品数/地点数/势力数），询问是否进入下一步：场景模块（xushikj-changjing）。
