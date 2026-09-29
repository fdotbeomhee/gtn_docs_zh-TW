# FloatInput {: #floatinput }

## 功能概述 {: #what-is-it }

FloatInput 節點提供了一種簡潔直接的方式，用於在工作流程中輸入浮點數 (Floating-point Number) 數值。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 定義具備小數精度的數值
- 為工作流程建立可靈活配置的數值參數
- 為下游數學運算節點提供前置數值
- 設定系統門檻值 (Thresholds)、比率 (Rates) 或其他小數數值

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 FloatInput 節點新增至工作流程中
1. 設定您所需的浮點數數值

### 參數說明 {: #parameters }

- **float**：浮點數數值（預設值為 `0.0`）

### 輸出埠說明 {: #outputs }

- **float**：輸出的浮點數數值，可直接供其他下游節點連線使用

## 範例展示 {: #example }

為 AI 模型設定採樣溫度 (Temperature) 參數：

1. 將 FloatInput 節點新增至工作流程中
1. 將 "float" 數值設定為 `0.7`
1. 將輸出引腳連線至 AI 驅動節點的 `temperature` 參數埠
1. 該 AI 模型後續將以 `0.7` 作為其生成時的溫度配置

## 重要注意事項 {: #important-notes }

- 該節點專門用於建立單一浮點數值。
- 若未特別設定，預設初始值為 `0.0`。
- 您可隨時在節點的屬性面板中手動調整修改此數值。
- 浮點數支援表示各種帶有小數的數值，例如 `0.5`、`3.14159` 或 `-2.75`。

## 常見問題排解 {: #common-issues }

- 無特殊已知問題。
