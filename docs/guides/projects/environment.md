# 環境與內建變數 (Environment & Builtin Variables) {: #environment-builtin-variables }

## 環境變數 (Environment) {: #environment }

專案設定檔中的 `environment` 區塊承載自訂鍵值對。這些數值可在巨集與目錄的 `path_macro` 欄位中隨處引用：

```yaml
environment:
  RENDER_STYLE: "realistic"
  CLIENT_CODE: "ACME"
```

### 覆疊合併行為 {: #overlay-behavior }

專案設定檔中的環境變數條目會覆蓋合併於系統預設值之上。若某個鍵名已存在於預設範本中，您的數值將直接取代它；新鍵名則直接追加。

### 在環境變數值中引用其他變數 {: #referencing-other-variables-from-an-environment-value }

環境變數的數值本身即為巨集字串。透過 `{NAME}` 語法，數值可以自由引用系統內建變數、目錄名稱、其他專案環境變數或作業系統 Shell 環境變數：

假設啟動 Griptape Nodes 的終端機已預先導出了 `SHARED_DRIVE=/mnt/renders`：

```yaml
directories:
  outputs:
    # 直接引用 Shell 環境變數 — 無需在 `environment:` 中重複宣告
    path_macro: "{SHARED_DRIVE}/outputs"

environment:
  CLIENT_CODE: "ACME"
  # 引用系統內建變數
  PROJECT_RENDERS: "{project_dir}/renders"
  # 複合組合 Shell 環境變數、專案自訂環境變數與字面文字
  CLIENT_RENDERS: "{SHARED_DRIVE}/{CLIENT_CODE}/renders"
```

