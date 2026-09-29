# IntegerInput {: #integerinput }

## 功能概述 {: #what-is-it }

IntegerInput 節點提供了一種簡潔直接的方式，用於在工作流程中輸入整數 (Integer, 全整數值)。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 定義不含小數點的純整數數值
- 為工作流程建立可靈活配置的整數參數
- 設定計數器 (Counters)、陣列索引 (Indices) 或數量數值
- 設定系統限制門檻 (Limits)、邊界或整數配額

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 IntegerInput 節點新增至工作流程中
1. 設定您所需的整數數值

### 參數說明 {: #parameters }

- **integer**：整數數值（預設值為 `0`）

### 輸出埠說明 {: #outputs }

- **integer**：輸出的整數數值，可直接供其他下游節點連線使用

## 範例展示 {: #example }

為 AI 生成回應設定最大 Token 限制 (Max Tokens)：

1. 將 IntegerInput 節點新增至工作流程中
1. 將 "integer" 數值設定為 `500`
1. 將輸出引腳連線至 AI 驅動節點的 `max_tokens` 參數埠
1. 該 AI 模型將把回應生成上限嚴格限制在 500 個 Token 內

## 重要注意事項 {: #important-notes }

- 該節點專門用於建立單一整數數值。
- 若未特別設定，預設初始值為 `0`。
- 您可隨時在節點的屬性面板中手動調整修改此數值。
- 整數用於表示不帶小數的正負整數與零，例如 `1`、`42`、`-7` 或 `0`。
