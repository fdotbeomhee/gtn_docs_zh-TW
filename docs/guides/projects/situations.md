# 情境規則 (Situations) {: #situations }

情境規則是具名的檔案儲存場景。它定義了：

- 檔案儲存至**何處**（透過巨集範本）
- 當目標位置已存在同名檔案時**如何處理**（透過衝突覆寫原則）
- 當儲存失敗時的**應變措施**（透過選填的備援情境規則）

當節點需要儲存檔案時，它會宣告自身所屬的情境名稱（例如 `save_node_output`），專案系統便會透過該情境的巨集解析出完整實體路徑並執行對應的防衝突原則。

## 衝突覆寫原則 (Collision policies) {: #collision-policies }

| 衝突原則 | 行為說明 |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `create_new` | 自動遞增檔名中的數字計數器，直到找到不衝突的全新檔名為止。巨集可包含 `{_index?:NN}`（選填 — 首次儲存不帶序號，衝突時追加）或 `{_index:NN}`（必填 — 首次儲存即帶序號）。若兩者皆未定義，系統會在衝突時自動於解析檔名後追加 `_1`、`_2` 等後綴。 |
| `overwrite` | 直接覆寫現有同名檔案，不彈出詢問提示。 |
| `fail` | 若檔案已存在，立即中止執行並拋出錯誤中斷流程。 |

`create_dirs` 欄位控制當目標中間父資料夾不存在時，是否自動予以遞迴建立（`true`，行為同 `mkdir -p`）；若設為 `false` 則在父資料夾缺失時報錯。

## 備援機制 (Fallbacks) {: #fallbacks }

情境規則可宣告一個備援情境 (Fallback situation)。若首選情境無法順利解析其巨集（例如缺少必要變數），系統會自動嘗試調用該備援規則。系統預設的 `save_file` 是多數情境共用的最基礎備援規則。

## 系統預設情境規則一覽 {: #default-situations }

### `save_file` {: #save_file }

```
macro:  {file_name_base}{_index?:03}.{file_extension}
policy: create_new, create_dirs: true
```

通用檔案儲存於專案根目錄（或調用端上下文指定的路徑）。這是多數情境的最終備援方案。`{_index?:03}` 變數具備 3 位數補零且為選填項 — 首次儲存省略，碰撞時依序產生 `001`、`002` 等。

### `copy_external_file` {: #copy_external_file }

```
macro:    {inputs}/{node_name?:_}{parameter_name?:_}{file_name_base}{_index?:03}.{file_extension}
policy:   create_new, create_dirs: true
fallback: save_file
```

當使用者將外部檔案複製或拖曳匯入專案時觸發。檔案歸檔於 `inputs` 目錄中。節點名稱與參數名稱作為選用前綴，方便追溯檔案來源。

**範例**：

```
node_name="LoadImage", parameter_name="source", file_name_base="photo", file_extension="jpg"
→ inputs/LoadImage_source_photo.jpg

node_name 未提供, file_name_base="photo", file_extension="jpg"
→ inputs/photo.jpg
```

### `download_url` {: #download_url }

```
macro:    {inputs}/{sanitized_url}
policy:   overwrite, create_dirs: true
fallback: save_file
```

當節點自網路 URL 下載檔案時調用。網址會自動轉譯為安全的合法檔名。自相同 URL 下載的檔案會直接覆寫而非重複建立複本。

### `save_node_output` {: #save_node_output }

```
macro:    {outputs}/{sub_dirs?:/}{node_name?:_}{file_name_base}{_index?:03}.{file_extension}
policy:   create_new, create_dirs: true
fallback: save_file
```

當節點生成並儲存運算成品時調用。檔案存入 `outputs` 目錄中。選用的子目錄標籤 (`{sub_dirs?:/}`) 支援在輸出資料夾內建立巢狀結構。節點名稱為選用前綴。

**範例**：

```
outputs="outputs", node_name="ImageGen", file_name_base="render", _index=1, file_extension="png"
→ outputs/ImageGen_render001.png

sub_dirs="lighting/pass_a", node_name="ImageGen", file_name_base="render", file_extension="exr"
→ outputs/lighting/pass_a/ImageGen_render.exr
```

### `save_output_directory` {: #save_output_directory }

```
macro:  {outputs}/{sub_dirs?:/}{dir_name}_v{###}
policy: create_new, create_dirs: true
```

當節點將輸出成品寫入整個資料夾而非單一檔案時調用。每次執行皆會建立新的編號資料夾，確保先前的運算結果永不被覆寫。序號會從磁碟上已存在的最高版本號向上遞增。

**範例**：

```
dir_name="renders"
→ outputs/renders_v001      (首次執行)
→ outputs/renders_v002      (第二次執行)
```

### `save_file_sequence` {: #save_file_sequence }

