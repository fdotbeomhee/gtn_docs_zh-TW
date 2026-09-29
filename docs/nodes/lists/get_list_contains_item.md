# GetListContainsItem {: #getlistcontainsitem }

## 功能概述 {: #what-is-it }

GetListContainsItem 節點是一個實用工具節點，專門用於檢查特定項目是否存在於清單中。它提供布林值 (Boolean) 輸出，指出輸入清單中是否包含該目標元素。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用 GetListContainsItem 節點：

- 欲檢查清單中是否存在特定元素
- 欲驗證集合中是否具備所需的關鍵資料
- 依據元素是否存在來構建條件分支工作流程
- 欲根據項目存在與否對清單進行過濾或處理
- 欲在動態變更的清單結構中即時監控特定目標

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 GetListContainsItem 節點新增至工作流程中
1. 將清單連線至 "items" 輸入埠
1. 將欲檢查比對的目標元素連線至 "item" 輸入埠
1. 節點將自動進行檢查，並輸出該項目是否存在於清單中的判定結果

### 參數說明 {: #parameters }

- **items**：欲進行檢查的目標清單（清單型別輸入）
- **item**：欲在清單中尋找的目標元素（可為任意型別）
- **contains_item**：指出是否找到項目的布林值（找到為 `true`，未找到為 `false`）

### 輸出埠說明 {: #outputs }

- **contains_item**：布林值輸出（若找到目標元素則為 `true`，否則為 `false`）

## 範例展示 {: #example }

檢查任務佇列清單中是否存在特定待辦事項：

1. 將 GetListContainsItem 節點新增至工作流程中
1. 將現有的任務清單連線至 "items" 輸入埠
1. 將欲檢查的任務字串連線至 "item" 輸入埠
1. "contains_item" 輸出引腳將在清單包含該任務時輸出 `true`，未包含時輸出 `false`

## 重要注意事項 {: #important-notes }

- 當輸入的清單或比對目標發生變動時，節點會自動即時更新輸出結果。
- 若未提供清單或目標元素任一輸入，輸出必定為 `false`。
- 節點採用精確比對機制 (Exact Matching) 來查找項目。
- 輸出結果始終為標準布林值。

## 常見問題排解 {: #common-issues }

- **缺少輸入連線 (No Input)**：若清單或比對目標未正確接線，輸出將為 `false`。
- **無效輸入 (Invalid Input)**：若傳入的輸入並非合法清單，輸出將為 `false`。
- **型別不匹配 (Type Mismatch)**：目標元素的型別需與清單內元素型別相符才能正確比對。
- **字串大小寫敏感 (Case Sensitivity)**：比對文字字串時預設嚴格區分英文大小寫。
