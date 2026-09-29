# KeyValuePair 節點 {: #keyvaluepair-node }

## 功能概述 {: #what-is-it }

`KeyValuePair` 節點用於建立一個包含單一鍵值對 (Key-Value Pair) 的基礎字典。它接收一個鍵名 (Key) 與一個數值 (Value) 作為輸入，並產出只包含該單一映射關係的字典物件。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用 KeyValuePair 節點：

- 欲建立一個僅含單一鍵值映射的極簡字典
- 欲動態注入或組裝特定的組態參數
- 欲以模組化方式逐步拼裝大型資料結構
- 欲將兩個獨立的純文字或數值包裝為標準字典格式
- 為僅接收字典輸入的下游節點準備單一參數

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 KeyValuePair 節點新增至工作流程中
1. 將 "key" 參數設定為欲指派的字典鍵名
1. 將 "value" 參數設定為與該鍵名關聯的具體數值
1. 將 "dictionary" 輸出埠連線至需要字典輸入的下游節點

### 參數說明 {: #parameters }

- **key**：用作字典鍵名的字串（型別：string）
- **value**：與該鍵名關聯的字串數值（型別：string，完整支援多行文字）

### 輸出埠說明 {: #outputs }

- **dictionary**：包含該單一鍵值對的產出字典物件

## 範例展示 {: #example }

為後續節點動態設定單一組態選項：

1. 將 KeyValuePair 節點新增至工作流程中
1. 將 "key" 參數設定為 `"max_tokens"`
1. 將 "value" 參數設定為 `"1024"`
1. 將 "dictionary" 輸出埠連線至接收組態設定的節點

產出的字典結構為：`{"max_tokens": "1024"}`

該字典隨後可被工作流程中任何需要讀取此配置參數的節點直接調用。

## 重要注意事項 {: #important-notes }

- 在此節點中，鍵名與數值預設皆以字串 (String) 形式處理。
- `value` 參數支援輸入多行長文本。
- 若您需要一次建立包含多個鍵值對的複合字典，可建立多個 KeyValuePair 節點並透過 Merge Key Value Pairs 節點將它們合併為單一字典。
