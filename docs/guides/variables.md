# 工作流程變數 (Workflow Variables) {: #workflow-variables }

**變數 (Variable)** 是一個獨立存在於流程（而不是單一節點）內部的具名數值。只需建立一次，即可在需要該數值的任何地方進行讀取或寫入 — 包含在文字欄位中透過 `{variable_name}` 進行行內替換。

!!! info "與目錄巨集路徑變數的差異"

    [巨集 (Macros)](projects/macros.md) 是專案系統用來動態建構**檔案路徑**的範本語法（如 `{outputs}/{file_name_base}.{file_extension}`）。工作流程變數則是創作者自行建立、並在工作流程節點參數之間儲存與重複使用的數值。兩者在底層共用相同的 `{name}` / `{name:spec}` 語法結構與格式規範，且在文字欄位中輸入 `{` 時，專案巨集（例如 `workspace_dir` 和 `workflow_name`）亦會與您自訂的變數一同列出 — 但巨集路徑範本是用於定義如何產生檔名，而工作流程變數則是您在執行期自訂並動態變更的數值。若您正在建構檔案儲存路徑，請參閱[巨集語法 (Macros)](projects/macros.md)。

## 何時使用變數 {: #when-to-use-a-variable }

假設工作流程中分佈在不同位置的三個提示詞 (Prompt) 欄位皆需要帶入同一個客戶名稱。從單一來源節點拉線連線至每個需要的欄位雖然可行，但會使畫布線條雜亂不堪；而且若日後想在欄位中直接打字而非拉線，連線機制便會失效。

變數無需任何實體連線即可優雅解決此問題：設定一次變數，即可在任意數量的文字參數中直接引用 `{project_name}`。一旦變更該變數的值，所有引用它的欄位在下一次執行時皆會自動取用最新數值。

## 建立變數 {: #creating-a-variable }

節點程式庫中的 **Variables** 分類提供了一組用於建立與管理變數的專用節點：