```
macro:  {outputs}/{file_extension_directory?:/}{sub_dirs?:/}{file_name_base}_v{###}/{file_name_base}.####.{file_extension}
policy: create_new, create_dirs: true
```

當節點輸出連號序號檔案（例如從影片中擷取出的影格序列）時調用。每次執行獲配專屬的版本資料夾，其內部的影格則單獨進行編號。

兩組 `#` 標記分工不同：大括號內的 `{###}` 代表資料夾的版本號；外側裸寫的 `####` 則是每張影格填入序號的位置。單一巨集中僅允許一個 `{###}`，因此影格序號請使用裸寫形式。影格編號細節請參閱[圖像序列 (Sequences)](sequences.md)。

**範例**：

```
file_name_base="frames", file_extension="png"
→ outputs/images/frames_v001/frames.0001.png
→ outputs/images/frames_v001/frames.0002.png
→ outputs/images/frames_v002/frames.0001.png   (第二次執行)
```

### `save_preview` {: #save_preview }

```
macro:    {previews}/{drive_volume_mount?:/}{source_relative_path?:/}{source_file_name}.{preview_format}
policy:   overwrite, create_dirs: true
fallback: save_file
```

用於生成預覽縮圖。預覽檔案會完全鏡像對齊來源檔案的目錄層級結構，確保每個來源檔案對應單一預覽檔。預覽檔案採覆寫策略而非版本化。`previews` 目錄預設為 `.griptape-nodes-previews` 隱藏資料夾。

### `save_static_file` {: #save_static_file }

```
macro:    {workflow_dir?:/}{static_files_dir}/{file_name_base}.{file_extension}
policy:   overwrite, create_dirs: true
fallback: save_file
```

由靜態檔案管理器用於儲存靜態資產。檔案落入當前工作流程目錄下的 `static_files_dir` 子目錄中。重新生成時採直接覆寫。

### `save_temp_file` {: #save_temp_file }

```
macro:    {temp}/{node_name?:_}{file_name_base}{_index?:03}.{file_extension}
policy:   overwrite, create_dirs: true
fallback: save_file
```

當節點在運算處理期間需要寫入中繼過渡或草稿檔案時調用（例如在色彩空間轉換步驟之間產生的臨時 EXR）。檔案存放於 `temp` 目錄中，節點使用完畢後應主動予以刪除。

### `save_workflow` {: #save_workflow }

```
macro:    {workspace_dir}/{sub_dirs?:/}{file_name_base}.{file_extension}
policy:   overwrite, create_dirs: true
fallback: save_file
```

每次儲存工作流程檔案時調用：當您點擊手動儲存，或是編輯器為您建立工作流程複本（建立分支或自範本複製）時皆會觸發。所有流程儲存皆透過此情境派發，因此修改 `save_workflow` 會全面改變所有流程檔案的存檔落點。

