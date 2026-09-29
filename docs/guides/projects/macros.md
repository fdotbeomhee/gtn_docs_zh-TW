# 巨集語法 (Macros) {: #macros }

巨集是透過替換具名變數來動態生成檔案路徑的範本字串。巨集廣泛應用於情境規則 (Situations) 範本與目錄架構 (Directories) 定義中。

!!! note "與工作流程變數的區別"

    本頁專門介紹專案系統所使用的**檔案路徑巨集 (File-path macros)**。若您需要的是在工作流程內部建立與讀取的具名數值 — 包含在文字參數欄位中使用的 `{name}` 替換 — 請參閱[工作流程變數 (Workflow Variables)](../variables.md)。

在深入探討完整語法之前，以下兩個範例展示了巨集在實際運作中的具體樣貌：

```
範本：     {outputs}/{node_name?:_}{file_name_base}{_index?:03}.{file_extension}

提供完整變數時：
  outputs="outputs", node_name="ImageGen", file_name_base="render", _index=2, file_extension="png"
  → outputs/ImageGen_render002.png

省略選用變數時：
  outputs="outputs", file_name_base="render", file_extension="png"
  → outputs/render.png
```

`{outputs}` 是專案系統自動提供的目錄名稱。`{node_name?:_}` 屬於選填變數 — 當其存在時，數值後方會自動追加 `_`；若不存在，該整個大括號區塊會徹底消失。`{_index?:03}` 亦為選填項，存在時會自動補零填塞至 3 位數。

## 變數語法參考手冊 {: #variable-syntax-reference }

### 必填變數 (Required variable) {: #required-variable }

```
{variable_name}
```

該變數必須由調用端提供。若巨集解析時該變數缺失，解析流程將失敗並拋出錯誤。

### 選填變數 (Optional variable) {: #optional-variable }

```
{variable_name?}
```

`?` 標記將該變數宣告為選填。若未提供該變數，整個 `{}` 區塊（以及其中的任何格式規範修飾元）將從最終輸出中徹底去除。巨集的其餘部分繼續正常生成。

**尾隨格式 (Trailing form)**：`?` 亦可寫在最後一個格式規範的末端 — `{shot:upper?}` 與 `{shot?:upper}` 完全等價。兩種拼寫皆能將變數標記為選填。此規則同樣適用於序列簡稱：`{###:upper?}` 與 `{###?:upper}` 效果相同。若欲將 `?` 作為字面字元保留在分隔符號中，請將其用單引號包裹（`{shot:'lower?'}`）。

### 後置分隔符號格式 (Separator format) {: #separator-format }

```
{variable_name:separator}
```

在變數的值後方追加 `separator` 分隔字串。凡不是系統保留的關鍵字（參閱下文的字串轉換）且非純數字補零者，皆被視為後置分隔符號。

這在建構前綴路徑時極為實用，當變數缺失時能乾淨消失。例如，`{node_name?:_}` 會在節點名稱已知時在檔名前追加 `node_name_`，但名稱未知時絕不殘留多餘的底線：

```
{node_name?:_}{file_name_base}

  node_name="ImageGen", file_name_base="render"  →  ImageGen_render
  node_name 未提供,      file_name_base="render"  →  render
```

路徑分隔斜線運作邏輯相同 — `{sub_dirs?:/}` 僅在指定了子目錄時追加子資料夾路徑前綴：

```
{outputs}/{sub_dirs?:/}{file_name_base}.{file_extension}

  sub_dirs="lighting/pass_a", file_name_base="render", file_extension="exr"
  → outputs/lighting/pass_a/render.exr

  sub_dirs 未提供, file_name_base="render", file_extension="exr"
  → outputs/render.exr
```

### 前置分隔符號 (Leading separator) {: #leading-separator }

```
{variable_name:^prefix}
```