參照關係會依據[變數優先順序](#variable-priority)遞迴完成解析。系統會主動偵測循環參照（例如 `A: "{B}"` 且 `B: "{A}"`）並拋出巨集解析錯誤。

#### 傳統 `$VAR` 語法相容說明 {: #legacy-var-syntax }

為了向下相容，若環境變數的值**剛好僅為** `$NAME`（無前後綴文字、無其他巨集），當其**在巨集中被調用時**會自作業系統環境變數中展開：

```yaml
environment:
  OUTPUT_ROOT: "$RENDER_FARM_SHARE"  # 在巨集中生效：{OUTPUT_ROOT} -> /mnt/renders
```

此傳統格式存在嚴格局限：

- **僅限單一完整數值**。若追加任何字元（如 `"$SHARED_DRIVE/outputs"`）或相鄰文字，將**不被**展開，而被視為純文字字面常數。
- **僅在巨集中生效，不注入進程**。`$VAR` 僅在巨集解析時展開。它**不會**展開寫入 `os.environ` — 在子進程或自訂節點中調用 `os.environ.get("OUTPUT_ROOT")` 將取得裸字串 `"$RENDER_FARM_SHARE"`。
- **無法複合串接**。以 `$` 開頭的數值無法再引用其他專案變數、內建變數或目錄。

新專案請一律使用 `{NAME}` 標準語法 — 它支援完美複合串接，且在巨集與 `os.environ` 中皆能一致解析。

## 系統內建變數 (Builtin variables) {: #builtin-variables }

系統內建變數自動在所有巨集中提供。您無需定義它們 — 系統在執行期即時動態供應其數值。它們具備唯讀保護，不可被覆寫：

| 變數名稱 | 資料型別 | 說明 |
| ------------------ | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `project_dir` | directory | 專案基礎目錄的絕對路徑（包含 `griptape-nodes-project.yml` 的資料夾；若無專案設定檔則為工作區目錄） |
| `workspace_dir` | directory | 工作區目錄的絕對路徑（未宣告時預設為專案目錄；參閱[工作區組態解析順序](workspace.md#config-resolution-order)） |
| `workflow_name` | string | 當前正在執行的工作流程名稱 |
| `workflow_dir` | directory | 包含當前工作流程檔案的資料夾絕對路徑；若工作流程尚未存檔，則為建立時所在的初始資料夾 |
| `static_files_dir` | string | 靜態檔案子目錄名稱（取自應用程式設定，預設為 `staticfiles`） |

### 內建變數如何解析 {: #how-builtins-are-resolved }

內建變數是在巨集實際被求值的瞬間即時解析 — 而不是在載入專案設定檔時。這意味著：

- `workflow_name` 與 `workflow_dir` 精確反映當前正在點擊執行的具體工作流程。
- `project_dir` 精確反映所載入專案設定檔的磁碟實體路徑。
- `workspace_dir` 精確反映解析後的工作區路徑。

若某個必填的內建變數無法解析（例如在沒有執行任何工作流程的脫機狀態下調用 `workflow_name`），巨集解析將中斷報錯。若標記為選填（帶有 `?`），該區塊會被靜默省略。

### 情境規則中的內建變數應用 {: #builtin-variables-in-situation-macros }

`save_static_file` 情境規則同時結合了 `workflow_dir` 與 `static_files_dir`：

```
{workflow_dir?:/}{static_files_dir}/{file_name_base}.{file_extension}
```

若 `workflow_dir` 可用，靜態檔案存入該流程目錄下的子資料夾中；若不可用，`{workflow_dir?:/}` 自動省略，路徑轉換為相對於工作區根目錄。

### 尚未存檔的工作流程處理機制 {: #unsaved-workflows }

從未命名存檔過的工作流程缺少實體磁碟路徑，因此原本無法推導出 `workflow_dir`。當您在瀏覽特定資料夾時建立新流程，編輯器會主動將該資料夾通知引擎，在首次存檔前 `workflow_dir` 會暫時以此資料夾回應。首次存檔前生成的檔案會落於該資料夾中。

一旦手動儲存流程，`workflow_dir` 會切換為該已儲存檔案的真實所在目錄。先前已產生的實體檔案仍留於原地，但後續建構於 `{workflow_dir}` 之上的參照會對齊至新目錄。若編輯器未提供初始資料夾，`workflow_dir` 將保持不可用，直到首次完成存檔為止。

## 變數優先順序 (Variable priority) {: #variable-priority }

當巨集進行動態解析時，各來源變數依以下優先順序依序匹配（高優先權勝出）：

1. **系統內建變數 (Builtin variables)** — 具備最高優先權；絕不可被任何其他來源覆寫。
1. **目錄名稱 (Directory names)** — 取自專案的目錄架構定義；不可被調用端自訂變數覆寫。
1. **調用端傳入變數 (Caller-supplied variables)** — 請求路徑解析的節點或操作所明確傳遞的鍵值。
1. **衍生變數 (Derived variables)** — 依據上述變數與專案狀態自動計算、並在解析前注入的變數。若調用端已主動提供同名數值，衍生變數自動讓位。
1. **專案變數 (Project variables)** — 取自專案設定檔中的 `variables:` 區塊（參閱[專案變數 (Project Variables)](variables.md)）。僅限字串與整數參與。
1. **專案環境變數 (Project environment variables)** — 取自專案設定檔中的 `environment:` 區塊。支援遞迴解析。
1. **Shell 環境變數 (Shell environment variables)** — 最終退回來源。啟動 Griptape Nodes 終端機中導出的所有環境變數（包含 `HOME`、`USER` 等）皆可透過 `{NAME}` 在巨集中調用。同名的專案環境變數具備較高優先權。保留關鍵字（內建變數、目錄名稱）會自動遮蔽 Shell 變數。

若調用端試圖為內建變數或目錄傳遞與系統定義相衝突的數值，解析將拋出 `RESERVED_NAME_COLLISION` 錯誤。

### 衍生變數清單 {: #derived-variables }

| 變數名稱 | 衍生計算依據 | 事實來源規範 |
| -------------------------- | ------------------------------------------------------------------------ | --------------------------------------------------------------- |
| `file_extension_directory` | 檔案副檔名 `file_extension` 加上專案的 `file_extension_directories` 映射 | 參閱[副檔名目錄分流 (File Extension Directories)](file_extension_directories.md) |
