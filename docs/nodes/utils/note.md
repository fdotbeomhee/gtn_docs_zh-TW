# Note {: #note }

## 功能概述 {: #what-is-it }

Note 節點允許您在工作流程畫布中建立說明性便箋筆記。它們不具備實際運算邏輯，主要用於在流程圖中提供視覺化展示、註解說明與文檔指引。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用 Note 節點：

- 欲描述工作流程中某個特定區塊或邏輯分支的功能職責
- 為自己記錄特定節點的配置技巧與注意事項備忘
- 為團隊其他協作成員提供流程使用指引與操作說明

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 Note 節點新增至工作流程中
1. 於 "note" 參數中輸入您所需的多行便箋說明文字

### 參數說明 {: #parameters }

- **note**：便箋筆記文字內容（字串型別，預設為空字串）

## 範例展示 {: #example }

為 Agent 節點建立一份關於如何使用 `prompt_context` 的教學便箋：

1. 將 Note 節點新增至工作流程中

1. 將 Note 節點拖曳移至目標 Agent 節點附近

1. 於便箋中輸入以下說明文字：

    ```
    欲使用 prompt_context，請先建立一個新的 KeyValuePair 節點。
    將 key 設定為類似 "style"，value 設定為 "haiku"。
    接著將其 dictionary 輸出連線至 Agent 節點的 prompt_context 埠。
    在 Agent 的提示詞中，使用 {{ }} 雙大括號指定何處要以 "haiku" 替換 "style"。範例：

    Tell me about skateboards in the style of a {{style}}
    ```

1. 將 Note 便箋擺放在顯眼易讀的位置，以利快速理解工作流程邏輯

## 重要注意事項 {: #important-notes }

- 便箋中輸入的換行符號會被完整保留。
- 支援任意合法的字串文字內容。
- 空字串屬於合法輸入。
- 無字元數量上限限制。

## 常見問題排解 {: #common-issues }

- 目前版本尚未支援自訂便箋文字大小 (Font size)。
- 目前版本尚未支援自訂便箋節點的背景顏色。
