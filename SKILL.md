---
name: skill-localizer
display_name: "技能汉化官"
display_name_en: "Skill Localizer"
description: "把一个英文的 Agent Skill 完整汉化成中文：界面大标题、描述、自动触发条件全部变中文，且不破坏技能加载。自动触发条件：用户说汉化这个技能、把技能改成中文、这个技能是英文的看不懂、给技能起中文名、技能名字汉化、添加中文触发条件、本地化技能等需求时启用。专门改造技能本身的语言，与创造全新技能类技能区分——本技能是把已有英文技能翻译改造，不是从零新建。"
description_zh: "把英文技能汉化：中文大标题+中文描述+中文触发条件。触发：汉化这个技能、技能改成中文、技能名字汉化。"
description_en: "Localize an English Agent Skill into Chinese: Chinese display name, description, and trigger conditions."
version: "1.1.0"
---

# 技能汉化官

把一个英文 Skill 完整汉化成中文——让它在 WorkBuddy 技能卡片上显示中文大标题、中文描述和中文触发条件，同时**不破坏技能的加载和触发**。

> 本技能为**自包含单文件**版本：核心流程、机制原理、测试用例全部收录在本文档中，无需任何外部依赖文件。

## 核心原则：先给方案，再动手

这个技能采用**两段式工作流**：先诊断并展示汉化方案，等用户确认后再修改文件。原因是直接改技能文件有风险（可能改错字段、遗漏文件、破坏加载），用户看一眼方案只需几秒，却能避免返工。

**永远不要在用户确认前修改任何技能文件。** 唯一例外：用户明确说"直接改""不用看方案""你看着办"。

## 关键背景知识（必读，这是踩坑验证出来的）

WorkBuddy 的技能界面从**多个不同来源**读取信息，改错地方就会白忙活。完整原理见本文末尾「附录 A：汉化机制原理」。这里记住四个关键点：

| 界面元素 | 真正的数据源 | 改哪里 |
|---|---|---|
| 卡片大标题 | 缓存的 `name` 字段 | `_skillhub_meta.json` 的 `name` + 缓存 `name` |
| 卡片描述文字 | 缓存的 `description` | SKILL.md 的 `description` 字段本身 |
| hover 显示名 | `displayName` | SKILL.md 的 `display_name` |
| 技能能否加载 | `filePath` + `overrideKey` | 保持英文原名，不要动 |

**最大的坑**：只加 `description_zh` 字段是没用的——WorkBuddy 界面读的是 `description` 字段**本身**。必须把 `description` 直接写成中文。

**第二个坑**：卡片大标题不是读 `display_name`，而是读 `_skillhub_meta.json` 里的 `name`。手动安装的技能往往缺这个文件，必须补建。

## 工作流

### 第一步：定位与诊断

