# SearchReplaceText {: #searchreplacetext }

## 功能概述 {: #what-is-it }

SearchReplaceText 節點允許您對多行文字內容執行搜尋與替換操作。它同時支援純文字字串替換以及基於正規表示式 (Regular Expression, Regex) 的進階樣式比對與替換。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用 SearchReplaceText 節點：

- 欲藉由替換特定文字樣式來修改多行文字
- 欲執行區分大小寫 (Case-sensitive) 或不區分大小寫 (Case-insensitive) 的文字替換
- 運用正規表示式進行複雜的字串樣式比對
- 欲替換所有相符項目 (Replace All) 或僅替換首個出現的項目 (First Occurrence)
- 處理包含多行或多段落結構的長文字資料

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 SearchReplaceText 節點新增至工作流程中
1. 連線輸入埠或手動輸入待處理文字（支援多行文字）
1. 設定搜尋樣式 (search_pattern)
1. 設定替換文字 (replacement_text，支援多行文字）
1. 依據需求配置大小寫與正規表示式選項
1. 將輸出埠連線至接收文字輸入的下游節點

### 參數說明 {: #parameters }

- **input_text**：欲執行搜尋替換的多行文字（字串）
- **search_pattern**：欲搜尋的文字字串或正規表示式樣式（字串）
- **replacement_text**：用以替換的新文字內容（字串）
- **case_sensitive**：搜尋是否嚴格區分大小寫（布林值，預設為 `true`）
- **use_regex**：是否將搜尋樣式視為正規表示式解析（布林值，預設為 `false`）
- **replace_all**：是否替換所有相符項目，或是僅替換首次出現的項目（布林值，預設為 `true`）

### 輸出埠說明 {: #outputs }

- **output**：完成搜尋與替換操作後的多行文字字串

## 範例展示 {: #examples }

### 基礎文字替換 {: #simple-text-replacement }

1. 將 SearchReplaceText 節點新增至工作流程中
1. 將輸入文字設定為：
    ```
    Hello World
    Welcome to Griptape
    ```
1. 將搜尋樣式 (search_pattern) 設定為："World"
1. 將替換文字 (replacement_text) 設定為："Griptape"
1. 輸出結果將為：
    ```
    Hello Griptape
    Welcome to Griptape
    ```

### 不區分大小寫替換 {: #case-insensitive-replacement }

1. 將 SearchReplaceText 節點新增至工作流程中
1. 將輸入文字設定為：
    ```
    Hello WORLD
    Welcome to the WORLD
    ```
1. 將搜尋樣式設定為："world"
1. 將替換文字設定為："Griptape"
1. 將 case_sensitive 設為：`false`
1. 輸出結果將為：
    ```
    Hello Griptape
    Welcome to the Griptape
    ```

### 正規表示式範例 {: #regular-expression-examples }

!!! example "基礎正規表示式樣式"

    ```
    輸入: "Line 1\nLine 2\nLine 3"
    搜尋樣式: "Line \\d"
    替換文字: "Item"
    啟用 Regex: true
    輸出結果: "Item\nItem\nItem"
    ```

    此樣式會比對 "Line" 後方緊隨任意單一數字 (`\d`) 的字串。

!!! example "移除數字"

    ```
    輸入: "Product123, Item456, Order789"
    搜尋樣式: "\\d+"
    替換文字: ""
    啟用 Regex: true
    輸出結果: "Product, Item, Order"
    ```

    此樣式會比對一個或連續多個數字 (`\d+`) 並將其移除。

!!! example "單詞邊界比對"

    ```
    輸入: "cat in the hat"
    搜尋樣式: "\\bcat\\b"
    替換文字: "dog"
    啟用 Regex: true
    輸出結果: "dog in the hat"
    ```

    此樣式僅會精確比對獨立出現的單字 "cat"，而不會誤傷 "catch" 或 "scatter"。

!!! example "多行整行比對"

    ```
    輸入: "First line\nSecond line\nThird line"
    搜尋樣式: "^.*$"
    替換文字: "New line"
    啟用 Regex: true
    輸出結果: "New line\nNew line\nNew line"
    ```

    此樣式會比對整行文字（`^` 開頭、`.*` 任意字元、`$` 結尾）。

## 正規表示式快速參考 {: #regex-reference }

!!! note "常用正規表示式語法"

    | 語法樣式 | 說明 | 範例 |
    | ------- | -------------------------- | --------------------------------------- |
    | `\n`    | 比對換行符號 | `Line 1\nLine 2` |
    | `\s`    | 比對任何空白字元（空格、Tab、換行） | `Hello\sWorld` |
    | `\d`    | 比對任何單一數字 (0-9) | `\d+` 比對 "123" |
    | `[a-z]` | 比對任何小寫英文字母 | `[a-z]+` 比對 "hello" |
    | `[A-Z]` | 比對任何大寫英文字母 | `[A-Z]+` 比對 "WORLD" |
    | `.`     | 比對任意字元（預設換行除外） | `a.c` 比對 "abc" |
    | `*`     | 比對 0 次或多次 | `a*` 比對 ""、"a"、"aa" |
    | `+`     | 比對 1 次或多次 | `a+` 比對 "a"、"aa" |
    | `?`     | 比對 0 次或 1 次 | `a?` 比對 ""、"a" |
    | `\b`    | 單詞邊界 (Word boundary) | `\bcat\b` 比對 "cat" 但不比對 "catch" |

!!! warning "正規表示式模式注意"

    當啟用 regex 模式時，搜尋樣式中的特殊符號將被視為語法關鍵字解析。若欲進行符號本身的字面比對，請務必使用反斜線 (`\`) 進行跳脫 (Escape)。

!!! tip "純文字替換模式"

    當未啟用 regex 模式時，搜尋樣式會被嚴格視為字面純文字處理，所有特殊字元皆會被自動安全跳脫。對於一般的字串替換情境，此模式更為直覺安全。

## 重要注意事項 {: #important-notes }

- 使用正規表示式時，請確認特殊保留字元的跳脫是否正確。
- 不區分大小寫模式同時適用於純文字與正規表示式。
- 若在文字中未找到相符樣式，將原樣輸出原始文字且不拋出錯誤。
- 若傳入無效或語法錯誤的正規表示式，節點將安全回退並輸出原始文字。
- 換行符在輸入與輸出文字中皆會被完整保留。
- 啟用 regex 模式時，您可使用 `\\n` 比對跨行文字中的換行位置。

## 常見問題排解 {: #common-issues }

- 啟用 `use_regex` 時出現正規表示式語法解析錯誤。
- 混合大小寫敏感設定時產生非預期的替換結果。
- 面對超龐大文字區塊與高複雜度正規表示式時，可能引發效能負擔。

## 延伸參考資源 {: #additional-resources }

如需更詳盡的正規表示式教學與語法範例，歡迎參考：

- [Python Regular Expression HOWTO](https://docs.python.org/3/howto/regex.html)
- [Regex101](https://regex101.com/) - 線上互動式正規表示式測試與除錯工具
- [Regular-Expressions.info](https://www.regular-expressions.info/) - 深度正規表示式教學手冊
