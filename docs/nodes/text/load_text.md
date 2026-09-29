# LoadText {: #loadtext }

## 功能概述 {: #what-is-it }

LoadText 節點是一個檔案讀取工具節點，專門用於從本機檔案系統讀取文字檔案內容。它讓您能夠將既有的文字文件匯入工作流程中進行後續處理。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用 LoadText 節點：

- 欲從本機現有的文字檔案中匯入內容
- 欲透過 AI Agent 代理處理本機文件
- 處理儲存的文字日誌 (Logs)、筆記 (Notes) 或報告 (Reports)
- 需要將外部文字檔案資料餵入工作流程中

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 LoadText 節點新增至工作流程中
1. 於 "path" 參數中指定目標文字檔案的本機路徑
1. 將其輸出埠連線至能處理文字內容的下游節點

### 參數說明 {: #parameters }

- **path**：欲載入的文字檔案之本機路徑（字串）

### 輸出埠說明 {: #outputs }

- **output**：讀取自檔案的完整純文字字串內容
- **path**：所載入檔案的路徑字串（與輸入路徑相同）

## 範例展示 {: #example }

讀取本機文字檔案並交由 AI Agent 進行摘要分析：

1. 將 LoadText 節點新增至工作流程中
1. 將 "path" 參數指定為類似 `"C:/Users/username/Documents/project_notes.txt"` 的路徑
1. 將 "output" 引腳連線至 Agent 節點的文字輸入埠
