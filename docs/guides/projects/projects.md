# 專案設定檔與繼承 (Projects) {: #projects }

專案範本是一套定義檔案如何組織與儲存的組態設定。它是[情境規則 (Situations)](situations.md)、[目錄架構 (Directories)](directories.md)以及[環境變數 (Environment variables)](environment.md)的有機結合，每個需要儲存檔案的節點皆會主動查詢這些規範。

預設情況下，Griptape Nodes 原生隨附一套內建專案範本，能妥善處理最常見的日常任務 — 將生成圖像、音訊及其他輸出成品儲存至工作區內的 `outputs` 資料夾中。而專案設定檔讓您能自由自訂上述所有行為：重新導向特定類型檔案的存放位置、新增具名的儲存路徑（例如 `renders/4k`），或注入專案專屬的環境變數。您只需在設定檔中宣告欲變更的項目；其餘未設定之處將持續套用[預設情境規則](situations.md#default-situations)與[預設目錄架構](directories.md#default-directories)。

## 專案設定檔案 {: #the-project-file }

您在工作區層級的自訂設定儲存於以下命名的檔案中：

```
griptape-nodes-project.yml
```

請將此檔案放置於您的工作區目錄中。該檔案為選填項目 — 若不存在，系統將自動套用[系統預設值](#the-system-defaults)。

## 專案檔案結構範例 {: #project-file-structure }

```yaml
project_template_schema_version: "1.0.0"
name: "My Project"
description: "選填描述"

directories:
  outputs:
    path_macro: "renders"       # 覆寫 outputs 目錄路徑
  renders_4k:                   # 新增一個具名目錄
    path_macro: "renders/4k"

situations:
  save_node_output:             # 覆寫現有的情境規則
    macro: "{outputs}/{workflow_name?:_}{file_name_base}{_index?:03}.{file_extension}"
  archive_render:               # 新增自訂情境規則
    macro: "{renders_4k}/{workflow_name?:_}{file_name_base}.{file_extension}"
    policy:
      on_collision: overwrite
      create_dirs: true

environment:
  PROJECT_CODENAME: "aurora"

file_extension_directories:
  png: "images"
  mp4: "{outputs}/videos"
```

### 欄位完整參考手冊 {: #fields-reference }

| 欄位名稱 | 必填 | 說明 |
| --------------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `project_template_schema_version` | 是 | 專案的綱要版本。其**主版號 (Major)** 決定了繼承基準線（參閱[綱要版本](#schema-versions)）；新專案一律以最新版 (`"1.0.0"`) 建立。舊版主版號（如 `"0.5.1"`）仍能基於自身的基準線穩定載入。 |
| `name` | 是 | 該專案的人類可讀顯示名稱 |
| `description` | 否 | 選填的專案備忘描述 |
| `parent_project_path` | 否 | 父專案 YAML 的實體路徑；父專案合併後的範本將作為本專案的基礎。參閱[父專案繼承](#parent-projects)。 |
| `workspace_dir` | 否 | 本專案作為工作區的實體目錄；優先權最高的工作區來源。可為字串（絕對路徑或相對於此 YAML 的相對路徑）或跨平臺映射物件。不繼承自父專案。參閱[工作區目錄](#workspace-directory)。 |
| `libraries_dir` | 否 | 本專案安裝與解析節點程式庫的實體目錄。可為字串（絕對路徑或相對於此 YAML 的相對路徑）或跨平臺映射物件。**會沿父專案鏈向下繼承**（與 `workspace_dir` 不同），使子專案能共用父專案的程式庫。參閱[節點程式庫目錄](#libraries-directory)。 |
| `situations` | 否 | 情境規則的覆寫與新增字典 |
| `directories` | 否 | 目錄架構的覆寫與新增字典 |
| `environment` | 否 | 自訂鍵值變數的字典 |
| `file_extension_directories` | 否 | 副檔名至資料夾的分流映射；參閱[副檔名目錄分流 (File Extension Directories)](file_extension_directories.md) |

### 情境規則欄位 (Situation fields) {: #situation-fields }

`situations` 下的每筆條目皆以情境名稱為鍵名。您可以僅提供欲變更欄位的子集：

| 欄位名稱 | 新增時必填 | 說明 |
| --------------------- | ---------------- | -------------------------------------------------- |
| `macro` | 是 | 用於建構檔案路徑的巨集範本字串 |
| `policy` | 是 | 必須同時包含 `on_collision` 與 `create_dirs` |
| `policy.on_collision` | 是 | 可選值：`create_new`、`overwrite`、`fail` |
| `policy.create_dirs` | 是 | 設為 `true` 則在目標資料夾不存在時自動建立 |
| `fallback` | 否 | 若本情境處理失敗時，轉為調用的備援情境名稱 |
| `description` | 否 | 人類可讀的功能描述說明 |

當*修改*現有情境規則時，您只需填寫欲異動的欄位即可。當*新增*全新情境時，`macro` 與 `policy` 為必填項。

### 目錄設定欄位 (Directory fields) {: #directory-fields }

`directories` 下的每筆條目皆以邏輯名稱為鍵名：

| 欄位名稱 | 新增時必填 | 說明 |
| ------------ | ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `path_macro` | 是 | 路徑字串（可包含巨集語法或環境變數參照），或跨平臺映射物件。參閱[目錄架構 → 跨平臺路徑配置](directories.md#per-platform-paths)。 |

## 設定合併模型 (The merge model) {: #the-merge-model }

專案系統採用分層疊加模型：

1. **系統預設值 (System defaults)** — Griptape Nodes 隨附的完整內建專案範本。它定義了所有預設情境規則與目錄，一律最先被載入。
1. **父專案繼承鏈 (Parent chain，選用)** — 當專案宣告了 `parent_project_path` 時，父專案合併後的範本會被遞迴載入，並取代系統預設值成為本專案的底層基底。參閱[父專案繼承](#parent-projects)。
1. **本專案覆疊層 (This project's overlay)** — 當前 `griptape-nodes-project.yml` 的檔案內容。它會覆蓋合併於上述基底*之上*。

合併行為具備遞加性與欄位級精細度：

- 覆疊層中的情境與目錄會與基底合併。與基底同名的覆疊情境僅會變更您明確指定的欄位（例如僅覆寫巨集，或僅修改覆寫原則）。全新名稱的情境則會與基底並存新增。
- 覆疊層中的環境變數會覆寫基底中的同名鍵值。新鍵值則直接追加。
- `file_extension_directories` 依鍵名比照環境變數邏輯進行合併。傳入 `null` 值代表刪除 (Tombstone) 基底中的該筆映射。
- `name` 欄位一律直接取自當前覆疊層（必填）。

您絕不需要重複撰寫繼承而來的數值。專案設定檔中只需包含您欲自訂變更的項目。

## 父專案繼承 (Parent projects) {: #parent-projects }

專案可以宣告另一個專案作為其父級。父專案完全合併後的範本（在父專案完成解析其*自身*父繼承鏈之後）會成為基底，本專案的覆疊設定再疊加於其上。這讓您能在單一集中位置維護共用基礎配置 — 全團隊統一的目錄結構、固定環境變數集合、共用自訂情境 — 並讓衍生子專案直接繼承，省去重複配置的負擔。

```yaml
project_template_schema_version: "1.0.0"
name: "Marketing Renders"
parent_project_path: "../team-base/griptape-nodes-project.yml"

# 只需宣告與父專案不同的差異項目
directories:
  outputs:
    path_macro: "renders/marketing"
```

### 路徑格式 (Path forms) {: #path-forms }

`parent_project_path` 支援兩種書寫格式：

| 格式 | 範例 | 說明 |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| 相對路徑 | `../team-base/griptape-nodes-project.yml` | 推薦採用。相對於*本*專案 YAML 檔案所在目錄解析，在跨機器與跨作業系統移動時仍能保持連結有效。 |
| 絕對路徑 | `/Users/alice/projects/team-base/griptape-nodes-project.yml` | 寫死特定機器的絕對路徑；本機可用但無法跨環境遷移。 |

在 `parent_project_path` 中**不會**展開巨集語法。諸如 `{workspace_dir}`、`{project_dir}` 與 `{outputs}` 等標籤在此處不進行替換。巨集專屬於 `path_macro` 與 `situations.*.macro` 欄位。波浪號 `~` 與系統環境變數*則會*被展開 — 詳見[路徑值展開規範](#path-values-in-workspace_dir-libraries_dir-and-parent_project_path)。

### 繼承與刪除標記 (Inheritance and tombstones) {: #inheritance-and-tombstones }

父繼承鏈在合併前完成遞迴解析。孫專案 → 子專案 → 父專案 → 預設值的多層繼承鏈，其合成邏輯與工作區覆疊層套用至預設值完全相同：每一層的欄位皆會覆寫或擴充其下層內容。

若欲**剔除**繼承而來的項目而非覆寫它，請將數值設為 `null`：

```yaml
directories:
  scratch: null   # 徹底移除父專案中定義的 `scratch` 目錄
```

將 `parent_project_path: null` 會明確解除繼承關聯，使專案回退至以系統預設值為基底。完全省略該欄位則會繼承父專案的路徑設定（這通常不是您想要的 — 通常您會在每個子專案中明確宣告 `parent_project_path`）。

若專案的基底*本身即為*系統預設值，應完全省略 `parent_project_path` 欄位 — 省略本身即代表「以系統預設值為基底」，無需額外指向任何地方。僅在父級為*另一個獨立專案檔案*時才需配置。這在團隊共用時尤為重要：若在共用儲存上的預設衍生專案中寫入父路徑，容易引入無法在他人電腦上解析的本機私有路徑。

### 父專案隨同子專案自動載入 {: #parents-load-with-their-children }

您永遠無需單獨手動註冊父專案。載入子專案時會自動遞迴載入其繼承鏈中的所有專案，因此僅透過 `parent_project_path` 參照的父專案會自動被引擎識別，並並列呈現在專案清單中 — 即便該子專案是您唯一開啟或透過 `project_file` 指定的專案。

父專案僅是隨同載入，而非被永久*新增*：它們不會被強制寫入您的本機持久化專案清單中，因此開啟子專案不會永久累積歷史祖先專案。若您先前已手動註冊過該父專案，它會維持原有的登錄狀態。

此載入僅在子專案成功載入時生效。若子專案損毀無法讀取，或您無權開啟，其繼承鏈中的任何內容皆不會殘留在專案清單中。

### 循環依賴與父專案缺失 {: #cycles-and-missing-parents }

若專案的父繼承鏈中包含循環依賴，引擎會直接拒絕載入。無論是直接自我參照（A → A）還是間接循環（A → B → A），皆會被及時攔截並在當前子專案上回報為驗證錯誤。

無法讀取或解析的父專案亦會在子專案上回報為驗證錯誤。子專案絕不會靜默降級退回系統預設值 — 載入流程會明確中斷報錯，確保問題清晰可見。

### 跨機器共用專案 {: #sharing-across-machines }

在某台電腦上打包為 Zip 壓縮包並在另一台電腦解壓縮的工作區，只要父專案是透過相對路徑參照、且與子專案一同存放在工作區內部，其父子關聯即可完好保存。絕對路徑在跨電腦時會失效（家目錄不同、磁碟代號差異）；相對路徑是確保父繼承鏈可攜性的關鍵。

## `workspace_dir`、`libraries_dir` 與 `parent_project_path` 中的路徑值展開規範 {: #path-values-in-workspace_dir-libraries_dir-and-parent_project_path }

這三個欄位皆接受路徑輸入，並以完全相同的邏輯依序解析：

1. **`~` 與環境變數最先展開**。`~` 展開為您的家目錄，`${NAME}`（或 Windows 上的 `%NAME%`）會替換為對應環境變數的值。
1. **展開後若仍為相對路徑**，則相對於宣告該設定項的專案檔案所在目錄進行解析，確保路徑隨專案隨行攜帶。
1. 將最終路徑進行正規化 (Normalize)，確保同一個實體資料夾的不同拼寫能精確對齊。

展開操作*最先*執行是其能正確運作的核心：一旦 `STUDIO_LIBS` 完成賦值，`${STUDIO_LIBS}/griptape-libraries` 便直接成為絕對路徑，系統會將其視為絕對路徑處理，而非錯誤地附加至專案資料夾後方。

```yaml
libraries_dir: "${STUDIO_LIBS}/griptape-libraries"   # 一旦設定 STUDIO_LIBS 即為絕對路徑
workspace_dir: "~/Projects/marketing"                # 指向個人家目錄
```

**可用的變數範圍**：僅包含啟動引擎時作業系統傳入的環境變數 — 即啟動 Griptape Nodes 前由 shell、啟動腳本或工作室 Pipeline 導出的全域變數。專案內部的 `environment:` 區塊**刻意不在**此處生效：因為該區塊是在專案*啟動 (Activated)* 時才套用，而上述三個路徑欄位在此之前就必須先完成解析。若專案自身的環境變數能反向改變其工作區與程式庫路徑的意義，那麼路徑將取決於當時碰巧開啟了哪個專案，造成不確定性。

**`{macro}` 巨集標籤在此不展開**：`{workspace_dir}`、`{project_dir}`、`{outputs}` 及其他巨集標籤會被拒絕解析 — 引擎會回報該欄位無法解析，絕不會建立字面名為 `{outputs}` 的怪異資料夾。巨集專屬於 `path_macro` 與 `situations.*.macro` 欄位，它們依賴上述路徑先解析完畢後的執行期狀態。

**永遠可用的內建環境變數：`${GTN_DEFAULT_LIBRARIES_ROOT}`**：引擎在開機啟動時自動發布此變數，因此與上述 `${STUDIO_LIBS}` 不同，它在任何電腦上皆無需手動設定。它持有程式庫預設安裝的絕對路徑 — 即 `griptape-nodes init` 部署標準程式庫的相同目錄 — 這使得專案能以高度可攜的方式直接指向既有程式庫樹，避免下載重複套件：

```yaml
libraries_dir: "${GTN_DEFAULT_LIBRARIES_ROOT}/shared"
```

它不會循環追蹤專案自身的 `libraries_dir`。參閱[引擎發布的環境變數](../configuration.md#variables-the-engine-publishes)。

使用環境變數前值得留意的三件事：

- `%NAME%` 語法**僅在 Windows 上生效**。若需跨平臺相容，請一律使用 `${NAME}` 或跨平臺映射格式。
- 裸寫的 `$NAME`（無大括號）在此處**不被視為變數**。因為它無法與 `$Recycle.Bin` 等真實資料夾名稱區分，所以會保持字面原樣 — 請務必書寫為 `${NAME}`。
- 環境變數只有在每台電腦皆有定義時才能保障可攜性。相對路徑無需任何外在約定，因此若目標資料夾隨專案一同存放，請優先使用相對路徑。

### 無法解析的路徑將直接中斷報錯 {: #a-path-that-cannot-be-resolved-is-an-error }

完全**省略**這些欄位始終是安全的 — 專案單純退回至下一個來源（父專案設定、組態設定檔或全域預設值）。但若欄位*有填寫*卻無法解析出有效路徑，代表專案請求了當前電腦無法滿足的資源，引擎絕不會靜默改用其他目錄替代。**專案將載入失敗**，狀態標記為 [`UNUSABLE`](#validation-status)，驗證錯誤訊息會精確指明欄位、行號與具體肇因。任何將其列為父級的專案亦會連帶載入失敗，並在 `parent_project_path` 處報錯。

以下三種情況會觸發此錯誤：

- 當 `STUDIO_LIBS` 未設定時使用了 `${STUDIO_LIBS}/libraries` — 系統不會憑空建立字面名為 `${STUDIO_LIBS}` 的資料夾。
- 誤用了 `{outputs}/libraries` 或任何其他 `{macro}` 巨集標籤。
- 跨平臺映射中既未包含當前作業系統的鍵值，亦未提供 `default` 備援。

最後一項是極需預先防範的情境，而配置 `default` 便是解決之道。若映射中僅定義了 `windows`，代表未對 Linux 團隊成員的環境給予指示，因此在 Linux 電腦上誠實報錯載入失敗遠勝於憑空猜測：

```yaml
libraries_dir:
  windows: "D:/shared-libraries"      # 高速本機磁碟，僅限 Windows
  default: "./libraries"              # 其餘作業系統：存放於專案同級
```

這種報錯是刻意設計的警示機制，因為若不報錯後果更嚴重：共用專案若鎖定了 `${STUDIO_LIBS}`，每位未配置該變數的成員開啟時，將會在完全意料之外的未預期路徑靜默重複下載所有擴充套件。請設定該環境變數、補齊 `default`，或直接移除該欄位。

## 工作區目錄 (`workspace_dir`) {: #workspace-directory }

選填的 `workspace_dir` 欄位指定本專案所使用的[工作區 (Workspace)](workspace.md)實體目錄。一旦設定，它便是**最高優先權**的工作區來源 — 它會覆寫每位使用者的 `project_workspaces` 映射、`GTN_CONFIG_WORKSPACE_DIRECTORY` 環境變數、專案同級組態、父專案繼承以及全域預設值。

當您建立 **v1** 規格的專案時，建立介面會自動寫入 `workspace_dir: "./"`，達成獨立自足：工作區直接解析至專案自身的資料夾。這依然是一個普通的選填欄位 — 清空它（或設為 `null`）可回退至繼承父專案或全域預設值。**v0** 專案建立時不會預填 `workspace_dir`。（引擎的預設範本在任何版本中皆不填寫 `workspace_dir`；`./` 是在建立專案時寫入，而非繼承自預設範本）。

該值支援兩種形式：

```yaml
# 字串形式：絕對路徑，或相對於此專案 YAML 所在位置的相對路徑。
workspace_dir: "./workspace"

# 跨平臺映射形式：支援選填的 linux / darwin / windows / default 鍵值。
workspace_dir:
  darwin: "/Volumes/fast/ProjectA"
  windows: "D:/ProjectA"
  default: "./workspace"
```

相對路徑相對於*本*專案 YAML 檔案所在目錄解析 — 與 `parent_project_path` 規則一致 — 確保相對路徑隨專案跨機器與跨作業系統移動。`~` 與環境變數會自動展開（參閱[路徑值展開規範](#path-values-in-workspace_dir-libraries_dir-and-parent_project_path)）。原始值按字面儲存，僅在工作區解析當下才轉換為絕對路徑；儲存時絕不會被強制寫死為絕對路徑。目標目錄無需事先包含 `griptape_nodes_config.json`；空目錄完全合法。

對於跨平臺映射，引擎會挑選與當前平臺相符的項目。若當前平臺未定義則退回 `default`。若兩者皆未定義，專案在該平臺上將拒絕載入 — 參閱[無法解析的路徑將直接中斷報錯](#a-path-that-cannot-be-resolved-is-an-error)。否則解析後的路徑將作為最高優先權來源傳入[工作區路徑解析規則](workspace.md#workspace-resolution)。

與絕大多數欄位不同，`workspace_dir` **不會從父專案繼承**。它專屬於本專案自身的工作區，僅取自本專案的覆疊層 — 子專案不會自動採納父專案的 `workspace_dir`。（跨專案工作區繼承是由解析階梯的父專案回溯步驟單獨處理，而非透過範本合併模型）。設定 `workspace_dir: null` 或省略該欄位皆代表本專案未宣告專屬工作區。

## 節點程式庫目錄 (`libraries_dir`) {: #libraries-directory }

選填的 `libraries_dir` 欄位指定本專案安裝與解析節點程式庫的實體資料夾，使其與工作區徹底分離。這使得一組關聯專案能共用同一個程式庫存放位置 — 程式庫只需下載一次，所有指向該資料夾的專案皆可直接重複使用，無需重新下載。

該值支援兩種形式：

```yaml
# 字串形式：絕對路徑，或相對於此專案 YAML 所在位置的相對路徑。
libraries_dir: "./libraries"

# 跨平臺映射形式：支援選填的 linux / darwin / windows / default 鍵值。
libraries_dir:
  darwin: "/Volumes/fast/shared-libraries"
  windows: "D:/shared-libraries"
  default: "./libraries"
```

相對路徑相對於*本*專案 YAML 檔案所在目錄解析 — 與 `parent_project_path` 和 `workspace_dir` 規則一致 — 使相對路徑能隨專案跨機器無縫攜帶。`~` 與環境變數會自動展開（參閱[路徑值展開規範](#path-values-in-workspace_dir-libraries_dir-and-parent_project_path)）。原始值按字面儲存，僅在解析時轉換為絕對路徑。對於跨平臺映射，引擎挑選相符平臺，未相符則退回 `default`；若皆無相符則在該平臺上載入失敗。

### 繼承與解析規則 {: #inheritance-and-resolution }

`libraries_dir` 依以下順序解析，**優先順序由高至低**：

1. 專案**自身**宣告的 `libraries_dir` 欄位。
1. 沿著[父專案繼承鏈](#parent-projects)向上回溯，最近宣告了 `libraries_dir` 的祖先專案 — 相對於**該祖先專案**的實體目錄進行解析。這使得子專案能順利套用父專案的程式庫目錄樹。
1. 繼承鏈各處皆未宣告時：退回至工作區相對路徑下的 `libraries_directory` 組態值（預設為 `libraries`，即 `<workspace>/libraries`）。參閱[引擎組態配置](../configuration.md#separating-libraries-from-the-workspace)。

這是**與 `workspace_dir` 的核心差異**：`workspace_dir` 絕不從父專案繼承，但 `libraries_dir` **會**。未宣告自身 `libraries_dir` 的子專案會繼承最近祖先宣告的值（僅*明確宣告*的值可被繼承 — 留空的父專案不提供任何值，搜尋會繼續向上）。這也是建立介面在**頂層專案**（父級為系統預設值）中預先填入 `libraries_dir: "./libraries"`、但在子專案中留空的原因：頂層專案宣告了明確且可繼承的程式庫根目錄，其下所有子專案預設自動共用該目錄。子專案亦可自訂宣告專屬 `libraries_dir` 進行覆寫，或宣告 `libraries_dir: null` 來清除繼承並退回工作區相對預設值。

與 `workspace_dir` 相同，`libraries_dir` **不透過範本合併模型繼承**：它僅取自本專案覆疊層，子專案不會在[合併模型](#the-merge-model)中取得基底的值。跨專案繼承完全由上述解析順序（父鏈回溯）獨立掌控。

### 查詢實際生效的路徑 {: #seeing-where-the-paths-ended-up }

由於生效路徑可能來自上述三個地方的任何一處，您無需自行手動推算。`ListProjectTemplatesRequest` 介面會回報每個專案實際生效的 `workspace_dir` 與 `libraries_root` 絕對路徑，其解析結果與專案啟動時完全一致 — 包含父專案繼承，因此父專案與其子專案會呈現相同的程式庫根目錄。

從檔案載入的每個專案皆會回報這兩項路徑；只有無實體專案檔案的條目（系統預設值）或載入失敗的專案才會回傳 `null`。只要專案載入成功，解析結果絕不會為 `null`：因為無法解析路徑的專案根本無法載入。

## 專案驗證狀態 (Validation status) {: #validation-status }

專案檔案載入時會獲配以下四種狀態之一：

| 狀態名稱 | 具體意義 |
| ---------- | -------------------------------------------------------------------------------------- |
| `GOOD` | 載入成功且完全合法有效 |
| `FLAWED` | 載入成功但存在警告（例如綱要版本落後於當前主版號的最新版）— 仍可完全正常使用 |
| `UNUSABLE` | 存在致命錯誤，導致該範本無法被系統採用 |
| `MISSING` | 找不到指定的專案檔案 |

當專案狀態為 `UNUSABLE` 或 `MISSING` 時，Griptape Nodes 會自動退回套用系統預設值。您可以檢視具體的驗證問題清單（包含欄位路徑與錯誤行號）以快速排查修復。

## 系統原生預設值 {: #the-system-defaults }

系統預設值提供了開箱即用的情境規則與目錄結構。完整詳情請參閱[情境規則 (Situations)](situations.md)與[目錄架構 (Directories)](directories.md)。

## 綱要版本規範 (Schema versions) {: #schema-versions }

`project_template_schema_version` 遵循[語意化版本 (Semantic Versioning)](https://semver.org)規範；其**主版號 (Major)** 決定了該專案與哪套預設基準線進行合併。這使得預設規範在升級主版號時能進行重大調整，同時確保既有專案持續維持其建立時的原有行為：

- **v0 (`0.x`)** — 舊版基準線。檔案存放目的地相對於工作區根目錄，且檔案預設不依類型分流至子資料夾。
- **v1 (`1.x`)** — 當前基準線。檔案存放目的地相對於工作流程，輸出成品透過[副檔名目錄分流](file_extension_directories.md)歸檔，且新專案**預設獨立自足** — 建立介面會在全新的 v1 專案中寫入 [`workspace_dir`](#workspace-directory): `./`，使其工作區解析至專案自身資料夾（此為普通且可移除的覆寫項，未寫死在預設範本中）。全新的**頂層** v1 專案亦會獲配 [`libraries_dir`](#libraries-directory): `./libraries`，方便子專案繼承共用。新專案預設建立於 `1.0.0` 版本。

專案一律與**自身主版號**的基準線進行合併，因此開啟舊版主版號專案絕不會靜默打亂其既有檔案版面結構。

### 儲存與版號自動推進 {: #saving-and-version-roll-forward }

儲存專案會將其綱要版本推進至**其自身主版號內的最新版本**（此為向下相容的遞加變更 — 例如 `0.4.0` 專案存檔時會推進至 `0.5.2`）。存檔絕不會跨越主版號：v0 專案絕不會被靜默轉換為 v1，因為這會改變檔案的實體存放目的地。落後於其所屬主版號最新版的專案在載入時會標記 `FLAWED` 警告（但功能完全正常）。

### 跨主版號手動升級 {: #upgrading-across-a-major }

將專案升級至新的主版號是**完全明確且由使用者主動選擇的**（在 GUI 中點選[管理專案](gui_guide.md)中的「Upgrade schema」動作）。升級會將專案以新版主版號預設值為基準重新重置：僅保留您曾明確覆寫的值，其餘所有設定皆套用新版預設規範。這項變更屬於**重大破壞性變更** — 專案實際生效的工作區、程式庫目錄與檔案儲存路徑皆可能發生位移 — 這也是升級絕不自動執行的原因。
