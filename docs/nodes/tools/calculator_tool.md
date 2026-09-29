# Calculator {: #calculator }

## 功能概述 {: #what-is-it }

Calculator 工具節點是一個數學計算工具，可直接掛載指派給 AI Agent 代理，協助模型精確執行各類數值與代數運算，避免 LLM 的數值幻覺。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 賦予 AI Agent 執行高精度數學計算的能力
- 在工作流程中解答各類複雜數理問題
- 無需手動撰寫計算程式碼即可精準處理各類數值數據

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 Calculator 節點新增至工作流程中
1. 將其輸出引腳連線至需要計算能力的節點（例如 Agent 節點的 tools 輸入埠）

### 輸出埠說明 {: #outputs }

- **tool**：配置完成的計算機工具物件 (Tool Object)，可直接供其他節點引用

## 範例展示 {: #example }

建立一個具備數學運算能力的問題解答 Agent：

1. 將 Calculator 節點新增至工作流程中
1. 將 "tool" 輸出埠連線至 Agent 節點的 "tools" 輸入埠
1. 現在該 Agent 在面對使用者對話中的數值計算題目時，便能自主呼叫計算機工具獲得精確答案

## 實作細節 {: #implementation-details }

Calculator 工具底層基於 Griptape 的 `CalculatorTool` 類別實作，提供標準化的數學表達式求值介面。此工具專門設計供 Agent 自主呼叫，以高效率、零誤差地執行數值運算任務。
