# 目錄架構 (Directories) {: #directories }

目錄是邏輯名稱至實體路徑的映射機制。無須在各處硬編碼寫死諸如 `outputs/renders` 的實體路徑，您只需為其賦予邏輯名稱（如 `outputs`）並在巨集中引用該名稱。若日後需要變更輸出成品的存放位置，只需在單一集中處更新目錄定義即可全面生效。

## 系統預設目錄 {: #default-directories }

系統預設範本預先定義了六個標準目錄：

| 目錄名稱 | 預設相對路徑 | 說明 |
| --------------------------- | ---------------------------- | --------------------------------------------------------------------- |
| `inputs` | `inputs` | 匯入專案的外部檔案 — 上傳、複製與下載的資源。 |
| `outputs` | `outputs` | 工作流程執行期間由節點運算生成的輸出成品檔案。 |
| `temp` | `temp` | 運算期間產生的暫存快取檔案；可在多次執行之間安全刪除清空。 |
| `griptape-nodes-previews` | `.griptape-nodes-previews` | 自動生成的預覽與縮圖資產；鏡像對齊來源檔案的目錄層級。 |
| `griptape-nodes-metadata` | `.griptape-nodes-metadata` | 專案檔案的附隨詮釋資料 (Sidecar metadata)；鏡像對齊來源目錄。 |
| `griptape-nodes-thumbnails` | `.griptape-nodes-thumbnails` | 在 GUI 介面中展示的工作流程縮圖圖像。 |

所有預設路徑皆為相對路徑，並相對於專案基礎目錄進行解析。

## 在巨集中引用目錄名稱 {: #using-directory-names-in-macros }

目錄名稱可作為變數直接在任何巨集中引用。當巨集進行動態解析時，專案系統會自動將目錄名稱替換為其所配置的實體路徑：

```
範本：     {outputs}/{file_name_base}.{file_extension}
           ↓
解析結果： outputs/my_image.png
```

您無需自行傳遞目錄數值 — 它們自動取自目錄定義。目錄名稱屬於**保留關鍵字**：若嘗試傳入與目錄同名的自訂變數，系統將主動拋出錯誤以杜絕語意歧義。

## 自訂目錄實體路徑 {: #customizing-directory-paths }

在您的 `griptape-nodes-project.yml` 中覆寫目錄路徑：

```yaml
project_template_schema_version: "1.0.0"
name: "My Project"

directories:
  outputs:
    path_macro: "renders/final"
```

此時任何巨集中的 `{outputs}` 皆會自動解析為 `renders/final` 而非預設的 `outputs`。

## 目錄描述資訊 (Directory descriptions) {: #directory-descriptions }

每個目錄定義皆可配置選填的 `description` — 用於向團隊成員解釋該目錄用途與使用規範的人類可讀說明。這些描述會呈現在專案管理視窗 (GUI) 等介面中，讓手動維護的專案 YAML 更易理解。

```yaml
directories:
  outputs:
    path_macro: "renders/final"
    description: "準備交付給客戶的最終彩現輸出成品。"
```

`description` 為選填項，預設為 `null`。若需清除繼承自基底或父範本的描述，在覆疊層中將其設為 `null` 即可：

```yaml
directories:
  outputs:
    description: null
```

## 新增自訂目錄 {: #adding-new-directories }

新增預設範本中不存在的專案專屬目錄：

```yaml
directories:
  deliverables:
    path_macro: "client_deliverables"
```

新增完成後，`{deliverables}` 即可在任何巨集中隨處調用。

## 結合巨集與環境變數的目錄路徑 {: #directory-paths-with-macros }

`path_macro` 欄位完整支援波浪號 (`~`) 家目錄展開、巨集語法以及環境變數參照：

```yaml
directories:
  downloads:
    path_macro: "~/Downloads"
```

這會將 `downloads` 目錄映射至當前登入使用者的本機「下載」資料夾，無論在哪台作業系統電腦上運行皆能精準對齊。

```yaml
directories:
  outputs:
    path_macro: "$OUTPUT_BASE/renders"
```

若環境中（或專案的 `environment` 區塊中）設定了 `$OUTPUT_BASE`，路徑解析時便會自動完成替換。

您亦可在目錄路徑中引用系統內建變數：

```yaml
directories:
  outputs:
    path_macro: "{workflow_dir}/renders"
```

這使得 `outputs` 目錄相對於當前工作流程所在位置解析，而非相對於專案根基目錄。

## 跨平臺路徑配置 (Per-platform paths) {: #per-platform-paths }

目錄的 `path_macro` 既可為全域共用的單一字串，亦可定義為跨平臺映射物件，以適應不同作業系統間的路徑差異。當多位夥伴在 Linux、macOS 與 Windows 之間共用同一個工作區、且目錄需要指向不同的本機絕對路徑時極為實用：

```yaml
directories:
  scratch:
    path_macro:
      linux: "/mnt/fast-scratch"
      darwin: "/Volumes/scratch"
      windows: "D:/scratch"
      default: "{workspace_dir}/scratch"
```

在解析當下，引擎會挑選與當前作業系統相符的項目（`linux`、`darwin` 或 `windows`）。若當前平臺未定義，則退回 `default`。四個鍵值中至少必須配置一個，否則專案驗證將會報錯。

```yaml
directories:
  models:
    path_macro:
      darwin: "~/Library/Caches/models"
      default: "{workspace_dir}/.models"
```

跨平臺映射值支援相同的字串語法 — 波浪號展開、環境變數與巨集語法在各平臺中皆能正常運作。

該映射在覆疊合併時具備**不可分割性 (Atomic)**：提供跨平臺映射的子範本會完全替換父範本的 `path_macro`，而非逐鍵合併。若希望保留某個平臺的設定並覆寫另一個平臺，請在子範本中重述所有欲保留的平臺鍵值。

## 系統保留關鍵字名稱 {: #reserved-names }

目錄名稱在整個變數命名空間中屬於全域保留字。您無法在呼叫巨集時傳入與目錄同名的事後參數 — 若嘗試此操作系統將拋出錯誤。這項防護確保了目錄實體路徑絕不會因自訂變數命名衝突而被意外篡改。

內建系統變數（`project_dir`、`workspace_dir`、`workflow_name`、`workflow_dir`、`static_files_dir`）同樣屬於受保護的保留字且不可被覆寫。參閱[環境與內建變數 (Environment & Builtin Variables)](environment.md)。
