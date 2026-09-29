# MergeKeyValuePair 節點 {: #mergekeyvaluepair-node }

## 功能概述 {: #what-is-it }

`MergeKeyValuePair` 節點用於將多個獨立的字典合併為單一統一字典。它最多支援接收四個輸入字典，並將其內部所有鍵值對聚合為單一輸出字典。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用 MergeKeyValuePair 節點：

- 欲將多個來源的字典整合成單一結構化實體
- 欲將來自不同模組的零散組態參數統一合併
- 欲聚合工作流程不同分支所產出的資料集合
- 欲自細碎的個別元件組裝出龐大複雜的參數字典
- 欲將工作流程不同階段產出的鍵值對進行跨階段彙整

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 MergeKeyValuePair 節點新增至工作流程中
1. 將最多四個輸出字典的節點輸出埠連線至此節點的輸入埠
1. 該節點將自動合併所有輸入字典並產出單一整合字典

### 參數說明 {: #parameters }

- **key_value_pair_1**：欲合併的第一個字典（字典型別輸入）
- **key_value_pair_2**：欲合併的第二個字典（字典型別輸入）
- **key_value_pair_3**：欲合併的第三個字典（字典型別輸入）
- **key_value_pair_4**：欲合併的第四個字典（字典型別輸入）

### 輸出埠說明 {: #outputs }

- **output**：聚合所有輸入字典鍵值對後的單一合併字典

## 範例展示 {: #example }

整合來自不同配置節點的設定參數：

1. 將 MergeKeyValuePair 節點新增至工作流程中
1. 將資料庫配置字典連線至 "key_value_pair_1"（例如：`{"host": "localhost", "port": 5432}`）
1. 將身分驗證設定字典連線至 "key_value_pair_2"（例如：`{"username": "admin", "password": "secure123"}`）
1. 將應用程式參數字典連線至 "key_value_pair_3"（例如：`{"app_name": "MyApp", "debug": true}`）
1. 輸出埠將會產出整合後的完整配置字典：`{"host": "localhost", "port": 5432, "username": "admin", "password": "secure123", "app_name": "MyApp", "debug": true}`

## 重要注意事項 {: #important-notes }

- **鍵名衝突覆寫原則**：若同一個鍵名出現在多個輸入字典中，**序號較後者將會覆寫前者**（例如 `key_value_pair_2` 中的同名鍵會覆寫 `key_value_pair_1`）。
- 節點會自動安全忽略未連線、為 `None` 或非字典型別的輸入引腳。
- 無需填滿全部四個輸入埠 — 節點可彈性支援 1 到 4 個輸入連線。
- 當預期存在同名鍵時，輸入埠的接線順序至關重要。