<!-- screenshot (#5166): the Variables category in the node library panel, showing Create Variable, Set Variable, Get Variable, Has Variable, Set Variables from Data, and Set Variable Substitution -->

- **Create Variable** — 建立具備名稱、型別與初始值的變數；若目前流程中已存在同名變數，則予以更新。將任何輸出連線至其 `value` 輸入端，系統會自動推斷該變數的資料型別。
- **Set Variable** — 設定變數的值；若該變數尚不存在則先行建立。其 `variable_name` 欄位為目前作用域中可用變數的下拉選單，並包含 **Create new variable** 選項，選取後會展開名稱輸入欄位。
- **Set Variables from Data** — 將 dict、JSON/YAML 字串或鍵值對清單一次轉換為多個變數。當您已手握現成的 JSON 資料（解析後的設定檔、API 回應結果）並希望每個鍵值自動成為獨立變數時極為實用，省去為每個鍵建立一個 **Set Variable** 節點的麻煩。
- **Get Variable** — 將變數的當前值讀取為標準輸出連接埠，用於連線至不支援行內 `{name}` 替換的下游節點。
- **Has Variable** — 檢查特定變數是否存在，以便進行條件分支處理（例如：僅在流程首次執行時建立變數）。

<!-- screenshot (#5166): a Create Variable node configured with variable_name "project_name" and a text value wired into its value input -->

**Create Variable** 與 **Set Variable** 一律在節點當前所屬的流程中建立變數（流程作用域，詳見下文）— 目前沒有任何節點可直接建立全域變數。這些節點上的 **scope** 設定項（位於 **Advanced** 面板下）控制的是它們去哪裡*尋找*既有變數，而非決定新變數的建立位置。

這些節點將其存取的變數視為即時狀態 (Live State)，而非僅解析一次的靜態值。**Create Variable** 在每次工作流程或該節點執行時，都會強制將自身重設為未解析 (Unresolved) 狀態，確保一律重新套用其當前的 `value`。**Get Variable**、**Set Variable** 與 **Has Variable** 每次被查詢解析狀態時，皆會主動比對變數的實際最新值；若與上次讀寫的數值不符，便會將自身標記為未解析 — 如此一來，即可即時感應在其他位置（另一個 **Set Variable** 節點或不同子流程）所做的變更，無需手動重新觸發。

## 作用域 (Scopes) {: #scopes }

每項變數請求 — create、get、set、list — 皆需指定 **scope (作用域)**，用以控制搜尋查找的範圍：

| 作用域 (Scope) | 行為說明 |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| `hierarchical` (預設) | 先搜尋目前流程，再逐層向上搜尋各父級流程直到根流程，最後搜尋全域變數。由最先相符的項目勝出。 |
| `current_flow_only` | 僅搜尋該節點所在的精確流程。徹底忽略父級流程與全域變數。 |
| `global_only` | 僅搜尋全域變數 — 即不歸屬於任何特定流程的變數。 |
| `all` | 包含所有流程中的所有變數，主要用於列舉遍歷（填充下拉選擇器或清單），而非單一數值解析。 |
|

在絕大多數情況下，`hierarchical` 都是最佳選擇：在最頂層流程中定義一次 `project_name`，其內部巢狀嵌套的所有子流程皆可直接讀取 `{project_name}` 而無需重新宣告。若某個巢狀子流程定義了專屬的同名 `project_name`，該巢狀定義會在子流程內部**遮蔽 (Shadow)** 父級的同名變數；而父級的數值依然完好無損，一旦跳出該子流程即可重新顯現。

簡而言之：距離讀取點最近的定義優先權最高，且流程層級變數優先於全域變數。

## 變數資料型別 {: #variable-types }

變數的 `type` 與一般節點參數上的型別標籤完全相同 — `str`、`int`、`float`、`bool`、`json`、圖像型別等。**Create Variable** 與 **Set Variable** 會根據連線至其 `value` 輸入端的資料自動推斷型別；除非您未接線而是在節點中直接輸入字面常數，否則無需手動設定。

僅有持有 `str` 或 `int` 數值（不含 `bool`）的變數，才能在文字欄位中透過 `{name}` 標籤進行行內替換 — 詳見下方的[文字欄位中的行內替換](#inline-substitution-in-text-fields)。持有其他型別（JSON、圖像、清單）的變數依然能與 **Get Variable** / **Set Variable** 完美搭配運作；只是無法以純標籤形式直接插入文字欄位中，因為圖像等物件無法以合理方式在行內彩現為文字。

## 在執行期間設定與讀取數值 {: #setting-and-reading-values-during-a-run }

若在節點之外直接針對引擎的請求 API 撰寫自動化腳本，相關的請求介面為 `CreateVariableRequest`、`GetVariableValueRequest` 以及 `SetVariableValueRequest` — 每個請求皆接受 `name` 與 `lookup_scope`。`SetVariableValueRequest` 會將下游所有引用該變數且已解析的節點標記為未解析，因此在編輯中途修改變數值，能正確使依賴舊值運算的所有項目失效並重新觸發。

對於行內替換，在純變數查找之上還有一層整合機制：當節點執行時，引擎會將其參數值中的所有 `{name}` 標籤，與由**您的工作流程變數加上專案唯讀巨集數值**（`workspace_dir`、`workflow_name` 以及專案範本中定義的所有目錄）合併而成的清單進行解析比對 — 若名稱發生衝突，以您自訂的工作流程變數為最高優先權。正是這種合併機制，使得文字欄位能同時並排引用 `{workflow_name}` 與 `{project_name}`，即便前者源自專案設定、後者源自您自訂的變數。

## 文字欄位中的行內替換 {: #inline-substitution-in-text-fields }

在任何支援替換的文字參數欄位中輸入 `{`，編輯器便會彈出變數選取器，列出目前作用域內的所有變數與專案巨集，並依來源分組呈現：

<!-- screenshot (#5166): a text parameter field with the { picker open, showing grouped variables and project macros -->

點選其中一項，或自行鍵入變數名稱並補齊右大括號 — `{project_name}` — 引擎便會在執行期自動替換為相應數值。若該名稱無法解析為作用域內的任何項目，標籤會完好保留在文字中（或者若是選用標籤則靜默丟棄 — 詳見下文），因此拼寫錯誤絕不會靜默輸出錯誤數值。

若您特別希望字面上的 `{literal text like this}` 保持原樣傳遞而不被當作標籤解析，可透過 **Set Variable Substitution** 節點在個別工作流程層級關閉此替換行為。

### 格式規範 (Format specs) {: #format-specs }

標籤可在 `:` 之後附加格式規範修飾元（由左至右依序套用），其語法與底層支援清單與[巨集語法 → 字串轉換](projects/macros.md#string-transformations)所記錄的完全一致：

| 格式規範 | 轉換效果 |
| ------------------ | --------------------------------- |
| `:lower` | 全小寫字母 |
| `:upper` | 全大寫字母 |
| `:title` | 首字大寫 (Title Case) |
| `:snake` | 蛇形命名法 (snake_case) |
| `:pascal` | 帕斯卡命名法 (PascalCase) |
| `:camel` | 駝峰命名法 (camelCase) |
| `:screaming_snake` | 全大寫蛇形 (SCREAMING_SNAKE_CASE) |
| `:slug` | 網址友善 Slug (url-safe-slug) |
| `:dot` | 點分隔命名 (dot.case) |
| `:abbrev` | 擷取每個單字的首字母 |
| `:trim` | 剔除頭尾空白字元 |

```
{project_name}            → "Autumn Campaign"
{project_name:lower}      → "autumn campaign"
{project_name:slug}       → "autumn-campaign"
{project_name:snake:upper} → "AUTUMN_CAMPAIGN"
```

此外，還有兩項繼承自巨集語法的機制在此處運作邏輯相同：

- **選用標記 `?`** — 當變數未找到時，`{project_name?}` 會彩現為空字串而非留下字面標籤，讓您能靈活建構在變數未設定時自動去除該子句的語句。
- **預設值 `|default`** — 當 `project_name` 不在作用域內時，`{project_name|untitled}` 會替換為 `untitled`。

數值補零填塞 (`:03`)、序號欄位 (`{###}`) 以及前導分隔符號 (`:^prefix`) 同樣由同一個底層引擎解析，在文字欄位中技術上完全有效，但它們主要是為了建構帶有自動遞增計數器的檔名而設計 — 如需此類行為請參閱[巨集語法 (Macros)](projects/macros.md)。對於一般文字替換，上述的字串轉換以及選用/預設值規範即可滿足絕大多數日常情境。

## 實戰範例：跨多個提示詞共用專案名稱 {: #worked-example-a-project-name-across-several-prompts }

假設有一個工作流程負責生成概念藝術圖，且多個提示詞欄位皆需要引用同一個專案名稱：

1. 在流程頂部放置一個 **Create Variable** 節點。將 `variable_name` 設為 `project_name`，在 `value` 中直接鍵入 `Autumn Campaign`（常數字面值無需接線），並讓 `variable_type` 維持自動推斷的 `str`。

1. 在提示詞建構節點的文字欄位中輸入：

    ```
    A cinematic key art poster for {project_name}, dramatic lighting, wide shot
    ```

1. 在流程另一處的第二個提示詞欄位中輸入：

    ```
    Concept sketch, {project_name:slug} style guide, muted palette
    ```

1. 執行一次流程。兩個提示詞皆會帶入 "Autumn Campaign"（或您輸入的任何文字），且第二個提示詞在套用 slug 規範處會成功彩現為 `autumn-campaign`。

<!-- screenshot (#5166): the finished flow with a Create Variable node feeding project_name into two prompt fields that show the resolved {project_name} text -->

若要在新的執行階段變更專案名稱，只需修改 **Create Variable** 節點上的數值（或拉線連入不同的來源）— 下一次執行流程時，所有引用 `{project_name}` 的欄位皆會自動更新，完全無需重新接線。
