# RulesetList {: #rulesetlist }

## 功能概述 {: #what-is-it }

RulesetList 節點將多個規則集 (Rulesets) 合併為單一清單。藉由將多個獨立的規則集歸組聚合，能為 AI Agent 代理定義更豐富且精準的複合行為準則。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用 RulesetList 節點：

- 欲合併多個規則集以構建更複雜細緻的 Agent 行為
- 欲將不同面向的行為規範整合至單一集合中
- 構建需要同時遵循多套規範與邊界約束的 AI Agent
- 欲模組化混搭不同的業務邏輯規則體系

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 RulesetList 節點新增至工作流程中
1. 將最多四個獨立的 Ruleset 節點連線至輸入參數（ruleset_1、ruleset_2、ruleset_3、ruleset_4）
1. 將輸出埠 (rulesets) 連線至任何接收規則集清單的下游節點（如 Agent 節點）

### 參數說明 {: #parameters }

- **ruleset_1**：加入合併清單的第一個規則集
- **ruleset_2**：加入合併清單的第二個規則集
- **ruleset_3**：加入合併清單的第三個規則集
- **ruleset_4**：加入合併清單的第四個規則集

### 輸出埠說明 {: #outputs }

- **rulesets**：包含所有非空 (Non-null) 輸入規則集的合併清單物件

## 範例展示 {: #example }

為 Agent 聚合對話規範與業務處理兩套專屬規則集：

1. 新增第一個 Ruleset 節點至工作流程，命名為「對話規範 (Conversation Rules)」
1. 新增第二個 Ruleset 節點至工作流程，命名為「任務處理規範 (Task Rules)」
1. 新增 RulesetList 節點至工作流程中
1. 將「對話規範」節點連線至 RulesetList 的 ruleset_1 輸入埠
1. 將「任務處理規範」節點連線至 RulesetList 的 ruleset_2 輸入埠
1. 將 RulesetList 的輸出埠 (rulesets) 連線至 Agent 節點的 ruleset 輸入引腳
