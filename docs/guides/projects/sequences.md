# 圖像與檔案序列 (Sequences) {: #sequences }

Griptape Nodes 能將帶有編號的目錄序列檔案辨識並讀取為**序列 (Sequences)** — 例如彩現輸出序列 `render.0001.exr`, `render.0002.exr`, … `render.0100.exr`；亦包含對話錄音條目 (`take_##.wav`)、文字切片區塊 (`chapter_###.md`)，或任何檔名中包含數字鍵、將項目歸類為循序集合的情境。您只需將支援序列的節點指向*路徑或樣式模式*（帶有數字預留位置的檔名，或直接指向字面檔案路徑），引擎便會自動搜尋磁碟上相符的檔案、妥善處理缺格斷層，並傳回帶有整數序號與固定補零字串格式的完整條目清單。

本頁詳解路徑/樣式模式語法、缺漏項目處理原則，以及在編寫工作流程前應掌握的各項規範。

## 樣式模式語法 (Pattern syntax) {: #pattern-syntax }

序列樣式模式的本質為：在檔名中以專用語彙標記 (Token) 取代具體的序號數字。系統完整支援四種標記格式：

| 語彙標記 | 位元寬度 | 說明備註 |
| ------ | ----- | ------------------------------------------------------------- |
| `####` | 4 位數 | 每個 `#` 代表一個數字。`##` = 2 位數，`####` = 4 位數等。 |
| `%04d` | 4 位數 | C 語言風格的 printf 語法。`%04d` = 4 位數補零。 |
| `@@@@` | 4 位數 | Houdini / RV 風格語法。意義與 `####` 完全相同。 |
| `$F4` | 4 位數 | Houdini 變數語法。意義與 `####` 完全相同。 |

四種格式在功能上完全等價；您可依團隊管線慣例自由挑選。對於新專案範本，我們建議採用 `####` 或 `%04d` — 它們在業界各類 DCC 創作工具中的識別度最高。

典型範例：

```
render.####.exr             項目 5  → render.0005.exr
render.%04d.png             項目 12 → render.0012.png
take_##.wav                 項目 7  → take_07.wav
```

語彙標記一律僅能存在於**檔案名稱**中。不支援在目錄名稱中嵌入標記（例如 `render/####/beauty.exr`）— 請務必將序號維持在檔名主體部分。

## 路徑未包含序列標記時的行為 {: #when-the-path-has-no-sequence-token }

若傳入的路徑不含任何序列標記（例如 `/work/photo.png`、`{inputs}/poster.png`、`render.0002.png`），在語意上存在*歧義性*。創作者的原意可能是：

1. **指向單一具體檔案的字面名稱** — 我只想載入 `render.0002.png` 這個單一檔案；
2. **隱式序列的其中一張影格** — 期望系統自動探測數字，將同目錄下所有符合 `render.NNNN.png` 的檔案組合為單一序列；
3. **單純疏忽遺漏** — 創作者忘記輸入 `####`，此時最佳應對是明確報錯以便及時修正。

引擎透過 `NoTokenBehavior`（定義於 `griptape_nodes.common.sequences`）向調用端提供三種解讀策略：

| 列舉數值 | 具體解析行為 |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `SINGLE_FILE` *(預設)* | 將整個檔名視為純字面常數。若檔案存在則回傳單一項目的序列 (`first=last=1, padding=0`)；檔案不存在則回傳空結果。同目錄下的其他相鄰檔案會被忽略 — 即便存在 `0001..0005`，輸入 `render.0002.png` 亦僅傳回 `render.0002.png`。 |
| `EXPLORE_SEQUENCE` | 允許 fileseq 將檔名中的數字解讀為隱式序列標記。`render.0002.png` 會被視為推斷出的 `render.####.png` 序列的其中一格；掃描器會自動遍歷所有同前綴檔案。當上游工具僅輸出單一檔名、但您需要整批序列時極為實用。 |
| `REJECT` | 立即失敗並回報 `INVALID_TEMPLATE`，提示創作者補上序列標記。適用於絕不允許系統私自揣摩意圖的嚴格管線。 |

