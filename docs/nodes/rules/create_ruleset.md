# Ruleset {: #ruleset }

## 功能概述 {: #what-is-it }

Ruleset 節點允許您在工作流程中建立並輸出規則集 (Ruleset) 物件。規則集由名稱與一系列具體的規則文字清單組成，可直接供其他節點（例如 AI Agent 代理節點）引用。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 欲約束、規範或引導由大型語言模型 (LLM) 生成回應的下游節點行為

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 Ruleset 節點新增至工作流程中
1. 設定 "name" 參數以定義規則集的識別名稱（選填，預設為 "Behavior"）
1. 設定 "rules" 參數以定義該規則集的具體規則清單（選填，預設為空字串，以逗號分隔或多行定義）
1. 將其輸出引腳連線至流程中具備 Ruleset 參數的下游節點

### 參數說明 {: #parameters }

- **name**：規則集的識別名稱字串
- **rules**：規則集的具體規則內容
- **ruleset**：由本節點聚合生成的 Ruleset 規則集物件

### 輸出埠說明 {: #outputs }

- **ruleset**：輸出的 Ruleset 物件，可作為其他節點的屬性或輸入連線

## 範例展示 {: #example }

建立包含特定名稱與規則的新規則集：

1. 將 Ruleset 節點新增至工作流程中

1. 將 "name" 參數設定為：

    ```
    MyBehavioralRules
    ```

1. 將 "rules" 參數設定為：

    ```
    Rule 1: This is my first rule.
    Rule 2: This is my second rule.
    Rule 3: This is my third rule.
    ```

1. 將本節點的輸出埠連線至 Agent 節點的 Ruleset 輸入參數引腳

## 重要注意事項 {: #important-notes }

- Ruleset 節點僅支援字串型別的規則定義。
- 若未提供初始數值，節點名稱與規則將套用系統預設值。
- 本節點能與其他文字處理節點靈活串聯，依據動態輸入參數即時組裝 Ruleset 物件。
