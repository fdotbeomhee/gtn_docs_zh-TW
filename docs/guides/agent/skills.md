# 技能 (Skills) {: #skills }

技能 (Skills) 是能夠為智慧體提供額外特定指令或領域專業知識的 Markdown 檔案 — 它們會在啟動時自動載入，且在每次執行時動態重新整理，因此修改內容無需重啟引擎即可立即生效。

Griptape Nodes 原廠隨附了一套用於構建與執行工作流程的內建模組技能。您可以在其基礎上自由擴充自己的自訂技能。

## 技能檔案存放位置 {: #where-to-put-skills }

在您的工作區目錄內建立一個 `.agents/skills/` 資料夾，並為每個技能建立專屬資料夾，內部放入一個 `SKILL.md` 檔案：

```
<workspace_directory>/
└── .agents/
    └── skills/
        ├── my-skill/
        │   └── SKILL.md
        └── another-skill/
            └── SKILL.md
```

預設工作區目錄為您啟動引擎路徑下的 `GriptapeNodes/`。您可以在 **Settings → File System → Workspace Directory** 中確認精確的絕對路徑。

## 技能檔案格式 {: #skill-file-format }

每個 `SKILL.md` 都是一個頂部帶有 YAML Frontmatter 區塊的標準 Markdown 檔案：

```markdown
---
name: my-skill-name
description: What this skill does and when the agent should use it.
---

# Skill title

Instructions, reference material, or domain knowledge the agent should apply
when this skill is relevant. Write in plain English — the agent reads this as
part of its context.
```

其中 `name` 必須與該技能的資料夾名稱完全一致，而 `description` 則負責向智慧體傳達在何種情境下應該主動啟用這項技能。正文內容可依需求自由撰寫長短 — 包含程式碼範例、逐步操作指引、速查對照表等等。

## 範例：團隊文案風格規範 {: #example-a-house-style-guide }

```markdown
---
name: writing-style
description: Apply our house style when drafting or editing text.
---

# Writing Style Guide

- Use sentence case for headings, not title case.
- Prefer active voice.
- Avoid jargon; explain technical terms on first use.
- Maximum sentence length: 25 words.
```

## 熱重載 (Hot reload) {: #hot-reload }

!!! tip "提示"

    技能具備自動感應機制 — 完全無需重啟引擎。只要在 `.agents/skills/` 下添加新的技能資料夾，您發送的下一條訊息就會立即套用該項新技能。

## 相關參考資源 {: #related }

- [Agent Skills 官方說明文件](https://agentskills.io/home) — 深入瞭解技能規範格式與進階擴充能力的完整參考