1. 确认要汉化的技能。用户可能给：技能名、技能文件夹路径、或截图里的英文标题。
2. 找到技能目录：在用户配置目录的 `skills/` 下查找。WorkBuddy 的用户配置目录通常是 `~/.workbuddy/`（Windows 为 `%USERPROFILE%\.workbuddy\`，macOS/Linux 为 `~/.workbuddy/`）。**不要假设具体的用户名路径**——用环境变量（如 `$HOME`、`%USERPROFILE%`）或 glob 搜索定位，兼容不同电脑和不同用户。
3. 读取 `<技能目录>/SKILL.md` 的 frontmatter（开头 `---` 之间的部分）。
4. 检查 `<技能目录>/_skillhub_meta.json` 是否存在。
5. 读取缓存 `<用户配置目录>/.skill-list-cache.json`，找到该技能对应条目。用同样的方式定位用户配置目录，不要写死任何用户名。

### 第二步：生成汉化方案（展示给用户，等待确认）

向用户展示一张清晰的方案表：

1. **中文技能名**：翻译原名，风格上建议用「功能」命名的中文名（如"技能汉化官""PDF 解读者"）。如果用户已有自己的编号习惯，可以沿用。给出 1-3 个候选让用户选。
2. **中文描述 + 自动触发条件**：把英文 description 翻译成中文，并按这个模板补写触发条件：
   > `[做什么]。自动触发条件：用户说[触发词1]、[触发词2]、[触发词3]等[场景]时启用。与[易混淆的其他技能]区分——[边界说明]。`

   触发词要写用户会说的**大白话**，不是术语。边界说明用来防止和相似技能打架。
3. **需要改动的文件清单**：列出将改哪些文件的哪些字段。

等用户确认（或用户说"直接改"）后再进入下一步。

### 第三步：执行汉化（确认后）

按顺序执行，每步都要验证：

1. **备份**：把 `SKILL.md` 复制一份到技能目录**之外**（如 `/tmp/`）。
   ⚠️ 备份绝对不能放在技能目录内——会被识别成重复技能。

2. **改 SKILL.md frontmatter**：
   - `description`: 换成第二步确认的中文描述（含触发条件）——这是最关键的一步
   - `display_name`: 中文技能名
   - `display_name_en`: 保留或补上英文原名
   - `description_zh`: 中文描述（可简短版）
   - `description_en`: 保留原英文描述
   - `name`: **保持英文原名不变**（Skill 标准要求，也用于系统定位）

3. **补 `_skillhub_meta.json`**（不存在则新建，存在则更新 `name` 字段）：
   ```json
   {
     "name": "中文技能名",
     "installedAt": <当前毫秒时间戳>,
     "source": "local",
     "version": "1.0.0",
     "description_zh": "中文描述",
     "iconSource": "letter:auto"
   }
   ```

4. **更新缓存** `.skill-list-cache.json`：
   - 找到该技能条目（用 `filePath` 或 `overrideKey` 定位，不要用 name，因为 name 可能被改过）
   - 把 `name` 改成中文
   - 把 `displayName` 改成中文
   - 把 `description` 改成中文
   - **保持 `filePath` 和 `overrideKey` 原样**（这是技能加载的锚点，改了会坏）
   - 写回 JSON 时使用 `ensure_ascii=False` 保留中文

### 第四步：三重验证（文件层面）

改完后必须验证，全部通过才算成功：

1. 读 `_skillhub_meta.json`，确认 `name` 是中文
2. 读 `.skill-list-cache.json`，确认该条目 `name`、`displayName`、`description` 都是中文
3. 确认 `<技能目录>/SKILL.md` 文件仍存在、frontmatter 格式完好（用 YAML 解析或读取开头确认）
4. 确认技能目录里没有遗留 `.bak` 等备份文件夹

### 第五步：触发验证（最关键，绝对不能省）

文件改对了不等于技能还能用。汉化改的 `description` 正是 AI 判断"何时启用这个技能"的依据——改不好就会出现"卡片显示中文、但喊它不动"或"不该触发时乱触发"。这一步专门验证触发是否依然正确。

**A. 静态检查（改完立刻做）**

1. 新的 description 里必须明确含"自动触发条件：用户说……时启用"，且触发词是用户真会说的大白话，不是术语堆砌。
2. 检查技能处于启用状态（缓存里 `disable` 为 false）。
3. **相似技能交叉检查**（最重要）：把新 description 和其他技能的 description 放一起看，有没有触发词重叠、会互相抢的场景。若发现重叠（例如两个技能都写"改技能"），必须在各自描述里补一句边界说明，讲清"这个活归我、那个活归它"。如果用户有自己的编号习惯，可沿用；边界写清"与某类技能的区别"即可。
4. **两处 description 必须一致**：SKILL.md 的 description 与缓存的 description 是两处独立存储，只改一处会导致界面显示和 AI 判断脱节。改完务必同步。

**B. 动态验证（真实召唤测试）**

光看文字不够，要实际喊几句话测：

1. 为这个技能写 2-3 句典型的用户大白话（从 description 的触发词里挑）。
2. 实际用这些话召唤技能，确认该技能被正确触发、并按预期工作。
3. 如果它和某个相似技能容易混，专门造一句"边界句"来测——比如测"改一下这个技能"应该找"创造技能"类技能，而不是本汉化技能。

**C. 向用户交付验证报告**

用一张清楚的表告诉用户：测了哪几句话、分别触发了哪个技能、对不对。让用户亲眼看到"说这句话→叫醒这个技能"。若有任何一句触发错误，回到第二步重写 description 的触发条件，再走一遍第五步。

只有第五步也全部通过，汉化才算真正完成。向用户报告时提醒：**新开一个会话**后界面卡片才会刷新显示中文（WorkBuddy 的显示缓存机制）。

## 常见问题

- **改完界面还是英文？** 新开会话刷新。若仍不行，检查 `_skillhub_meta.json` 的 `name` 和缓存的 `name` 是否都改了。
- **技能突然加载失败？** 检查是不是误改了 SKILL.md 的 `name` 字段或缓存的 `filePath`/`overrideKey`——这三个必须保持原样。从备份恢复后重来。
- **用户只想改显示名，不想改描述？** 尊重用户，但提醒：不改 `description` 的话触发条件还是英文，可能影响 AI 判断何时启用。
- **一次汉化多个技能？** 逐个走流程，每个都单独展示方案、单独确认，不要批量改动。

---

## 附录 A：汉化机制原理（WorkBuddy 技能）

这份附录记录了对 WorkBuddy 技能系统的实测发现，是本技能的底层依据。所有结论都经过实际验证。

### A1. 界面从哪里读数据

WorkBuddy 的技能界面不是一个单一数据源，而是从多处拼接：

```
技能卡片
├─ 大标题   ← .skill-list-cache.json 的 name 字段
├─ 描述文字 ← .skill-list-cache.json 的 description 字段
└─ 开关状态 ← 用户全局配置（多端同步，不改技能文件）
```

而 `.skill-list-cache.json` 里的值，又是从技能目录的两个文件生成的：

```
缓存 name        ← _skillhub_meta.json 的 name（若有）或 SKILL.md 的 display_name
缓存 description ← SKILL.md 的 description 字段本身
缓存 displayName ← SKILL.md 的 display_name
```

### A2. 字段对照表（实测结论）

对比了内置中文技能和手动安装的英文技能，发现：

| 技能类型 | SKILL.md name | _skillhub_meta.json name | 缓存 name | 卡片显示 |
|---|---|---|---|---|
| 内置中文技能 | 英文 `media-captions-frame` | 中文 `字幕画面调整助手` | 中文 | 中文 |
| 内置中文技能 | 英文 `funasr-asr` | 中文 `FunASR 本地语音识别` | 中文 | 中文 |
| 手动安装（汉化前） | 英文 `internal-comms` | 无此文件 | 英文 | 英文 |
| 手动安装（汉化后） | 英文 `internal-comms`（不变） | 中文（自定义） | 中文 | 中文 |

**关键结论**：
1. SKILL.md 的 `name` 始终是英文，不要改（Skill 标准 + 系统定位需要）。
2. 中文大标题来自 `_skillhub_meta.json` 的 `name`。
3. `display_name` 影响 hover 显示名，但不决定卡片大标题。
4. `description_zh` 字段**不被界面直接读取**——界面读 `description` 本身。

### A3. 踩过的三个坑

**坑 1：只加 `description_zh` 没用。** 最初给英文技能加了 `description_zh` 中文字段，界面纹丝不动。原因：WorkBuddy 缓存生成时读的是 `description` 字段的值。正确做法：直接把 `description` 字段的值改成中文。

**坑 2：改 `display_name` 后卡片大标题还是英文。** 改了 `display_name`，hover 显示名变了，但卡片上的大字标题没变。原因：大标题读的是缓存 `name`，而缓存 `name` 来自 `_skillhub_meta.json`。正确做法：补 `_skillhub_meta.json`，写中文 `name`，并同步改缓存。

**坑 3：备份文件夹被当成重复技能。** 把技能备份成 `xxx.bak` 放在技能目录内，结果技能列表里出现了两个同名技能。正确做法：备份一律放技能目录之外。

### A4. 安全边界（改了会坏的三样东西）

以下三处必须保持原样，否则技能加载失败：

1. **SKILL.md 的 `name` 字段** — 保持英文原名
2. **缓存条目的 `filePath`** — 指向实际 SKILL.md 的绝对路径
3. **缓存条目的 `overrideKey`** — 系统内部定位用的英文键

改 `name` 为中文是安全的，因为缓存里 `filePath` 和 `overrideKey` 独立于 `name`，系统靠后者找文件。

### A5. 缓存文件位置与写法

- 通用定位：`<用户配置目录>/.skill-list-cache.json`
- Windows: `%USERPROFILE%\.workbuddy\.skill-list-cache.json`
- macOS/Linux: `~/.workbuddy/.skill-list-cache.json`

定位时优先用环境变量（`%USERPROFILE%` / `$HOME`），不要写死具体用户名——技能包可能被装在任何人的电脑上。

结构是 `{"version": N, "results": [...]}`，每个技能一个对象。改完用 Python 写回时务必 `ensure_ascii=False`，否则中文会变成 `\uXXXX` 转义。

### A6. 生效时机

- 改 SKILL.md 的 `description` → 缓存重建后生效（通常较快）
- 改缓存 `name` → 界面刷新后生效，稳妥起见让用户**新开一个会话**
- 某个管理工具可能仍显示英文（它读另一套缓存），属正常现象，不影响实际界面和使用

### A7. 验证清单

**文件层面（三重验证）**
- [ ] `_skillhub_meta.json` 的 `name` 是中文
- [ ] 缓存里该技能的 `name` / `displayName` / `description` 都是中文
- [ ] 缓存的 `description` 含触发条件，且与 SKILL.md 完全一致
- [ ] SKILL.md 存在、frontmatter 完好、`name` 仍是英文
- [ ] 缓存的 `filePath` 和 `overrideKey` 未改
- [ ] 技能目录内没有 `.bak` 等备份残留

**触发层面（第五步，不能省）**
- [ ] 技能处于启用状态（缓存 `disable` 为 false）
- [ ] description 明确含"自动触发条件：用户说……时启用"
- [ ] 与其他技能无触发词冲突；有重叠则在各自描述里写明边界
- [ ] 用 2-3 句典型用户语句实际召唤，确认触发正确
- [ ] 若有相似技能，造"边界句"交叉测试

### A8. 踩坑补充：两处 description 必须同步

SKILL.md 的 `description` 和缓存的 `description` 是**两处独立存储**。汉化时若只改了其中一处，会出现界面显示的和 AI 用来判断触发的不一致。**每次改完 description，都要确认两处一致。**

注意：在 shell 里直接写含正则/引号的 Python 单行命令容易被转义搞坏，复杂脚本要写成 .py 文件再执行。

---

## 附录 B：测试用例（evals）

用于验证技能行为是否符合预期：

| # | 用户请求 | 预期行为 |
|---|---|---|
| 1 | "我刚从 GitHub 装了一个英文技能，卡片上全是英文看不懂。帮我汉化一下，先看方案别直接改。" | 先定位技能目录、读取 frontmatter 和元数据，展示中文名候选、中文描述含触发条件的方案表，等待用户确认；**不修改任何文件**。 |
| 2 | "把 xxx 技能汉化成中文，名字要带编号，直接改吧不用看方案了。" | 按例外情况直接执行：备份到目录外，改 SKILL.md 的 description/display_name 等字段，补 `_skillhub_meta.json`，更新缓存三处字段，保持 name/filePath/overrideKey 不变，最后三重验证并提醒新开会话。 |
| 3 | "我汉化完一个技能，但界面卡片大标题还是英文，怎么回事？帮我查查。" | 按故障排查路径：检查 `_skillhub_meta.json` 的 name、缓存的 name 和 displayName 是否都改成中文，指出只改 display_name 或 description_zh 不够，并提醒新开会话刷新。 |
