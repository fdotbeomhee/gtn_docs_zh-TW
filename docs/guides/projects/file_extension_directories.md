# 副檔名目錄分流 (File Extension Directories) {: #file-extension-directories }

`file_extension_directories` 是一套專案範本映射機制，能將檔案副檔名對齊至特定資料夾路徑片段，實現依檔案類型自動分流歸檔。當情境規則巨集引用衍生變數 `{file_extension_directory}` 時，專案系統會主動在此映射表中查找該檔案的副檔名，並替換為對應的資料夾數值。

最典型的應用：將輸出成品中的圖像、影片、音訊與文件自動分流存入 `outputs/` 下各自獨立的子資料夾中，而無需為每種類型撰寫繁瑣的情境規則。

## 快速上手範例 {: #quick-example }

```yaml
project_template_schema_version: "1.0.0"
name: "My Project"

file_extension_directories:
  png: "images"
  jpg: "images"
  mp4: "videos"
  wav: "audio"

situations:
  save_node_output:
    macro: "{outputs}/{file_extension_directory?:/}{node_name?:_}{file_name_base}{_index?:03}.{file_extension}"
```

在上述配置下：

```
file_extension="png" → outputs/images/Node_render.png
file_extension="mp4" → outputs/videos/Node_render.mp4
file_extension="xyz" → outputs/Node_render.xyz        (未映射：插槽自動折疊消失)
```

`{file_extension_directory?:/}` 中的 `?:/` 語法將該欄位標記為選填，且僅在數值存在時在尾端追加 `/` — 因此未在表中定義的副檔名會平穩落入情境目錄根目錄，絕不報錯中斷。

## 兩種數值形式 {: #two-value-forms }

設定值可為**純文字名稱**或**巨集範本**。

### 純文字名稱 (Plain name) {: #plain-name }

```yaml
file_extension_directories:
  png: "images"
```

字串按原樣字面採用，不執行二次解析。此為最常見場景且具備零運算開銷。

### 巨集範本數值 (Macro value) {: #macro-value }

```yaml
file_extension_directories:
  mp4: "{outputs}/videos"
  wav: "{workspace_dir}/shared/audio"
```

包含 `{...}` 的數值在代入情境巨集之前，會先針對專案內建變數、目錄架構定義以及調用端提供的上下文（如 `node_name`）完成動態解析。

巨集數值讓單一 `file_extension_directories` 表格具備將特定類型重新導向至完全不同根目錄（例如將影片輸出至共用網路磁碟）的能力 — 完全無需為特定類型新增情境規則。

### 巨集數值允許引用的來源 {: #what-a-macro-value-can-reference }

| 來源類型 | 範例標籤 | 是否可用？ |
| ----------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------- |
| 系統內建變數 | `{workspace_dir}`, `{workflow_dir}`, `{project_dir}`, `{project_name}`, `{static_files_dir}` | 是 |
| 目錄架構定義 | `{outputs}`, `{inputs}`, `{temp}`, 任何自訂目錄 | 是 |
| 調用端上下文 | `{node_name}`, `{parameter_name}`, `{sub_dirs}`, `{_index}` | 是 |
| 檔名組成部分 | `{file_name_base}`, `{file_extension}` | **否** — 分流層不處理檔名細節 |

檔名組成部分刻意予以排除：`file_extension_directories` 是負責決定檔案存入哪個資料夾的*路徑分流*層。檔名本身專屬於情境巨集的檔名定義區塊。

## 底層解析機制 {: #how-resolution-works }

`file_extension_directory` 屬於**衍生變數 (Derived variable)**。它不是系統原生內建變數，調用端亦不直接傳入。專案系統在情境巨集範本引用它時，會動態觸發一段精簡的推導規則：

1. 調用端指定情境名稱並提供變數（包含 `file_extension`）。
1. 在情境巨集正式解析前，推導規則觸發：
    - 若 `file_extension_directory` 已由調用端預先指定，規則主動讓位（調用端勝出）。
    - 否則，規則在當前專案的 `file_extension_directories` 表中搜尋 `file_extension`（不區分大小寫）。
    - 若值為純字串，直接作為變數值。
    - 若值為巨集，先將其解析為具體實體路徑字串。
1. 將計算出的數值注入變數集合中，情境巨集隨後按標準流程解析。

若查找失敗（副檔名為空、未載入專案、未映射該副檔名或解析報錯），該變數單純保持未設定。使用選填格式 `{file_extension_directory?:/}` 的巨集會乾淨降級為無前綴；使用必填格式 `{file_extension_directory}` 的巨集則會按常規拋出變數缺失錯誤。

## 與情境巨集結構的協同影響 {: #interaction-with-the-situation-macro }

引擎底層沒有任何私自重置父路徑的特異邏輯。最終產出的路徑完全忠實遵循情境巨集所書寫的結構：

| 情境巨集結構樣式 | 具體分流行為 |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `{outputs}/{file_extension_directory?:/}{file_name_base}.{ext}` | 分流目標為 `{outputs}` 下的**子資料夾**。數值必須為相對路徑。 |
| `{file_extension_directory?:/}{file_name_base}.{ext}` | 分流目標直接決定**根目錄**。數值可為絕對路徑以完全脫離 `{outputs}`。 |
| `{outputs}/{file_extension_directory?:/}...` 搭配絕對路徑數值 | 字串錯誤拼接 — `outputs//Volumes/share/videos/foo.mp4` — 這不是您想要的結果。 |

請依據您實際需要的分流架構挑選合適的情境巨集格式。

## 覆疊合併規則 (Overlay merge behavior) {: #overlay-merge-behavior }

`file_extension_directories` 按逐筆條目進行合併，與 `environment` 完全一致：

- 覆疊層中未提及的鍵值自動從基底繼承。
- 覆疊層中宣告的鍵值會覆寫基底中該副檔名的映射。
- 覆疊層中設為 `null` 的鍵值代表刪除標記 (Tombstone) — 基底中的該筆映射被徹底丟棄。

```yaml
# 繼承基底的圖像分流規則，將 mp4 導向其他位置，並移除 csv 分流
file_extension_directories:
  mp4: "{workspace_dir}/shared/videos"
  csv: null
```

## 調用端手動覆寫 {: #caller-override }

任何調用端皆可在傳入的變數集合中預先填充 `file_extension_directory`。一旦已存在數值，推導規則便會主動棄權並沿用調用端的值。這使得 UI 層級的自訂控制項（例如明確覆寫輸出資料夾）能繞過分流字典，同時繼續參與同一個情境巨集。

## 系統原生預設規則一覽 {: #whats-in-the-defaults }

系統預設範本預先內建了針對常見圖像、影片、音訊、文字以及 Python 原始碼副檔名的映射條目，分別自動分流歸檔至 `images`、`videos`、`audio`、`text` 與 `python` 子資料夾。您可自由覆寫特定項目、擴充全新映射，或透過 `null` 刪除任何不需要的預設分流。
