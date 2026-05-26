# Skill: 卡牌美术生产工作流

## When to Use

当任务涉及以下任一场景时使用本 skill：

- 为卡牌游戏批量生产角色卡图、技能卡图或敌方动作图。
- 需要保持同一角色在多张卡图中的身份一致性。
- 用户提到六视图 anchor、图生图、before / after / iterate、v1/v2、审核定档。
- 需要把生图 agent 和审核 agent 分开，避免“自己生成自己放行”。
- 需要清理多次试验留下的旧脚本、旧 prompt、旧图片，重新确定当前正式产线。

## Prerequisites

- 项目中应有卡牌设计数据，例如 `data/card_designs.json`。
- 每个核心角色最好先有六视图 anchor：`anchor_000.png`、`anchor_060.png`、`anchor_120.png`、`anchor_180.png`、`anchor_240.png`、`anchor_300.png`。
- 需要一个可调用的图片生成或图片编辑接口。接口可以是 Gemini、OpenAI Images、ComfyUI、Replicate 或项目自带脚本。
- 如果使用 subagent 并行生产，先读 `rules/skills/workflow_parallel_subagents.md`。

## 核心原则

先生产角色 anchor，再用 anchor 做正式卡图。  
正式卡图必须保留 `prompt / output / critique` 三件套。  
生成和审核分开。生产 agent 只负责产出候选，reviewer 负责 PASS/FAIL。  
漂亮不等于通过。角色身份、动作语义、道具逻辑、系列一致性都是硬门槛。

## 推荐目录

```text
asset_generation/
  <character_key>/
    character_description.md
    anchor_000.png
    anchor_060.png
    anchor_120.png
    anchor_180.png
    anchor_240.png
    anchor_300.png
    stage2_audit.md
  card_art/
    common_player.md
    common_enemy.md
    rubric.md
assets/
  cards/
  iterations/
    <card_id>/
      v1_prompt.txt
      v1_output.png
      v1_critique.md
data/
  card_designs.json
```

## 步骤

### 1. 扫描当前产线

先区分“正式产线”和“历史试验”：

- 找当前被游戏或数据引用的 `assets/cards/`。
- 找当前生图脚本，优先确认是否支持 before / after / iterate。
- 找当前 rubric，确认审核规则来源。
- 找 archive、tmp、old、backup、test 等目录，把它们标记为历史试验，不要让 subagent 自己猜。

输出一个短结论：当前正式脚本是什么、正式输入是什么、正式输出目录是什么、哪些文件已废弃。

### 2. 固定公共块

公共块只写跨卡稳定信息：

- 角色身份：脸型、发型、服装、体型、标志物。
- 阵营视觉语言：材质、颜色、光效、背景复杂度。
- 卡图构图：竖图、主体安全区、不要 UI 和文字。
- 负面约束：不要 logo、水印、卡框、额外角色、错阵营道具。

公共块一旦进入批量生产，不要让多个 subagent 各自修改。需要改公共块时，停止生产，统一改完再继续。

### 3. 写单卡块

单卡块只描述这一张卡：

- 这张卡的功能是什么。
- 角色做了什么可见动作。
- 表情为什么匹配这个效果。
- 关键道具是什么，如何参与动作。
- 背景和特效如何支持语义，而不是抢主体。

不要把卡牌数值、UI、卡名文字塞进图片。卡图源文件只画插画。

### 4. 跑 before / after / iterate

- `before`：无 anchor 文生图，只用作跑偏对照，不可定档。
- `after`：正式图生图，使用角色 anchor，生成 v1。
- `iterate`：基于上一轮 critique 修 v2/v3。

推荐输出：

```text
assets/iterations/<card_id>/v1_prompt.txt
assets/iterations/<card_id>/v1_output.png
assets/iterations/<card_id>/v1_critique.md
```

### 5. 并行生产

适合并行时，按角色、阵营或卡牌批次拆给多个生产 subagent。每个 subagent 的任务必须包含：

- 可使用的公共块路径。
- 负责的卡牌 ID 列表。
- 可用 anchor 路径。
- 输出目录。
- 禁止定档，只能生成候选。

主 agent 不要干等，可以继续做目录审核、prompt 审核、已有图片审图、游戏接入检查。

### 6. 单卡审核

每张图都用 PASS/FAIL：

- Official flow：是否来自正式 after/iterate，是否使用 approved anchors。
- Format gate：是否为竖向卡图，主体是否在安全区域。
- Character identity：脸、发型、体型、服装语言是否匹配 anchor。
- Expression fit：表情是否符合卡牌功能。
- Action match：画面是否执行这张卡的具体效果。
- Prop logic：关键道具是否服务效果，物理关系是否说得通。
- Composition：脸、手、动作方向和核心道具是否清楚。
- Leakage：是否有文字、数字、logo、水印、UI、卡框。
- Series fit：亮度、饱和度、背景类型、渲染风格是否一致。
- Semantic specificity：为什么这是这张卡，而不是同角色普通插画？

FAIL 必须写具体原因和可见位置。

### 7. 系列审核与定档

同一角色或阵营至少三张单卡 PASS 后，再做系列 cross-check。  
系列 PASS 后才能复制到 `assets/cards/<card_id>.png`，并更新游戏数据中的 `image_file` 字段。  
不要覆盖已有定档图，除非明确记录替换原因。

## 审核模板

```markdown
# <card_id> <version> Critique

Status: PASS / FAIL / CONTENT PASS FORMAT HOLD

## Summary
一句话说明是否可进入下一步。

## Rubric
- Official flow: PASS/FAIL. ...
- Format gate: PASS/FAIL. ...
- Character identity: PASS/FAIL. ...
- Expression fit: PASS/FAIL. ...
- Action match: PASS/FAIL. ...
- Prop logic: PASS/FAIL. ...
- Composition: PASS/FAIL. ...
- Leakage: PASS/FAIL. ...
- Series fit: PASS/FAIL/HOLD. ...
- Semantic specificity: PASS/FAIL. ...

## Decision
定档、继续 v2、重开公共块、或等待系列 cross-check。
```

## OpenClaw 使用方式

把本文件复制到目标 workspace 的 `rules/skills/workflow_card_art_production.md`，再在 `rules/skills/INDEX.md` 的 Workflow 分类添加一行：

```markdown
- [卡牌美术生产工作流](./workflow_card_art_production.md) - 六视图 anchor、单卡迭代、并行生产、审核定档
```

之后当用户提到“卡牌美术生产、卡图、anchor、before/after/iterate、subagent 生图、审核定档”时，agent 应先读取本 skill。
