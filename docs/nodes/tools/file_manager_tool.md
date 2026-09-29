# File Manager {: #file-manager }

## 功能概述 {: #what-is-it }

File Manager 工具節點是一個檔案操作工具，可掛載指派給 AI Agent 代理，協助模型在您指定的工作區目錄 (Workspace Directory) 中安全執行各類檔案讀寫與管理操作。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 賦予 AI Agent 讀取與寫入本機檔案的能力
- 自動化管理工作區目錄中的檔案結構
- 執行複製、移動、重新命名或刪除等檔案系統維護操作

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 File Manager 工具節點新增至工作流程中
1. 將其輸出引腳連線至需要檔案操作能力的節點（例如 Agent 節點的 tools 輸入埠）

### 輸出埠說明 {: #outputs }

- **tool**：配置完成的檔案管理工具物件 (Tool Object)，可直接供其他節點引用

## 範例展示 {: #example }

建立一個能自動歸檔與讀取日誌檔案的智慧 Agent：

1. 將 File Manager 工具節點新增至工作流程中
1. 將 "tool" 輸出埠連線至 Agent 節點的 "tools" 輸入埠
1. 該 Agent 隨後便能在對話執行過程中，依需自主讀取專案文件或將分析結果持久化寫入本機檔案

## 實作細節 {: #implementation-details }

File Manager 工具底層基於 Griptape 的 `FileManagerTool` 類別並搭配 `LocalFileManagerDriver` 實作。它提供了一個簡潔且受沙盒邊界保護的介面，專門用於在本機工作區目錄內進行檔案操作。

附註：雲端儲存 (Cloud Storage) 相關支援功能目前暫時停用，將於後續版本更新中正式提供。