為[後置分隔符號格式](#separator-format)的鏡像機制，但其文字內容會**前置新增**在變數值之前。格式以格式規範開頭的 `^` 進行標記；`^` 之後的所有字元皆為字面字串前綴。僅在該變數存在輸出時才彩現 — 未賦值的選填變數會連同其前置分隔符號一同消失。

```
{file_name_base}{version?:^_v}.{file_extension}

  file_name_base="render", version=3, file_extension="png"  →  render_v3.png
  file_name_base="render", version 未提供                   →  render.png
```

最核心的經典實踐：與[序列插槽](#sequence-slot)搭配使用，建立隨序列自動出現與消失的版本後綴：

```
render{###?:^_v}.png

  第 1 次儲存 (插槽省略)  →  render.png
  第 2 次儲存 (插槽觸發)  →  render_v001.png
  第 3 次儲存             →  render_v002.png
```

在一般文字中同樣通用：

```
Hello, {name?}!{intro?:^ Nice to meet you.}

  name="Alice", intro="y"   →  Hello, Alice! Nice to meet you.y
  name="Alice", intro 未提供 →  Hello, Alice!
  name 未提供,  intro 未提供 →  Hello, !
```

**組合規則**：

- 單一變數最多只能配置**一個**前置分隔符號。
- 無論您在範本中的書寫位置為何，前置分隔符號一律在該變數的所有其他格式規範**執行之後**才套用。`{shot:03:^_v}` 與 `{shot:^_v:03}` 在 `shot=5` 時皆會彩現為 `_v005` — 解析器會將前置修飾元正規化至規範清單的末尾，確保順序絕不干擾前綴結構。

**語法錯誤規範**：

| 錯誤代碼 | 肇因說明 |
| ----------------------------- | ------------------------------------------ |
| `EMPTY_LEADING_SEPARATOR` | 寫入 `:^` 但脫字號後未帶任何文字 |
| `MULTIPLE_LEADING_SEPARATORS` | 在同一個變數上宣告了兩個以上的 `:^` 前置修飾元 |

### 數值補零填塞 (Numeric padding) {: #numeric-padding }

```
{variable_name:03}
```

將整數數值以 0 補齊至指定的固定寬度。該變數必須為整數型別。

```
{_index:03}   當 _index = 5   → "005"
{_index:04}   當 _index = 12  → "0012"
```

常用於 `create_new` 碰撞衝突策略下的自動遞增檔名。在單一未解析變數上套用補零規範 (`:NN`) 即代表啟用自動遞增：首次儲存落於索引 `1`（或選用形式時省略），後續儲存依同一個範本循序遞增 — 整個序列皆會保持相同的補零寬度。

- **選填格式** `{_index?:03}` — 首次儲存時省略，碰撞衝突時依序產生 `_001`、`_002` 等（保持補零寬度）。
- **必填格式** `{_index:03}` — 從首次儲存起即強制顯示：`_001`、`_002`、`_003` 等，全序列寬度一致。

```
範本： {file_name_base}_v{_index:03}.{file_extension}

  第 1 次儲存 → render_v001.png
  第 2 次儲存 → render_v002.png
  第 3 次儲存 → render_v003.png
```

變數名稱不強制限定為 `_index`；任何帶有 `:NN` 補零規範的單一未解析必填變數皆會被系統自動分派遞增。若未加上補零修飾元，未解析的必填變數會被視為缺少必要繫結（組態配置錯誤）並中止儲存 — 這可防止使用者因忘記連線 `{shot}` 而被系統意外填充為 `1, 2, 3, …`。

### 序列插槽 (`{###}`) (Sequence slot) {: #sequence-slot }

```
{#}       → 至少 1 位數 (1, 2, ..., 9, 10, 11, ...)
{###}     → 至少 3 位數 (001, 002, ..., 999, 1000, ...)
{####}    → 至少 4 位數 (0001, 0002, ..., 9999, 10000, ...)
{##?}     → 至少 2 位數，選填（首次儲存省略；碰撞時補上 01, 02 等）
```

在 **`{}` 大括號內部** 連續書寫 `#` 字元是定義序列插槽的明確標準語法。每個 `#` 代表**最小**彩現位數寬度。低於 `10 ^ width` 的數值會自動補零至該寬度；大於或等於該寬度的數值則按自然寬度彩現（絕不截斷）。這與 ffmpeg (`%03d`)、Houdini (`$F4`)、Nuke (`####`) 以及 Python 的 `:03` 格式規範完全相符。

在大括號內部追加 `?`（如 `{##?}`）可將該插槽宣告為**選填**。選填插槽在首次儲存時會被省略，僅在發生檔名衝突時才動態填入。

```
範本： {file_name_base}_v{###}.{file_extension}

  第 1 次儲存    → render_v001.png
  第 2 次儲存    → render_v002.png
  ...
  第 999 次儲存  → render_v999.png
  第 1000 次儲存 → render_v1000.png   (溢位：展示 4 位數，不截斷)
```

```
選填範本： {file_name_base}{##?}.{file_extension}

  第 1 次儲存 → render.png            (插槽省略)
  第 2 次儲存 → render01.png          (發生衝突時自動填入)
  第 3 次儲存 → render02.png
```

每當需要系統自動分配序列索引時，請優先使用 `{###}`。它明確指示「此插槽即為 `create_new` 碰撞時應遞增的位置」，無需依賴數值補零推斷，因此巨集編寫者若確實需要使用者手動傳入的 `{shot:03}` 變數時，能清楚區隔無歧義。

**為何需要 `{}` 大括號包覆**：巨集範本經常出現在裸寫 `#` 具有其他特殊意義的場合（Markdown 標頭、程式碼註解、Shell 腳本）。透過大括號包裹能確保語法與一般變數邊界完全一致，無需額外對靜態文字中的 `#` 制定跳脫規則。

**單一巨集僅允許一個序列插槽**：包含兩個 `{###}` 區塊的範本（如 `{###}_take_{##}.png`）會在解析時被直接拒絕 — 因為系統無法推測碰撞時應遞增哪一個插槽。若需多組序號，第二組數字請定義為明確的 `{var}` 變數。

**與 `{_index:NN}` 的相容關係**：在底層實作中，`{###}` 會自動解語法糖為帶有序列格式標記的 `_index` 變數。舊式的 `{_index:03}` / `{_index?:03}` 語法仍受向下相容支援，但官方全面推薦採用 `{###}` 格式。

### 未解析序列插槽的處理機制 {: #unresolved-sequence-slots }

必填的 `{###}` 插槽在正式寫入磁碟並由寫入路徑分派數值之前是沒有內容的。任何在實體分派*之前*需要預先解析巨集的程式碼（例如節點預覽其輸出目的地、UI 判斷使用者輸入為絕對路徑或相對路徑），皆必須指示解析器如何處理該空白插槽。`GetPathForMacroRequest` 透過 `unresolved_sequence_slot_behavior` 參數提供以下列舉選項 (`UnresolvedSequenceSlotBehavior`)：

| 行為選項 | 彩現結果 | 適用情境 |
| ------------------------- | ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `FAIL` *(預設)* | 拋出 `MISSING_REQUIRED_VARIABLES` 錯誤 | **實體寫入路徑** — 此錯誤是 `on_write_file_request` 用於設定初始索引並在碰撞時重試的關鍵訊號。任何其他程式碼皆不應覆寫此預設值。 |
| `RENDER_SEQUENCE_PATTERN` | `###`（或 `####`，對齊來源寬度） | **純介面展示**。將插槽彩現為裸字串標記（對齊 ffmpeg / Houdini / Nuke 標準習慣），方便路徑呈現磁碟檔案的抽象格式。絕不可將此字串傳入任何檔案 I/O 動作中，因為它不是合法的實體路徑。 |
| `START_AT_ZERO` | `000` | 在首次儲存前預覽從 0 開始計數的序列。 |
| `START_AT_ONE` | `001` | 預覽「首次儲存實際會落在哪個檔名」— 與寫入路徑的種子索引精確吻合。 |

選填插槽 (`{###?}`) 不受影響 — 未繫結時已自動省略，因此該旗標僅對必填插槽生效。

### 字串轉換規範 (String transformations) {: #string-transformations }

| 格式修飾元 | 說明 | 轉換範例 |
| ------------------ | ------------------------------------------------ | ------------------- |
| `:lower` | 全小寫字母 | `"my autumn shoot"` |
| `:upper` | 全大寫字母 | `"MY AUTUMN SHOOT"` |
| `:title` | 首字大寫 (Title Case) | `"My Autumn Shoot"` |
| `:snake` | 蛇形命名法 (snake_case) | `"my_autumn_shoot"` |
| `:pascal` | 帕斯卡命名法 (PascalCase) | `"MyAutumnShoot"` |
| `:camel` | 駝峰命名法 (camelCase) | `"myAutumnShoot"` |
| `:screaming_snake` | 全大寫蛇形 (SCREAMING_SNAKE_CASE) | `"MY_AUTUMN_SHOOT"` |
| `:slug` | 網址友善 Slug（空白轉連字號，移除非英數字元） | `"my-autumn-shoot"` |
| `:dot` | 點分隔命名 (dot.case) | `"my.autumn.shoot"` |
| `:abbrev` | 擷取每個單字的首字母 | `"MAS"` |
| `:trim` | 剔除頭尾空白字元 | `"My Autumn Shoot"` |

例如，當 `workflow_name` 為 `"My Autumn Shoot"` 時：

```
{workflow_name:lower}           →  "my autumn shoot"
{workflow_name:upper}           →  "MY AUTUMN SHOOT"
{workflow_name:title}           →  "My Autumn Shoot"
{workflow_name:snake}           →  "my_autumn_shoot"
{workflow_name:pascal}          →  "MyAutumnShoot"
{workflow_name:camel}           →  "myAutumnShoot"
{workflow_name:screaming_snake} →  "MY_AUTUMN_SHOOT"
{workflow_name:slug}            →  "my-autumn-shoot"
{workflow_name:dot}             →  "my.autumn.shoot"
{workflow_name:abbrev}          →  "MAS"
```

`:snake`、`:pascal`、`:camel`、`:dot` 與 `:screaming_snake` 能依據大小寫轉換點自動分割單字，因此即便輸入為駝峰或帕斯卡形式，`{varName:snake}` 亦能正確產出 `"var_name"`。

`:trim` 最適合與其他轉換鏈式串接，例如 `{name:trim:snake}` 會先剔除頭尾空格，再轉換為 snake_case。

### 預設值 (Default value) {: #default-value }

```
{variable_name|default_value}
```

若該變數未被傳入，則採用 `default_value` 作為數值替代。

```
{workflow_name|untitled}   → 若 workflow_name 未提供，則使用 "untitled"
```

### 鏈式串接格式規範 (Chaining format specs) {: #chaining-format-specs }

多個格式修飾元以 `:` 分隔，並由左至右依序套用。若使用了後置分隔符號，它必須放置於首位：

```
{variable_name:_:lower}    → 轉換為全小寫並在尾端追加底線
{variable_name:lower:slug} → 先轉為小寫，再轉換為 slug
```

### 帶引號的分隔符號 (Quoted separators) {: #quoted-separators }

若您的分隔文字碰巧與 `lower` 或 `upper` 等關鍵字重名，請使用單引號將其包裹以視為字面常數：

```
{variable_name:'lower'}    → 將文字 "lower" 作為後置分隔符號追加
```

## 巨集解析流程 {: #resolution }

當巨集進行動態解析時，專案系統會自動代入目錄名稱與內建變數。您只需提供操作專屬的變數（如 `file_name_base` 與 `file_extension`）。

以解析 `save_node_output` 情境規則為例：

```
範本：     {outputs}/{sub_dirs?:/}{node_name?:_}{file_name_base}{_index?:03}.{file_extension}

自動提供： outputs → 自 "outputs" 目錄定義解析 → "outputs"
外部傳入： node_name="StyleTransfer", file_name_base="portrait", _index=3, file_extension="png"
最終結果： outputs/StyleTransfer_portrait003.png
```

目錄名稱（如 `outputs`）會自動展開為所配置的路徑。參閱[目錄架構 (Directories)](directories.md)。

系統內建變數（如 `workflow_name`、`project_dir`）亦會自動供應。參閱[環境與內建變數 (Environment & Builtin Variables)](environment.md)。

## 反向路徑比對 (Reverse matching) {: #reverse-matching }

巨集系統亦支援反向推導：給定一個實體磁碟路徑與巨集範本，反向提取出各變數的具體數值。當系統需要判定某個檔案是否歸屬於已知專案目錄、以及檔名中編碼了哪些詮釋資料時（例如 `create_versioned_workflow` 讀取現有檔案以判定應遞增哪個 `_index`），便會調用此機制。

公開 API 為 `ParsedMacro.extract_variables(path, known_variables, secrets_manager)`（或用於布林判定的 `matches(...)`）。

基本範例：

```
範本：     {outputs}/{node_name?:_}{file_name_base}{_index?:03}.{file_extension}
路徑：     outputs/StyleTransfer_portrait003.png
提取結果： outputs="outputs", node_name="StyleTransfer", file_name_base="portrait", _index=3, file_extension="png"
```

### 反向提取如何判定各變數的截斷邊界 {: #how-extraction-decides-where-each-variable-ends }

對於類似 `{a}/{b}/{c}.{ext}` 的範本，提取器由左至右依序巡訪。每個變數的數值在遇到下一個**定錨點 (Anchor)**（必須在路徑中出現的固定字串）處截斷。存在兩種定錨點：

- **下一片段定錨點 (Next-segment anchor)**：後續靜態字串片段（例如 `.png`），或後續變數的前置分隔符號前綴（例如 `{###?:^_v}` 中的 `_v`）。
- **自身定錨點 (Self-anchor)**：變數自身的後置分隔符號（例如 `{node_name?:_}` 中的 `_`）。

提取器挑選能產生最緊湊、內部自洽分割的方案。當兩者皆可用時，搜尋方向取決於後續內容：若下一片段為靜態文字，取靜態文字前最晚出現的自身定錨點；若下一片段為另一個變數，取最早出現的自身定錨點（為後續變數保留消耗空間）。這正是 `{a?:_}{b?:_}file.png` 比對 `first_second_file.png` 時能正確分割為 `a=first, b=second` 而非 `a=first_second, b=""` 的關鍵。

### 選填變數 (`?`) 的歧義消除機制 {: #optional-variables-how-ambiguity-is-resolved }

路徑寫入時，選填變數可能曾輸出亦可能被省略。反向比對會枚舉所有 **2ᵏ 種可能性組合**（*k* 為範本中未繫結的選填變數數量），對每種組合進行提取，並透過正向來回驗證 (Forward round-trip) 進行檢驗：將提取出的數值重新帶入範本解析，生成結果必須與傳入的路徑字節完全一致。

組合依**漢明權重降序 (Popcount-descending)** 嘗試 — 優先採用資訊保留最多（選填變數輸出最多）的詮釋方案。第一個來回驗證成功的組合即為勝出解。

### 建構無歧義的範本指南 {: #building-unambiguous-templates }

當兩個變數並排且中間沒有任何固定文字分隔時，在文法上具備天生歧義性 — 文法無法判定邊界字元歸屬於哪一側。

**設計最佳實踐**：

- 在相鄰變數之間務必放置**靜態分隔符號**（或獨特的前置分隔符號前綴）。`{name}_{version}` 毫無歧義；而 `{name}{version}` 則容易產生混淆。
- **帶有前置分隔符號的序列插槽** — `{###?:^_v}` — 是版本化檔名的官方推薦標準寫法。`_v` 前綴是明確的定錨點，能協助提取器精準定位版本邊界。
- 透過 `known_variables` 預先傳入已知數值。每指定一個已知變數，即可從 2ᵏ 搜尋空間中剔除一個維度，徹底消除歧義。

### 語法錯誤提示 {: #syntax-errors }

巨集解析器在語法錯誤時會附帶精確的字元位置，方便排查：

- 未封閉的大括號：`{variable_name`（缺少結尾 `}`）
- 未成對的右大括號：`variable}name`
- 巢狀大括號：`{outer{inner}}`
- 空變數標籤：`{}`
