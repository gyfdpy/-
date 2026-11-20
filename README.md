# 🀄 技能汉化官（Skill Localizer）

> 把英文的 AI Agent Skill 一键汉化成中文——卡片大标题、描述文字、自动触发条件全部变中文，且不破坏技能加载。

专为 **WorkBuddy** 技能机制打造。汉化一个技能，从"看不懂的英文"到"喊得应的中文技能"，两步走完，安全可控。

## ✨ 它解决什么问题

装一个英文技能容易，问题是：

- 卡片大标题是英文，看不懂是哪个
- 描述一大片英文，不知道啥时候该用
- 想汉化，却不知道该改哪个文件——改错了技能直接失灵

「技能汉化官」把作者踩过的坑全部固化成流程，帮你安全汉化：

| 它自动做的事 | 说明 |
|---|---|
| 🏷️ 中文大标题 | 改对真正的数据源，卡片标题变中文 |
| 📝 中文描述 + 触发条件 | 用大白话写清"什么时候启用" |
| 🛡️ 两段式工作流 | 先出方案给你看，你点头才动手 |
| ✅ 触发验证 | 改完检查技能还能不能被正确喊醒 |

## 🚀 安装

1. 下载本仓库（点右上角绿色 `Code` → `Download ZIP`），解压。
2. 把解压出的文件夹（含 `SKILL.md`）放到 WorkBuddy 技能目录：
   - Windows: `%USERPROFILE%\.workbuddy\skills\skill-localizer\`
   - macOS / Linux: `~/.workbuddy/skills/skill-localizer/`
3. 新开一个对话，让 WorkBuddy 重新扫描技能。

> 也可以直接在 WorkBuddy 的「添加技能 → 上传技能」里导入仓库里的 `.skill` 文件（如果有的话）。

## 💬 怎么用

直接用大白话提需求，不用记命令：

```
帮我汉化这个英文技能，先看方案别直接改
```

```
把 xxx 技能改成中文，加上触发条件
```

技能会先展示汉化方案（中文名候选、中文描述、触发条件、要改的文件清单），**等你确认后才动手**。

## 🧠 三个已知的坑（本技能已规避）

1. 只加 `description_zh` 字段没用——WorkBuddy 界面读的是 `description` 字段本身。
2. 卡片大标题来自 `_skillhub_meta.json` 的 `name`，不是 `display_name`。
3. 改完必须验证触发——`description` 同时是对照触发的"招牌"，改不好技能就喊不应。

原理细节见 [`references/汉化机制原理.md`](references/汉化机制原理.md)。

## 📁 目录结构

```
skill-localizer/
├── SKILL.md                      # 核心流程（含两段式工作流 + 触发验证）
├── _skillhub_meta.json           # 中文显示名元数据
├── references/
│   └── 汉化机制原理.md            # 字段对照表、踩坑记录、验证清单
└── evals/
    └── evals.json                # 测试用例
```

## ⚠️ 适用环境

本技能针对 **WorkBuddy** 的技能机制编写。如果你用 Claude Code / Cursor 等其他 Agent，核心思路（改 `description` 写触发条件）通用，但涉及缓存和元数据的具体步骤需按对应平台调整。

## 🔒 安全说明

- 用户确认前不修改任何技能文件
- 备份放在技能目录之外，避免被误识别为重复技能
- 不改动 `name` / `filePath` / `overrideKey` 三个加载锚点

## 📄 协议

MIT — 可自由使用、修改、分享。

---

Made with 🤝 between a curious beginner and AI.