預設採用 `SINGLE_FILE` 能精確吻合「我挑選了單一檔案」的直覺預期。需要隱式分組行為的工作流程可主動進行切換。

## 與巨集語法協同運作 {: #combining-with-macros }

序列路徑可與專案的[巨集語法 (Macros)](macros.md)完美融合。巨集標頭由引擎內部預先解析以讀取正確的磁碟資料夾，但傳回的各路徑仍完整保留您傳入的巨集原始形狀：

```
輸入 ： {inputs}/shot_a/render.####.exr
輸出 ： Sequence(directory="{inputs}/shot_a",
                 entries=[{path: "{inputs}/shot_a/render.0001.exr"},
                          {path: "{inputs}/shot_a/render.0002.exr"}, ...])
```

這保障了掃描結果的可攜性。在某台電腦上 `{inputs}` 解析為 `/Volumes/Renders`，所生成的序列路徑標頭依然寫作 `{inputs}` — 將此工作流程移至 `{inputs}` 解析為 `C:\renders` 的另一台電腦上依然能順暢運作，因為下游節點皆會依據自身當前專案動態重新解析巨集。

普通的絕對路徑（無 `{...}` 區塊）則按原樣忠實回傳 — 輸入 `/work/render.####.png`，即傳回 `/work/render.0001.png`。

**相對路徑**：不帶前導斜線 `/` 且不帶巨集標頭的路徑（例如 `shot_a/render.####.png`）會相對於專案的工作區目錄進行解析。引擎在列出目錄前會自動補齊工作區路徑，因此 `shot_a/render.####.png` 與 `{workspace_dir}/shot_a/render.####.png` 解析結果完全相同。

大括號內部的巨集變數 `{...}` 與序列語彙標記完全相互獨立 — 兩者語法不互通，且在不同的階段由底層分別解析。

## 位元寬度嚴格相符原則 {: #width-matching-is-strict }

`#` 字元的數量（或 `%0Nd` 的寬度宣告）代表所要比對的**精確位元數**。若樣式宣告為 `####`，引擎僅會比對剛好 4 位數的檔案 — `render.0001.exr` 符合條件，但 `render.001.exr`（3 位數）或 `render.12345.exr`（5 位數）皆會被排除。

這與 Nuke 的行業標準完全對齊。若序列序號會溢出宣告寬度（例如 4 位數樣式但實際序號超過 9999），請使用更寬的樣式（如 `#####`）進行擷取。

若目錄中包含不同補零寬度的混雜檔案（例如同時存在 `render.0001.png` 與 `render.001.png`），它們會被視為**彼此獨立的不同序列**。引擎僅會挑選與您宣告的範本補零完全吻合的序列，其餘項目將被靜默略過。

## 缺漏項目處理原則 (Missing-item policies) {: #missing-item-policies }

實際的序列往往存在斷層缺失 — 算圖在第 47 影格崩潰、稀疏匯出僅儲存偶數序號、章節尚未編寫完畢等。掃描序列時，您可指定一套**處理原則 (Policy)** 來應對這些斷層：

