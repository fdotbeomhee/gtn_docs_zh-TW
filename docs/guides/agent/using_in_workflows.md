# 在工作流程中使用智慧體 (Using in Workflows) {: #using-in-workflows }

**Agent 節點**是將 AI 智慧推理能力引入自動化工作流程的核心手段。與側邊欄的 Chat 聊天視窗（主打雙向互動與日常對話）不同，Agent 節點作為自動化管線的一環參與執行，負責接收來自上游其他節點的資料輸入，並將運算產出順暢傳遞給下游節點。

## 核心關鍵參數 {: #key-parameters }

- **provider** — 所採用的 AI 服務供應商（與您在[服務供應商](./providers/index.md)中配置的項目一致）。
- **prompt model** — 該供應商提供的特定模型名稱。
- **rulesets** — 連接 Ruleset 節點或 TextInput 以規範引導智慧體的行為準則；注意：工作流程節點中目前不支援 Skills 技能機制。
- **tools** — 賦予智慧體的各項擴充工具能力。
- **output_schema** — 選填項目，強制將智慧體的輸出約束為特定的 JSON 結構。

## 完整參考指南 {: #full-reference }

完整的參數對照表、JSON 輸出結構定義範例以及常見問題排解，請參閱 [Agent 節點參考手冊](../../nodes/agents/create_agent.md)。
