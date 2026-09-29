# 透過 Worker 進行節點隔離 (Node Isolation with Workers) {: #node-isolation-with-workers }

本頁面為將程式庫以**隔離模式 (Isolated Mode)** 運行的操作指南：即在專屬的 Python 子程序中執行程式庫，確保您鎖定的重度依賴版本（如 `torch`、`transformers`、`diffusers`）絕不會與其他程式庫發生環境衝突。創作者可在編輯器中透過 **Shared / Isolated** 下拉選單進行模式切換（參閱 [程式庫指南](../../guides/libraries.md#shared-vs-isolated)）；在底層架構上，隔離的程式庫是由獨立的 **Worker 子程序** 承載執行的。本頁面面向程式庫作者闡述該機制的細節。有關攔截隔離錯誤的規則清單，請參閱 [嚴格模式參考 (Strict Mode Reference)](strict_mode.md)。

## 核心術語 (Vocabulary) {: #vocabulary }

- **編排程序 (Orchestrator)** —— Griptape Nodes 的主 Python 程序。它全權擁有拓撲流程圖、連線、參數註冊表、系統配置與敏感機密。編輯器 UI 直接與編排程序通訊。
- **Worker 子程序 (Worker Subprocess)** —— 專門執行您程式庫節點的獨立 Python 程序。每個啟用 Worker 模式的程式庫皆獲配專屬的子程序。Worker 透過 WebSocket 連線（即**事件匯流排 Bus**）與編排程序雙向通訊。
- **`process` 與 `aprocess`** —— 節點的核心運算執行方法。您只需如往常一樣實作 `process(self) -> ...`；框架會自動將其包裝為 `async def aprocess(self) -> None` 以便在 Worker 的事件迴圈中調度。嚴格模式規則所描述的「在 aprocess 內部」，在實務上即代表「在您編寫的 `process` 方法內部」。
- **架構探測 (Schema Probe)** —— 程式庫載入時執行的一次性掃描。Worker 會將每個註冊的節點類別實例化一次，以解析其參數佈局並回傳給編排程序。這意味著您的 `__init__` 會在接收任何執行請求前先行運作。

## 我是否應當啟用隔離模式？ {: #should-i-opt-in }

**您應當啟用隔離模式，若**您的程式庫鎖定了特定版本的重度機器學習套件（`torch`、`transformers`、`diffusers`、`accelerate`、`peft`、`controlnet-aux`、自訂 CUDA wheels），且您希望與鎖定不同版本的其他第三方程式庫和平共存。Worker 模式正是為解決跨程式庫的相依套件隔離而設計的專屬機制。

**您應當維持停用（共享模式），若**您的程式庫僅使用輕量、高度通用的依賴套件（Python 標準庫、`pydantic`、`griptape` 核心、常規 HTTP / YAML / JSON 工具庫）。在編排程序中直接運行可省去下述的跨行程序列化開銷。

若您難以抉擇，更安全的預設建議是**選擇啟用**——跨行程開銷雖然真實存在但極其微小，卻能一勞永逸地防範未來使用者安裝更沉重的程式庫時引發的版本衝突。

## 如何宣告啟用 Worker 模式 {: #how-to-opt-in }

Worker 託管行為是由 `griptape_nodes_library.json` 中 `metadata.declarations` 下的兩項宣告共同管轄的。它們並存於清單中，分別回答了兩個不同的問題：

- **`worker_mode_compatibility`** —— 該程式庫是否*具備*相容於 Worker 託管的能力。單一欄位 `compatibility`：
    - `COMPATIBLE`：該程式庫既可在編排主程序中運作，亦可在獨立 Worker 子程序中順暢執行。
    - `INCOMPATIBLE`：該程式庫僅能在編排主程序中執行，嚴禁置於 Worker 中託管。
- **`suggested_worker_mode`** —— 在無外部覆寫的情況下，該程式庫預設*以何種模式啟動*。單一欄位 `mode`：`ORCHESTRATOR` 或 `WORKER`。省略此宣告則採用引擎預設值（目前為編排程序）。編輯器為每個程式庫提供了 **Shared / Isolated** 下拉選單（Shared = 編排程序，Isolated = Worker），供使用者自由切換具備 `COMPATIBLE` 資格的程式庫；此宣告即為作者為該選單提供的官方建議預設值。

若省略這兩項宣告，等同於宣告 `worker_mode_compatibility` 為 `compatibility=COMPATIBLE` 且不指定 `suggested_worker_mode`——該程式庫具備 Worker 能力，但在使用者主動切換前預設於編排主程序中啟動。

若同時宣告 `worker_mode_compatibility` 為 `INCOMPATIBLE` 卻將 `suggested_worker_mode` 設為 `WORKER`，會構成語意矛盾；程式庫元數據驗證器會直接拒絕載入該清單。

```json
{
    "name": "My Library",
    "library_schema_version": "0.10.0",
    "metadata": {
        "author": "<Your Name>",
        "description": "<Description>",
        "library_version": "0.1.0",
        "engine_version": "0.85.0",
        "tags": ["AI", "Custom"],
        "declarations": [
            {
                "type": "worker_mode_compatibility",
                "compatibility": "COMPATIBLE"
            },
            {
                "type": "suggested_worker_mode",
                "mode": "WORKER"
            }
        ],
        "dependencies": {
            "pip_dependencies": [
                "torch==2.4.1",
                "transformers==4.45.2"
            ],
            "pip_install_flags": [
                "--extra-index-url",
                "https://download.pytorch.org/whl/cu121"
            ]
        }
    },
    "categories": [],
    "nodes": []
}
```

只有嚴格鎖定具體版本時，Worker 模式才能發揮精確的環境控制價值。模糊的 `torch>=2.0` 會被 pip 解析為任意版本，導致開發機與使用者端環境漂移。請一律鎖定如 `torch==2.4.1` 的精確版號。`pip_install_flags` 則是用於配置額外 index URL 等必要安裝參數的安全通道。

## 啟用隔離模式的代價與取捨 {: #what-you-give-up }

跨行程序列化網路開銷。當 Worker 端的節點在**節點執行期間**回頭存取編排程序所持有的全域狀態（流程圖、拓撲連線、參數註冊表、設定、機密）時，該請求必須經由 WebSocket 匯流排進行網路往返轉發。每次呼叫皆為一次網路往返，且回傳的資料快照是**調用當下的即時唯讀複本 (Stale-by-call)**——當 Worker 讀取到該結果時，編排程序的狀態可能早已推進變化。

兩項務必遵守的工程準則：

- **一律透過參數將資料傳入節點；嚴禁在運算期間主動反查流程圖狀態。** 雖然在 `process` 內部（或在輸入還原期間觸發的 `before_value_set` / `after_value_set` 中）查詢拓撲連線或相鄰節點狀態能夠被執行，但每次查詢皆伴隨一次跨行程網路往返，且結果在抵達 Worker 時即已過時。
- **請求機制是官方唯一的合法通道**，無論讀取或寫入皆然。請發布對應的正式請求（`SetParameterValueRequest`、`AddParameterToNodeRequest`、`RemoveParameterFromNodeRequest` 等），引擎會全自動妥善處理跨行程往返。

在節點執行**之外**發布的請求（例如程式庫載入或初始化引導期間）不會被轉發——因為此時 Worker 尚未與編排程序建立通訊連線。在 `__init__` 中調用匯流排會重入撞擊 Worker 自身的事件迴圈引發死結，這正是 `__init__` 設立專屬嚴格模式規則的原因。

## 傳遞不可序列化值 {: #passing-values-that-cannot-be-serialized }

運算管線、潛在張量或活躍的驅動器本質上無法轉為資料，因此無法直接作為參數數值在 Worker 與編排程序之間傳遞。只需將生產端輸出參數標記為 `serializable=False`，引擎便會將該實體物件保留在建構它的進程內部，僅跨行程傳輸輕量的引用鍵；消費端節點無需進行任何宣告，如常讀取該物件即可。完整運作機制與 GPU 記憶體釋放掛鉤請參閱 [傳遞不可序列化值 (Passing Values That Cannot Be Serialized)](passing_unserializable_values.md)。

## 必須掌握的生命週期變更 {: #lifecycle-changes-you-need-to-know }

### `__init__` 會在程式庫載入階段被提前執行 {: #init-runs-during-library-load }

Worker 子程序在啟動時會將每個已註冊的節點類別實例化一次，以向編排程序提取參數結構 (Schema)。由此衍生出三項關鍵規範：

- **嚴禁在 `__init__` 中執行阻塞性 I/O 操作。** 網路連線、憑證檢查、硬碟讀寫、資料庫連線皆會卡死程式庫的載入流程。架構探測具備有限的逾時閾值，凡在 `__init__` 中拋出例外或逾時的類別，將會被**直接自匯出的程式庫中靜默剔除**，且不會觸發任何常規規則警示。請將所有 I/O 移至 `process` 或建構後的生命週期掛鉤中。
- **嚴禁在 `__init__` 中發起事件匯流排請求。** 在架構探測期間重入匯流排會引發 Worker 死結。[`reentrant-bus-in-init`](strict_mode.md) 正確性規則會直接攔截並判定該類別違規，進而將其自程式庫中剔除。
- **在 `__init__` 中宣告參數是標準正規模式。** 在此處調用 `self.add_parameter(...)` 是完全合法的——架構探測正是節點宣告其靜態參數佈局的唯一正規時機。

### 每次 `ExecuteNodeRequest` 皆會建構全新的節點實例 {: #each-executenoderequest-constructs-a-fresh-node }

Worker 會根據請求元數據動態生成一個暫態 (Transient) 節點實例，執行其 `process` 方法，隨後立即將其銷毀。**您的節點在多次執行之間絕對不保留任何記憶體狀態。**

資料流動的標準官方模式：

- **輸入參數** 在每次執行開始時注入至 `self.parameter_values` 中，資料由編排程序所持有的真值自動還原而來。請在 `process` 內部讀取它們；絕不要假設上一輪運算的數值依然殘留。
- **輸出參數** 寫入 `self.parameter_output_values`。在 `process` 返回後，框架會將這些值打包發送回編排程序。
- **跨執行週期的持久化狀態** 必須委託編排程序保管。在 `process` 內部發布 `SetParameterValueRequest` 請求以更新真值；在下一輪執行時，新值將被重新還原注入至 `self.parameter_values` 中。切勿依賴在執行中途直接修改 `self.parameter_values[k] = v` 來傳遞狀態——此類本機修改絕不會向外傳播。

**絕對無效的做法**：直接指派實例屬性 `self.foo = ...` 並期望在下次執行時保留。下一次執行面對的是一個嶄新的節點實例。

### 運算執行期間修改參數清單不會自動傳播 {: #mutating-the-parameter-list-during-execute-does-not-propagate }

在 `process`（或 `aprocess`）內部調用 `self.add_parameter(...)` 或 `self.remove_parameter_element(...)`，僅作用於 Worker 本機當前的暫態節點。編排程序持有的權威節點對此變更毫無所知。

若必須在執行期間動態增刪參數，請一律透過請求匯流排路由：

- `AddParameterToNodeRequest` 用於新增參數
- `RemoveParameterFromNodeRequest` 用於移除參數

在 `process` 內部透過 `GriptapeNodes.handle_request(...)` 發送該請求。處理常式會將結構變更同步寫入編排程序。[`parameter-mutation-during-aprocess`](strict_mode.md) 規則會主動攔截直接在本機修改參數的違規行為。

**變更生效於編排程序的節點上，而非當前正在運算的節點實例。** 這意味著您無法在新增參數的該次執行中回頭讀取它：在 Worker 中調用 `self.get_parameter_by_name("new_param")` 將回傳 `None`，即便請求已成功且前端編輯器早已呈現該參數。

它在下一次執行時同樣不會自動現身。每次執行皆是自節點原始類別全新建構的，因此**參數結構**絕不會跨執行週期傳遞——僅有**數值**會傳遞。

這引出了核心約定：**節點的參數結構必須是其參數數值的確定性純函式 (Deterministic Function)。** 請在 `__init__` 中建立靜態參數，或在數值掛鉤中依據所指派的數值動態增減參數（例如 Diffusers VAE 解碼器在管線值設定時重新構建其輸出參數）。以此方式編寫的結構完全無需手動同步：相同的推導邏輯會在編輯時於編排程序的節點上執行，並在數值注入時於 Worker 的全新複本上再次執行，使各端自然收斂至相同佈局。

### 數值掛鉤 (Value Hooks) 的行為依程式庫型別而定 {: #value-hooks-it-depends-on-which-kind-of-library }

**宣告 `pip_dependencies_exec` 的執行依賴型程式庫** 在編排程序上保留了真實的節點類別，因此當使用者在 UI 編輯數值時，`before_value_set` 與 `after_value_set` 會如常在編排程序觸發。在編輯時響應使用者輸入而調整參數清單的掛鉤（如根據下拉選單顯示或隱藏欄位）皆能正常運作。

而在*運算執行*階段，這些掛鉤亦會在 Worker 內部數值還原注入時觸發。由於 Worker 的節點為全新建構，所有數值皆被視為新賦值，因此所有掛鉤皆會被執行一遍。請確保它們保持輕量且具備冪等性 (Idempotent)。

**傳統 Worker 模式程式庫** 表現則截然不同，因為編排程序僅持有您節點類別的 Stub 殘留副本（僅含參數結構，不含業務程式碼），因此使用者在 UI 編輯數值時絕不會調用您的覆寫邏輯。轉換傳入值依然有效，但在掛鉤中動態增刪參數會隨暫態節點一同被銷毀。對於這類程式庫，請務必在 `__init__` 中靜態定義完整的參數清單。覆寫數值掛鉤會在載入時觸發 [`value-hooks-execute-only-on-worker`](strict_mode.md) 警告。

### 連線掛鉤在隔離環境下永不觸發 {: #connection-hooks-never-fire-under-isolation }

`after_incoming_connection`、`after_outgoing_connection`、`allow_*` 校驗器及其對應的斷開事件是在連線變更時於編排程序上調用的——連線屬於編排程序管控的狀態，不會向 Worker 諮詢。在 Worker 託管的程式庫中，這些呼叫只會落在無邏輯的 Stub 類別上，因此您編寫的覆寫邏輯將會**被靜默忽略**。覆寫連線掛鉤會在載入時觸發 [`connection-hooks-inert-on-worker`](strict_mode.md) 警告。若您的節點必須動態響應連線拓撲，請將程式庫置於共享模式 (Shared Mode) 下運行。

### 參數轉換器、驗證器與特徵標籤不跨行程同步至編排程序 {: #parameter-converters-validators-and-traits-do-not-cross-to-the-orchestrator }

當架構探測匯出您的程式庫時，僅有 `Parameter` 的純量屬性（名稱、型別、預設值、提示資訊、允許模式）會被序列化至編排程序的 Stub 類別中。掛載於參數上的自訂 `converters`、`validators` 與 `traits` **不會**被跨行程傳遞——它們僅留存於 Worker 進程內部，且僅在 Worker 執行該節點時生效。

編排程序的 Stub 仍能接收使用者輸入並將數值轉發至 Worker，但前端 UI 無法在數值離開編輯器前預先執行這些自訂邏輯。此時會在載入時觸發 [`parameter-behaviors-dropped-in-schema`](strict_mode.md) 警告。

兩套實用因應方案：

- **將驗證或轉換邏輯移入 `process` 內部。** Worker 在真實運算前會重新執行查驗。代價是編輯器無法即時在行內提示驗證錯誤，創作者僅在執行時才會看到失敗。
- **將其視為純粹的 UI 潤色。** 若轉換器純粹是視覺呈現上的修飾（如首字母大寫），在編排端丟失不會影響核心運算。

## 系統配置、敏感機密與專案狀態的自動傳播 {: #configuration-secrets-and-the-current-project-propagate-automatically }

在配置或機密發生變更後，編排程序會自動向所有註冊的 Worker 廣播 `ReloadConfigRequest` 與 `RefreshSecretsRequest`。各個 Worker 會自動重新讀取磁碟上的共用檔案並更新記憶體快照。您完全無需編寫任何同步程式碼。

當前活動中的**專案 (Project)** 亦遵循完全相同的自動傳播機制。Worker 啟動時會自磁碟共用配置推導當前專案；隨後在編排程序中切換專案時，新專案會被即時推播至所有運作中的 Worker，確保環境變數、目錄巨集與情境巨集在兩端進程中解析出完全一致的路徑。

需注意的細節：由環境變數注入的作業系統原生金鑰（如容器層級的 `OPENAI_API_KEY`）在**重新整理廣播**與**顯式刪除**操作中皆會被嚴格保護，Worker 重新讀取 `.env` 檔案絕不會覆蓋這些全域環境變數。

## 程式庫隔離就緒自我檢核清單 {: #checklist }

- [ ] 在 `metadata.declarations` 中宣告了 `worker_mode_compatibility` 為 `compatibility: COMPATIBLE`（或完全省略該宣告，系統預設視為 COMPATIBLE），並搭配 `suggested_worker_mode` 為 `mode: WORKER`
- [ ] `__init__` 中無任何 I/O 操作，且未發布任何事件匯流排請求
- [ ] 嚴禁在 `process` 內部直接調用 `add_parameter` / `remove_parameter_element`；一律透過 `GriptapeNodes.handle_request(...)` 發送 `AddParameterToNodeRequest` / `RemoveParameterFromNodeRequest`
- [ ] 跨節點/流程圖狀態一律透過參數輸入端傳遞，切勿在 `process` 內部動態反查流程圖
- [ ] 未覆寫連線生命週期掛鉤（`after_incoming_connection` 等）——它們在 Worker 託管下永不觸發
- [ ] `before_value_set` / `after_value_set` 僅用於資料轉換，不用於編輯器即時響應或參數結構增刪
- [ ] 自訂 `converters` / `validators` / `traits` 已在 `process` 內部提供二次防禦，或接受其僅作為前端裝飾
- [ ] `pip_dependencies` 嚴格鎖定具體精確版本號
- [ ] 若安裝需要自訂索引 URL 或特定參數，已正確配置 `pip_install_flags`

## 嚴格模式 (Strict Mode) 安全防護網 {: #strict-mode-is-your-safety-net }

在開發時以本機引擎運行您的程式庫，並在主控台日誌中觀察節點執行時輸出的 `strict-mode` 提示。Worker 的日誌會輸出至啟動引擎的同一終端機中，並帶有 `Worker-<engine-id>` 前綴以便識別。請同時檢視 **WARNING** 與 **ERROR** 條目。

五項核心規則及其嚴重性級別：

| 規則標籤 | 編排主程序 | Worker 子程序 | 處置行為與意涵 |
| :--- | :--- | :--- | :--- |
| [`reentrant-bus-in-init`](strict_mode.md) | ERROR | ERROR | 正確性核心規則。該節點類別將被直接自程式庫中剔除。 |
| [`parameter-behaviors-dropped-in-schema`](strict_mode.md) | WARNING | WARNING | 當 `Parameter` 包含 Worker 無法序列化的轉換器/校驗器/特徵時觸發。不中斷載入。 |
| [`connection-hooks-inert-on-worker`](strict_mode.md) | WARNING | WARNING | 節點覆寫了連線生命週期掛鉤時觸發。掛鉤在編排端 Stub 上無法生效。不中斷載入。 |
| [`value-hooks-execute-only-on-worker`](strict_mode.md) | WARNING | WARNING | 節點覆寫了數值掛鉤時觸發。數值轉換仍生效，但編輯器即時結構異動無效。不中斷載入。 |
| [`parameter-mutation-during-aprocess`](strict_mode.md) | WARNING | ERROR | 在 Worker 運算期間直接修改參數清單時，會直接將節點運算結果判定為失敗。 |

若日誌中出現嚴格模式警示，規則提供的修復指引會明確指出違反了上述哪項準則及對應的修復方案。Worker 日誌完全無任何嚴格模式 WARNING 與 ERROR，即代表您的程式庫已完全達成「隔離就緒 (Isolation-ready)」。