| 處理原則 | 具體處置方式 |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ABORT` | 快速報錯中斷。在 `[first, last]` 區間內遇到第一個缺格時立即回報失敗並指出缺失號碼。不傳回任何 Sequence 物件。 |
| `SPLIT` *(預設)* | 依**連續區段**將序列切分為多個獨立序列。例如包含 1–5、8–12、15 序號的目錄會拆分為三個各自獨立的 Sequence 物件。 |
| `SKIP` | 傳回單一序列，僅包含實際存在的項目。缺格項目從輸出清單中剔除（但可透過序列的 `missing_numbers` 集合進行診斷查詢）。 |
| `FILL_NEAREST` | 傳回涵蓋完整 `[first, last]` 連續區間的單一序列。每個缺格項目自動填充離它最近的**較早**存在項目的路徑（若無更早項目則取最近的較晚項目）。 |

當缺格屬於嚴重異常時請選用 `ABORT`（例如算圖崩潰絕不應靜默推進）。當您希望*保留*斷層結構時請選用 `SPLIT`（每個連續片段各自具備獨立意義）。若需要稀疏序列請選用 `SKIP`；若需要強制對齊長度的緻密序列請選用 `FILL_NEAREST`。

**特定業務領域的缺格彩現屬於節點職責，而非引擎底層。** 若您希望用黑畫面影格、洋紅/黃色西洋棋棋盤格、靜音音訊片段或空白文字區塊來填充缺失項目，請使用 `SKIP` 模式掃描，並在自訂節點程式碼中巡訪 `missing_numbers` 進行自訂彩現。引擎專注於判定「磁碟上是否存在該號碼」，後續業務完全交由節點掌控。

## 子集範圍擷取 (Subset clipping) {: #subset-clipping }

支援序列的節點皆接受選填的 `start` 與 `end` 邊界參數。配置後，掃描結果會精確限制於該區間內：

- 低於 `start` 與高於 `end` 的項目會從輸出中剔除。
- 磁碟上的原始全量區間依然會透過 `discovered_first` / `discovered_last` 忠實記錄，方便您查詢裁剪前的磁碟真實狀況。
- 若指定的子集邊界完全落在磁碟發現範圍之外，將傳回空結果（失敗）。

## 序列資料模型結構 {: #what-you-get-back }

`Sequence` 採用 Pydantic 模型定義 — 可直接透過屬性讀取各欄位（如 `seq.first`, `seq.entries[0].number`）。自訂節點若需要處理序列，應將輸入參數宣告為 `type="Sequence"`。

每個 `Sequence` 包含以下詮釋資訊：

- **`first`** / **`last`** — 活躍區間（套用子集裁剪後的起點與終點）。
- **`discovered_first`** / **`discovered_last`** — 磁碟上實際發現的原始序號邊界，不受子集裁剪影響。
- **`padding`** — 宣告的補零寬度（例如 `####` 為 4）。
- **`pattern`** — 正規化後的規範樣式字串（例如 `render.####.exr`）。
- **`directory`** — 您傳入路徑中的目錄部分，保持相同形狀（傳入巨集則維持巨集，傳入絕對路徑則為絕對路徑）。
- **`policy`** — 所採用的缺格處理原則。
- **`entries`** — 活躍區間內每筆項目的 `SequenceEntry` 清單。每筆條目包含：
    - `number` — 整數序號值（例如 5）。
    - `padded_number` — 補零字串格式（例如 `0005`）。
    - `path` — 檔案實體路徑字串，形狀與傳入一致。傳入巨集標頭則完好保留（`{inputs}/render.0005.exr`）；傳入絕對路徑則原樣返回。在 `FILL_NEAREST` 原則下，缺格條目會填充相鄰項目的路徑；可透過比對 `entry.number in seq.present_numbers` 區分真實檔案與填充項。
- **`present_numbers`** — 在 `[first, last]` 區間內實際存在於磁碟上的序號集合。
- **`missing_numbers`** — 衍生計算出的缺漏號碼集合：活躍區間內磁碟上不存在的序號（適用於任何原則下的問題排查）。

## 刻意不支援的邊界案例 {: #cases-that-are-intentionally-not-supported }

以下案例刻意予以排除：

- **負數序號**：諸如 `render.-0005.exr` 的檔案在掃描時會被過濾忽略，剔除數量會記錄於序列狀態中。
- **目錄組件中的序列標記**：類似 `render/####/beauty.exr` 的樣式不支援比對。請將數字標記放置於檔名中。
- **多標記樣式**：包含兩個以上序列標記的範本（如 `v##_f####.exr`、`render.##.##.exr`）會在掃描時直接報錯拒絕。每個樣式僅允許單一標記。
- **時間碼 (Time codes)**：暫不支援。

## 技術淵源與架構底層 {: #where-this-comes-from }