工作流程檔案存入工作區根目錄，並透過選用的 `{sub_dirs?:/}` 前綴維持子目錄層級。儲存工作流程採原位覆寫現有檔案而非版本化；若需生成連號歷史版本，請參閱下方的 [`create_versioned_workflow`](#create_versioned_workflow)。

**範例**：

```
workspace_dir="/projects/demo", file_name_base="my_workflow", file_extension="py"
→ /projects/demo/my_workflow.py

sub_dirs="archived", file_name_base="my_workflow", file_extension="py"
→ /projects/demo/archived/my_workflow.py
```

### `create_versioned_workflow` {: #create_versioned_workflow }

```
macro:    {workspace_dir}/{sub_dirs?:/}{file_name_base}_v{_index:03}.{file_extension}
policy:   create_new, create_dirs: true
fallback: save_file
```

當工作流程以版本化儲存意圖存檔時調用。每次儲存皆會生成帶有下一個補零序號的全新檔案 — `my_workflow_v001.py`、`my_workflow_v002.py` 等 — 方便創作者隨時保留歷史快照而不覆寫先前成果。

版本號遞增是由**巨集反向驅動**的：當觸發版本化儲存時，引擎會將前一次儲存的路徑與本情境巨集進行反向比對，提取出範本定義的所有變數（包含補零插槽）。下一次儲存時直接複用這些變數，並透過衝突偵測將補零序號遞增超越既有檔案。由於版本後綴沒有任何硬編碼邏輯，自訂該巨集（例如將 `_v{_index:03}` 改為 `.{_index:04}`）依然能順暢運作。

此情境在 API 層級由 `SaveWorkflowRequest` 傳入 `create_versioned=True` 觸發；在 UI 介面中則呈現為獨立選單項目（如「另存為新版本」）。

## 節點如何調用情境規則 {: #how-nodes-use-situations }

節點所屬的情境規則是由編寫該節點的工程師決定的，而非由創作者在介面中指定。執行檔案儲存的節點在底層採用 `ProjectFileParameter`，情境名稱在建構該參數時便已宣告寫死。**節點外觀表面上沒有任何情境欄位**。節點僅呈現檔名參數（通常命名為 **Output File**），情境規則在幕後默默運作。

檔名參數*本質上即為* `ProjectFileParameter`。您在該處輸入的字串會轉換為 `file_name_base` 與 `file_extension` 變數，由情境巨集統一決定檔案實際寫入哪裡。因此在調用 `save_node_output` 的節點中輸入 `render.png` 會生成 `outputs/MyNode_render.png`，而不是落入專案根目錄的 `render.png`。節點僅提供檔名主體，其餘所有要素（目錄實體路徑、系統內建變數）皆由專案系統補齊。

絕大多數生成與儲存節點皆調用 `save_node_output`。其餘情境則由相應的系統組件專屬調用：拖曳匯入調用 `copy_external_file`、網址下載調用 `download_url`、縮圖調用 `save_preview`，儲存工作流程則調用 `save_workflow`。

### 將單一節點的輸出重新導向至其他位置 {: #sending-one-nodes-output-somewhere-else-entirely }

輸入諸如 `render.png` 的純檔名可交由情境規則指派目的地，這也是最常見的預期行為。若針對特定節點需要特殊處置，您可輸入另外兩種路徑型態進行覆寫：

- **相對資料夾路徑** — 輸入 `lighting/pass_a/render.png` 會將檔案巢狀存放於情境目錄內部，因此 `save_node_output` 會將其儲存於 `outputs/lighting/pass_a/`。起點依然由情境規則掌控。
- **絕對路徑** — 輸入 `/mnt/studio/renders/render.png` 或 `C:\renders\render.png`。此時您直接指定了磁碟上的實體絕對位置，情境巨集將不再負責建構路徑，檔案會完全落於您指定之處：不再附加 `outputs` 資料夾、不帶節點名稱前綴、不追加版本序號。但情境規則依然掌控同名衝突時的策略 — 在 `save_node_output` 下會自動產生 `render_1.png` 避免覆寫歷史成品 — 且仍會自動遞迴建立路徑中尚不存在的中間資料夾。

網路網址 (URL) 不屬於合法的儲存目的地。節點無法直接存檔至 `https://example.com/render.png`，輸入網址會直接報錯中斷。

由於絕對路徑完全繞過了專案系統，它亦無法享有專案系統帶來的可攜性：絕對路徑與您當前電腦環境高度綁定，將工作流程移交給他人時將因找不到該實體路徑而報錯。當您希望**所有**節點皆寫入全新位置時，請至專案設定檔中自訂情境規則，而非逐一手動輸入絕對路徑。

### 查詢節點所屬的情境規則 {: #finding-out-which-situation-a-node-uses }

將滑鼠懸停於節點的檔名參數上方。工具提示 (Tooltip) 會明確指明其調用的情境名稱，例如：

```
Output filename (uses 'save_node_output' situation template)
```

在自訂節點的原始碼中，此名稱對應傳遞給 `ProjectFileParameter` 的 `situation=` 引數。

### 針對單一節點單獨覆寫情境規則 {: #overriding-a-situation-on-one-node }

若要在不修改專案設定檔的前提下變更單一節點的儲存規則，點擊該節點檔名參數旁的**齒輪圖示 (Cog button)**。這會在畫布上建立一個 **File Output Settings** 節點並自動連線至該參數，其內部已預先填入該節點原有的情境與檔名設定。

File Output Settings 節點將隱藏的情境設定完全展開於畫布上，每項參數皆可獨立調整：

- **Situation**：挑選任何情境，包含專案檔中定義的自訂情境。變更後會自動重新載入其下方的巨集與防衝突原則。
- **Macro**：路徑範本，可針對本次連線單獨編輯。
- **If File Exists**：衝突覆寫原則（Increment Version 遞增版本、Overwrite Existing 覆寫現有，或 Abort / Error 中止報錯）。
- **Auto Create Path**：是否自動遞迴建立缺失的中間父資料夾。

在此處配置的任何規則僅對該連線節點單獨生效。若欲變更全域所有節點的寫入規則，請至專案設定檔中編輯情境設定。

## 新增自訂情境規則 {: #adding-custom-situations }

實戰範例請參閱[實用自訂指引 (Customization Guide)](customization.md)。

在專案設定檔中重新定義預設情境（例如 `save_node_output`）會直接改變所有使用該情境的節點目的地，完全無需逐一修改畫布上的節點。而新增*全新*的情境規則，唯有在節點主動指派它時才會生效：包含自訂節點在程式碼中傳入 `situation=`，或是透過畫布上的 **File Output Settings** 節點手動選取它。
