# 4399游戏盒IP卡牌美术生产 Skill

这是一个在探索制作4399游戏盒IP角色43君99娘为主角的卡牌游戏沉淀下来的卡牌美术生产工作流 skill。

它沉淀的是一套适合 AI agent 执行的卡牌游戏美术流水线：先做角色六视图 anchor，再按卡牌语义生成候选图，保留 `prompt / output / critique`，最后用独立 reviewer 做 PASS/FAIL 审核和系列定档。

## 适用场景

- 批量生产卡牌游戏的角色卡图、技能卡图、敌方动作图。
- 需要同一角色在多张卡图中保持身份一致。
- 需要用 before / after / iterate 管理生图试验与正式产线。
- 需要把生图 subagent 和审核 reviewer 分开，避免生产者自己放行。
- 需要清理多次探索后留下的旧脚本、旧 prompt、旧图片，重新明确当前正式产线。

## 安装方式

把 skill 文件复制到你的 OpenClaw workspace：

```bash
cp rules/skills/workflow_card_art_production.md /path/to/your/workspace/rules/skills/
```

然后在目标 workspace 的 `rules/skills/INDEX.md` 的 Workflow 分类添加：

```markdown
- [卡牌美术生产工作流](./workflow_card_art_production.md) - 六视图 anchor、单卡迭代、并行生产、审核定档
```

之后当用户提到“卡牌美术生产、卡图、anchor、before/after/iterate、subagent 生图、审核定档”时，agent 应先读取这个 skill。

## 仓库结构

```text
rules/
  skills/
    INDEX.md
    workflow_card_art_production.md
```

## 核心流程

1. 扫描当前产线，区分正式脚本和历史试验。
2. 固定角色公共块和阵营视觉规则。
3. 为每张卡写单卡语义块。
4. 使用 before / after / iterate 生成候选。
5. 用多个 subagent 并行生产候选图。
6. 主 agent 或独立 reviewer 按 PASS/FAIL 审核。
7. 系列一致性通过后才定档到 `assets/cards/`。

## 关键原则

- 正式卡图必须使用角色 anchor 作为图生图参考。
- 候选图、prompt、critique 必须同目录保存。
- 表情、动作、道具逻辑是硬门槛。
- 卡图源文件不要包含游戏 UI、文字、logo、水印或卡框。
- 单卡 PASS 不等于定档，仍需系列 cross-check。