序列底層建構於 [`fileseq`](https://github.com/justinfx/fileseq) — 視覺特效 (VFX) 產業事實上的標準影格範圍解析 Python 函式庫。我們將其作為語法解析與數字運算核心；而所有檔案系統的實際遍歷與讀取依然統一經過引擎的請求匯流排 (Request Bus)，因此工作區權限、路徑正規化以及 Windows 萬國碼長路徑支援皆能嚴密生效。fileseq 內部通篇使用 "frame" 術語，在對外公開的 API 中我們統一抽象為 "items" 與 "numbers"，使相同的程式碼能通用於任何包含序號的檔案類型，而不僅限於影格圖像。

## 公開 API 進入點：`ScanSequencesRequest` {: #public-entry-point-scansequencesrequest }

掃描操作透過引擎的事件匯流排派發，而非直接匯入函式。建構 `ScanSequencesRequest`（定義於 `griptape_nodes.retained_mode.events.os_events`）並調用 `await GriptapeNodes.ahandle_request(...)` 即可；事件處理常式會自動解析專案巨集、執行目錄列舉，並於背景 Worker 執行緒中運行 fileseq 解析，確保掃描龐大深層目錄時絕不卡頓主事件循環。

請求接受單一 `path` 欄位。呼叫範例：

```python
# 巨集樣式模式。輸出的每條路徑皆會保留 {inputs} 標頭。
ScanSequencesRequest(path="{inputs}/shot_a/render.####.exr")

# 普通絕對路徑樣式。原樣返回。
ScanSequencesRequest(path="/work/render.####.png", policy=MissingItemPolicy.SKIP)

# 無標記路徑。預設 no_token_behavior=SINGLE_FILE 會返回包含該檔案的單項目序列。
ScanSequencesRequest(path="/work/photo.png")

# 無標記路徑，但將其視為隱式序列的其中一格並遍歷相鄰檔案。
ScanSequencesRequest(
    path="/work/render.0002.png",
    no_token_behavior=NoTokenBehavior.EXPLORE_SEQUENCE,
)

# 嚴格模式：若路徑無標記則拋出 INVALID_TEMPLATE 錯誤。
ScanSequencesRequest(
    path="/work/render.0002.png",
    no_token_behavior=NoTokenBehavior.REJECT,
)

# 指定活躍區間子集。
ScanSequencesRequest(path="{inputs}/render.####.exr", start_number=10, end_number=50)
```

成功時回傳 `ScanSequencesResultSuccess`，承載：

- **`sequences: list[Sequence]`** — 套用處理原則後推斷出的序列物件清單。
- **`has_entries: bool`** — 當至少有一個 Sequence 且至少包含一筆條目時為 true。掃描正常完成但未找到任何項目會回傳帶有 `has_entries=False` 的成功狀態，而非報錯失敗。
- **`directory_had_matching_files: bool`** — 目錄中是否存在至少一個主檔名與副檔名相符的檔案。與 `has_entries` 搭配使用能精準判定掃描為空的原因：此項為 false 代表路徑錯誤或檔名完全不符；此項為 true 但 `has_entries=False` 則代表檔案存在但補零位元數不符或被子集範圍完全裁剪。
- **`discovered_first: int | None`** / **`discovered_last: int | None`** — 套用子集裁剪前，磁碟上推斷出的原始序號邊界。

失敗時回傳 `ScanSequencesResultFailure`，其 `failure_reason` 列舉包括 `INVALID_TEMPLATE`、`INVALID_BOUNDS`、`ABORTED_AT_GAP` 或底層作業系統的 `FileIOFailureReason`。

### 節點層級控制項 {: #node-level-controls }

標準節點庫中的 `ScanSequenceNode` 與 `ScanSplitSequenceNode` 皆對外提供以下參數：

- `fail_on_empty_result: bool = True`：預設為 true，當掃描結果為空時將節點引導至 Failure 控制流程輸出埠並附帶診斷訊息。設為 false 則允許空結果成功通過。
- **When there's no sequence marker (無序列標記時的處置)** 下拉選單：對應引擎的 `NoTokenBehavior` 列舉：
    - *Treat as a single file*（視為單一檔案，預設）
    - *Treat as part of a sequence*（視為序列的一部分）
    - *Fail unless a token is present*（除非存在標記否則報錯）
