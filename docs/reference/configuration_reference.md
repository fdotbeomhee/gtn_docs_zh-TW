<!-- GENERATED FILE - DO NOT EDIT BY HAND.
     Regenerate with `make docs/settings-reference` after changing the Settings model. -->

# 組態設定參考手冊 (Configuration Reference) {: #configuration-reference }

Griptape Nodes 引擎的所有設定項目，依分類進行分組。每個設定項目均可放置於任何 `griptape_nodes_config.json` 檔案中（載入順序請參閱 [引擎組態設定 (Engine Configuration)](../guides/configuration.md)）。巢狀設定項目以其完整的點分隔鍵 (Dotted key) 列出，這也是 `griptape-nodes config show` 所接受的格式，且下方的環境變數名稱亦是由此構建。組態檔案將該鍵的各個部分巢狀表示為 JSON 物件，因此 `worker.heartbeat_timeout_s` 在 JSON 中為 `{"worker": {"heartbeat_timeout_s": 30}}`；扁平的 `"worker.heartbeat_timeout_s"` 鍵會被忽略。帶有 `GTN_CONFIG_*` 環境變數的設定（包含在點分隔鍵各部分之間使用 `__` 的 `GTN_CONFIG_<PATH>` 形式，以及用於映射值項目設定的 `GTN_CONFIG_<NAME>__<KEY>` 形式），亦可透過環境變數進行覆寫；清單陣列值的設定則必須在組態檔案中編輯。映射的鍵名稱進行大小寫區分匹配，但整個變數名稱會轉為小寫，因此僅有原先即為全小寫的鍵可透過環境變數存取（詳情請參閱指南）。

## 檔案系統 (File System) {: #file-system }

應用程式的目錄與檔案路徑

| 設定項目 (Setting) | 型別 (Type) | 預設值 (Default) | 環境變數 (Environment variable) | 說明 (Description) |
| :--- | :--- | :--- | :--- | :--- |
| `workspace_directory` | string | `<current_working_directory>/GriptapeNodes` | `GTN_CONFIG_WORKSPACE_DIRECTORY` | 專案、工作流程與產生成果的根目錄。預設為當前工作目錄下的 GriptapeNodes 資料夾。其他檔案系統路徑（`libraries_directory`、`static_files_directory`、`sandbox_library_directory`、`synced_workflows_directory`）除非設定為絕對路徑，否則皆相對於此目錄進行解析。 |
| `static_files_directory` | string | `"staticfiles"` | `GTN_CONFIG_STATIC_FILES_DIRECTORY` | 靜態檔案目錄路徑，相對於工作區目錄。 |
| `sandbox_library_directory` | string | `"sandbox_library"` | `GTN_CONFIG_SANDBOX_LIBRARY_DIRECTORY` | 沙盒程式庫目錄路徑（在開發節點時非常有用）。相對路徑相對於工作區目錄解析。絕對路徑原樣使用。 |
| `libraries_directory` | string | `"libraries"` | `GTN_CONFIG_LIBRARIES_DIRECTORY` | 下載程式庫的存放目錄路徑。啟動時會遞迴自動探索所有搜尋到的 `griptape_nodes_library.json` 檔案。相對路徑相對於工作區目錄解析。絕對路徑原樣使用。專案可透過專案範本的 `libraries_dir` 欄位覆寫此位置（沿父專案鏈向下繼承），該欄位優先於此數值，以使子專案可共享父專案的程式庫安裝位置。 |
| `ffmpeg_directory` | string | `""` | `GTN_CONFIG_FFMPEG_DIRECTORY` | 引擎於初次使用時下載存放 ffmpeg/ffprobe 二進位執行檔的目錄絕對路徑。與其他目錄設定不同，此設定絕不相對於工作區解析：ffmpeg 快取屬於整台電腦而非單一工作區，因此跨所有工作區與專案共享。相對路徑會被忽略並輸出警告。留空（預設值）代表 `<XDG_DATA_HOME>/griptape_nodes/ffmpeg`。若要提供自訂執行檔而非下載，請將此路徑指向包含 `bin/<platform>/` 的目錄，該目錄需含有 ffmpeg、ffprobe 與一個空的 `installed.crumb` 檔案——static-ffmpeg 將該標記視為安裝完成的證明，每當遺失該標記時便會重新下載並覆寫執行檔。 |
| `synced_workflows_directory` | string | `"synced_workflows"` | `GTN_CONFIG_SYNCED_WORKFLOWS_DIRECTORY` | 同步工作流程目錄路徑，相對於工作區目錄。 |
| `enable_workspace_file_watching` | boolean | `true` | `GTN_CONFIG_ENABLE_WORKSPACE_FILE_WATCHING` | 啟用同步工作流程目錄的檔案監控功能。 |

## 應用程式事件 (Application Events) {: #application-events }

應用程式生命週期事件的組態設定

| 設定項目 (Setting) | 型別 (Type) | 預設值 (Default) | 環境變數 (Environment variable) | 說明 (Description) |
| :--- | :--- | :--- | :--- | :--- |
| `app_events.on_app_initialization_complete.libraries_to_download` | array of (string or [LibraryDownload](#librarydownload)) | `[]` | 不適用（請編輯組態檔案） | 當引擎啟動時自動下載至 `libraries_directory` 的程式庫清單。每個項目可以是純 Git URL 字串，或包含 `git_url` 以及選填 PEP 440 `version` 鎖定版本與資訊清單 `name` 的物件。Git URL 支援完整 URL 或 GitHub 簡寫（例如 `user/repo`）。可選填使用 `@ref` 語法指定分支、標籤或提交（例如 `user/repo@stable` 或 `https://github.com/user/repo@v1.0.0`）。若未指定 ref，則使用儲存庫的預設分支。引擎會將各項目佈署至其鎖定的版本，並可能覆寫錯誤的已安裝版本；僅列在 `libraries_to_register` 中的程式庫絕不會被覆寫。 |
| `app_events.on_app_initialization_complete.libraries_to_register` | array of (string or [LibraryRegistration](#libraryregistration)) | `[]` | 不適用（請編輯組態檔案） | 引擎在啟動時載入的程式庫清單。每個項目可以是單一 `griptape_nodes_library.json` 檔案的路徑，或是包含一個或多個程式庫的資料夾路徑。使用開關來啟用或略過某個程式庫，並在程式庫支援時選擇它與引擎一同執行或在其專屬的隔離程序中執行。 |
| `app_events.on_app_initialization_complete.workflows_to_register` | array of string | `[]` | 不適用（請編輯組態檔案） | 應用程式初始化完成後欲註冊的工作流程清單。 |
| `app_events.on_app_initialization_complete.secrets_to_register` | (array of string) or object | `{"HF_TOKEN": "", "GT_CLOUD_API_KEY": ""}` | `GTN_CONFIG_APP_EVENTS__ON_APP_INITIALIZATION_COMPLETE__SECRETS_TO_REGISTER__<KEY>` | 欲註冊的核心機密密鑰。可以是機密名稱清單（預設為空字串）或是將名稱映射至預設值的字典物件。程式庫專屬的機密密鑰會根據程式庫設定自動註冊。 |
| `app_events.on_app_initialization_complete.models_to_download` | array of string | `[]` | 不適用（請編輯組態檔案） | 應用程式初始化完成後欲自動下載的模型清單。 |
| `app_events.events_to_echo_as_retained_mode` | array of string | `["CreateConnectionRequest", "DeleteConnectionRequest", "CreateFlowRequest", ...]` (22 items) | 不適用（請編輯組態檔案） | 欲作為保留模式 (Retained Mode) 記錄回顯的事件清單。 |

## 專案 (Projects) {: #projects }

專案範本組態與註冊

| 設定項目 (Setting) | 型別 (Type) | 預設值 (Default) | 環境變數 (Environment variable) | 說明 (Description) |
| :--- | :--- | :--- | :--- | :--- |
| `app_events.on_app_initialization_complete.projects_to_register` | array of (string or [PerPlatformProjectPath](#perplatformprojectpath)) | `[]` | 不適用（請編輯組態檔案） | 啟動時載入的專案項目清單。每個項目可以是：(1) 單一路徑字串（支援 `${ENV_VAR}` 與 `~` 展開），或 (2) 帶有選填 `linux`、`darwin`、`windows` 與 `default` 鍵的個別平台映射，適用於同一專案在各作業系統上解析為不同路徑的跨平台部署環境。路徑項目可指向單一 `griptape-nodes-project.yml` 檔案，或指向一個會被遞迴掃描所有 `griptape-nodes-project.yml` 檔案（各個均作為註冊範本載入）的目錄。目錄項目原樣保留並在每次啟動時重新掃描；發現的檔案不會展開為獨立項目。沒有鍵符合活動平台且沒有 `default` 的個別平台項目將被略過並輸出警告。 |
| `app_events.on_app_initialization_complete.requires_engine` | string | `null` | `GTN_CONFIG_APP_EVENTS__ON_APP_INITIALIZATION_COMPLETE__REQUIRES_ENGINE` | 執行中引擎必須滿足的 PEP 440 版本規範字串（例如 `>=0.5,<0.6`）。不符時會阻止專案啟動。通常設定在專案相鄰的組態中，使專案成為其所執行引擎版本的真值來源。 |
| `project_file` | string | `null` | `GTN_CONFIG_PROJECT_FILE` | 引擎啟動時初始載入的專案檔案 (`griptape-nodes-project.yml`) 路徑。設定後，將覆寫預設位置 `<workspace_directory>/griptape-nodes-project.yml`。若指定路徑不存在，則回復至工作區預設值。哨兵值 `'<system-defaults>'` 代表引擎刻意維持系統預設值並抑制工作區預設回退機制（因此不會自動探索工作區的 `griptape-nodes-project.yml`）；這是引擎在刻意維持系統預設時所持久化的值。 |
| `project_workspaces` | object | `{}` | `GTN_CONFIG_PROJECT_WORKSPACES__<KEY>` | 專案識別碼對應至工作區目錄覆寫設定的映射。鍵可以是專案 ID 或專案檔案路徑：首先與已載入的專案 ID 比對，若皆不符則視為專案檔案路徑。載入專案時，若符合此處的鍵，則對應的值將用作工作區目錄，而非專案相鄰組態或自動預設值。 |

## 執行運算 (Execution) {: #execution }

工作流程執行與處理設定

| 設定項目 (Setting) | 型別 (Type) | 預設值 (Default) | 環境變數 (Environment variable) | 說明 (Description) |
| :--- | :--- | :--- | :--- | :--- |
| `log_level` | one of `CRITICAL`, `ERROR`, `WARNING`, `INFO`, `DEBUG` | `"INFO"` | `GTN_CONFIG_LOG_LEVEL` | 引擎日誌詳細程度。可為 CRITICAL、ERROR、WARNING、INFO 或 DEBUG，由最簡略至最詳細。 |
| `workflow_execution_mode` | one of `sequential`, `parallel` | `"sequential"` | `GTN_CONFIG_WORKFLOW_EXECUTION_MODE` | 節點處理的工作流程執行模式。SEQUENTIAL 模式使用 `ParallelResolutionMachine` 搭配 `max_nodes_in_parallel=1` 循序執行節點。PARALLEL 模式使用配置的 `max_nodes_in_parallel` 數值並行執行。 |
| `max_nodes_in_parallel` | integer | `5` | `GTN_CONFIG_MAX_NODES_IN_PARALLEL` | 平行執行模式下同時運算的最大節點數量。 |
| `worker.heartbeat_interval_s` | number | `5.0` | `GTN_CONFIG_WORKER__HEARTBEAT_INTERVAL_S` | 由編排程序發送 Worker 心跳挑戰的間隔秒數。 |
| `worker.heartbeat_timeout_s` | number | `15.0` | `GTN_CONFIG_WORKER__HEARTBEAT_TIMEOUT_S` | Worker 未回應心跳挑戰被判定驅逐前的等待秒數。Worker 在編排程序沉默達此時間後亦會自行關閉，但絕不會早於 30 秒，以免將忙於處理運算的編排程序誤判為已結束。 |
| `worker.library_load_timeout_s` | number | `600.0` | `GTN_CONFIG_WORKER__LIBRARY_LOAD_TIMEOUT_S` | 編排程序將程式庫標記為 FAILURE 之前，Worker 載入其程式庫所允許的秒數。亦限制執行節點等待其程式庫 Worker 完成載入的時間，以及專案切換等待每個 Worker 採納新設定的時間。大型程式庫（如 torch、diffusers）的初次安裝時間輕易可超過兩分鐘。此設定不影響心跳；心跳請參閱 `worker.heartbeat_timeout_s`。 |

## 記錄日誌 (Logging) {: #logging }

引擎日誌存放位置與歷程保留期間

| 設定項目 (Setting) | 型別 (Type) | 預設值 (Default) | 環境變數 (Environment variable) | 說明 (Description) |
| :--- | :--- | :--- | :--- | :--- |
| `logging.log_to_file` | boolean | `true` | `GTN_CONFIG_LOGGING__LOG_TO_FILE` | 除了終端控制台外，亦將引擎日誌寫入檔案。每個引擎程序寫入其專屬檔案，達到 10 MB 時輪替並保留 5 個輪替檔，因此每個程序的總大小設有上限。若您僅需當前執行中工作階段的日誌，可將此選項關閉。 |
| `logging.log_directory` | string | `""` | `GTN_CONFIG_LOGGING__LOG_DIRECTORY` | 存放引擎日誌檔案的目錄絕對路徑。與 `ffmpeg_directory` 類似，此路徑絕不相對於工作區解析：日誌屬於整台電腦而非單一工作區，因此所有工作區與專案共用單一位置。相對路徑會被忽略並輸出警告。留空（預設值）代表 `<XDG_STATE_HOME>/griptape_nodes/logs`。 |
| `logging.log_retention_days` | integer | `7` | `GTN_CONFIG_LOGGING__LOG_RETENTION_DAYS` | 刪除超過指定天數未曾寫入的引擎日誌檔案。在引擎啟動時檢查，並在日誌設定變更時再次檢查。引擎目前正在寫入的日誌檔案無論存在多久絕不會被刪除。設為 0 代表永久保留日誌檔案。 |
| `logging.session_log_buffer_lines` | integer | `5000` | `GTN_CONFIG_LOGGING__SESSION_LOG_BUFFER_LINES` | 引擎為當前工作階段在記憶體中保留的最新日誌行數，使問題報告能包含剛剛發生的事情而無需手動重現。這些行包含 `log_level` 所允許的任何內容，因此若您在報告中需要除錯細節，請在重現問題前將 `log_level` 調升為 DEBUG。設為 0 代表停用，這意味著問題報告僅能包含成功寫入日誌檔的內容。 |

## 儲存系統 (Storage) {: #storage }

資料儲存與持久化組態設定

| 設定項目 (Setting) | 型別 (Type) | 預設值 (Default) | 環境變數 (Environment variable) | 說明 (Description) |
| :--- | :--- | :--- | :--- | :--- |
| `storage_backend` | one of `local`, `gtc` | `"local"` | `GTN_CONFIG_STORAGE_BACKEND` | 用於持久化工作流程資料與產生成果的後端。`local` 將檔案儲存在工作區下的本機檔案系統中；`gtc` 使用 Griptape Cloud 儲存服務。 |
| `auto_inject_workflow_metadata` | boolean | `true` | `GTN_CONFIG_AUTO_INJECT_WORKFLOW_METADATA` | 自動將工作流程中繼資料注入支援格式的已存檔檔案中。 |
| `thread_storage_backend` | `"local"` (constant) | `"local"` | `GTN_CONFIG_THREAD_STORAGE_BACKEND` | 對話對話執行緒的儲存後端。僅支援 `local`（檔案系統）；Griptape Cloud 支援已在 Pydantic AI 遷移中移除。 |

## 系統需求 (System Requirements) {: #system-requirements }

系統資源需求與限制

| 設定項目 (Setting) | 型別 (Type) | 預設值 (Default) | 環境變數 (Environment variable) | 說明 (Description) |
| :--- | :--- | :--- | :--- | :--- |
| `minimum_disk_space_gb_libraries` | number | `10.0` | `GTN_CONFIG_MINIMUM_DISK_SPACE_GB_LIBRARIES` | 程式庫安裝與虛擬環境操作所需的最小硬碟空間（以 GB 為單位）。 |
| `minimum_disk_space_gb_workflows` | number | `1.0` | `GTN_CONFIG_MINIMUM_DISK_SPACE_GB_WORKFLOWS` | 儲存工作流程所需的最小硬碟空間（以 GB 為單位）。 |
| `discovery_max_depth` | integer | `5` | `GTN_CONFIG_DISCOVERY_MAX_DEPTH` | 當註冊項目指向目錄以遞迴探索檔案時（例如 `projects_to_register` 下的專案檔），引擎巡訪目錄的最大深度。限制啟動時的掃描範圍，以防止極深目錄樹與符號連結迴圈。0 僅掃描頂層目錄；每增加一層巢狀結構加 1。 |

## MCP 伺服器 (MCP Servers) {: #mcp-servers }

Model Context Protocol 伺服器組態設定

| 設定項目 (Setting) | 型別 (Type) | 預設值 (Default) | 環境變數 (Environment variable) | 說明 (Description) |
| :--- | :--- | :--- | :--- | :--- |
| `mcp_servers` | array of [MCPServerConfig](#mcpserverconfig) | `[]` | 不適用（請編輯組態檔案） | Model Context Protocol 伺服器組態清單。 |

## 靜態伺服器 (Static Server) {: #static-server }

用於提供媒體資源存取的靜態檔案伺服器設定

| 設定項目 (Setting) | 型別 (Type) | 預設值 (Default) | 環境變數 (Environment variable) | 說明 (Description) |
| :--- | :--- | :--- | :--- | :--- |
| `static_server_base_url` | string | `null` | `GTN_CONFIG_STATIC_SERVER_BASE_URL` | 靜態伺服器的基礎 URL。留空則自伺服器的主機/連接埠衍生（包括當配置連接埠無法使用時由作業系統分配的連接埠）。僅在欲覆寫衍生 URL 時設定此項，例如在伺服器前端架設穿透通道（ngrok、cloudflare）或反向代理時。 |

## 成果資產 (Artifacts) {: #artifacts }

成果提供者與預覽產生的設定

| 設定項目 (Setting) | 型別 (Type) | 預設值 (Default) | 環境變數 (Environment variable) | 說明 (Description) |
| :--- | :--- | :--- | :--- | :--- |
| `artifacts` | object | `{}` | `GTN_CONFIG_ARTIFACTS__<KEY>` | 控制如何為圖片與其他媒體檔案產生預覽圖。 |

## 代理 (Agent) {: #agent }

代理行為與系統提示詞

| 設定項目 (Setting) | 型別 (Type) | 預設值 (Default) | 環境變數 (Environment variable) | 說明 (Description) |
| :--- | :--- | :--- | :--- | :--- |
| `agent.system_prompt` | string | `""` | `GTN_CONFIG_AGENT__SYSTEM_PROMPT` | 附加至代理內建系統提示詞之後的額外文字。用於自訂語調、偏好範式或領域情境。 |

## 程式庫 (Libraries) {: #libraries }

程式庫管理與相依套件安裝設定

| 設定項目 (Setting) | 型別 (Type) | 預設值 (Default) | 環境變數 (Environment variable) | 說明 (Description) |
| :--- | :--- | :--- | :--- | :--- |
| `library.dependency_install_behavior` | one of `always`, `never` | `"always"` | `GTN_CONFIG_LIBRARY__DEPENDENCY_INSTALL_BEHAVIOR` | 控制是否自動安裝程式庫資訊清單中宣告的相依套件。`always` 在註冊時自動下載並安裝相依套件。`never` 略過安裝，並在缺少必要相依套件時將該程式庫標記為效能降級 (Degraded)。 |
| `library.lazy_node_loading` | boolean | `true` | `GTN_CONFIG_LIBRARY__LAZY_NODE_LOADING` | 當設為 True（預設值）時，節點的 Python 模組會在該型別的節點初次被建立時（或該型別被其他方式解析時，如內省查詢）採延遲匯入 (Lazy Import)，而非在啟動時全數匯入，這能大幅加速包含眾多或龐大節點的程式庫之引擎啟動速度。權衡之處在於損壞節點的匯入錯誤直到該型別初次建立時才會回報。當設為 False 時，引擎會在啟動時匯入每個節點的 Python 模組，因此匯入錯誤會在節點放置於畫布前立即作為程式庫問題浮現；如果您在編寫自訂節點時希望進行該預先檢查，請將此項設為 False。沙盒程式庫中的節點無論此設定為何始終採即時積極載入 (Eager Loading)。 |
| `library.minimum_release_age` | number | `0.0` | `GTN_CONFIG_LIBRARY__MINIMUM_RELEASE_AGE` | 套用程式庫更新前，目標發布版本的最少存在時數。當為 0（預設值）時，更新一有釋出即會套用。當大於 0 時，更新會被暫緩保留，直至其欲變更到的目標提交至少已釋出達此時數，以防止自動採用剛推送的新版本而來不及攔截與撤回損壞版本。若無法確定目標提交的釋出時間（例如遠端時間戳記無法讀取），則允許更新（故障開放 fail-open）並記錄警告日誌，因此中繼資料異常絕不會永久阻礙更新。時數是從目標提交的 Git committer 時間戳記開始計算，這不一定等於發布的實際時間：經變基 (Rebase)、擇優挑選 (Cherry-pick)、回溯填補日期 (`GIT_COMMITTER_DATE`) 或強制移動的標籤所報告的時數可能與實際發布時間有所出入。 |

## 測試版功能 (Beta Features) {: #beta-features }

可自由開啟或關閉的實驗性功能

| 設定項目 (Setting) | 型別 (Type) | 預設值 (Default) | 環境變數 (Environment variable) | 說明 (Description) |
| :--- | :--- | :--- | :--- | :--- |
| `beta_features` | object | `{}` | `GTN_CONFIG_BETA_FEATURES__<KEY>` | 啟用或停用的實驗性功能，以功能識別碼 (Feature ID) 為鍵。編輯器的 Beta 設定頁面會寫入這些內容。此映射中缺失的功能採用其預設值。接受任何字串鍵，因此僅限前端編輯器的功能無需等待引擎發布新版。 |
| `library_beta_features` | object | `{}` | `GTN_CONFIG_LIBRARY_BETA_FEATURES__<KEY>` | 由節點程式庫所定義、可啟用或停用的實驗性功能。鍵為全小寫且空格與標點符號替換為底線的程式庫名稱，其後再接功能識別碼。此映射中缺失的功能採用其預設值。 |

## 項目型別 (Entry Types) {: #entry-types }

出現在清單或映射設定內部的物件欄位規格。這些欄位屬於個別項目本身而非全域設定，因此沒有自屬的點分隔鍵或環境變數：請在組態檔案中與該項目一同編輯。

### LibraryDownload {: #librarydownload }

`libraries_to_download` 中由引擎佈署至指定版本的程式庫項目。

| 欄位 (Field) | 型別 (Type) | 預設值 (Default) | 說明 (Description) |
| :--- | :--- | :--- | :--- |
| `git_url` | string | 必要欄位 (required) | 採用引擎 `url@ref` 格式的 Git 來源：完整 URL 或 `user/repo` 簡寫，並附帶選填的 `@branch\|tag\|commit` 後綴（例如 `griptape-ai/griptape-nodes-library-standard@v2.0`）。 |
| `version` | string | `null` | 已安裝程式庫必須滿足的 PEP 440 版本規範字串（例如 `>=1.2,<2`）。設為 None 代表僅依來源鎖定。 |
| `name` | string | `null` | 程式庫名稱，需符合程式庫資訊清單中的 `name`。設定後，將依名稱比對已安裝版本以決定是否需要重新下載。 |

### LibraryRegistration {: #libraryregistration }

`libraries_to_register` 中帶有選填中繼資料的程式庫註冊項目。

| 欄位 (Field) | 型別 (Type) | 預設值 (Default) | 說明 (Description) |
| :--- | :--- | :--- | :--- |
| `path` | string | 必要欄位 (required) | 指向 `griptape_nodes_library.json` 檔案的路徑，或被遞迴掃描的目錄路徑。 |
| `enabled` | boolean | `true` | 當設為 False 時，該程式庫保留於組態中但不會在啟動時載入。 |
| `worker_mode_override` | one of `ORCHESTRATOR`, `WORKER` | `null` | 針對個別程式庫覆寫其資訊清單中宣告的啟動模式。可為 ORCHESTRATOR 或 WORKER。僅在資訊清單宣告 `WorkerModeCompatibility.COMPATIBLE` 時生效；對於 INCOMPATIBLE 程式庫將直接忽略。設為 None 則回復為資訊清單中的 `SuggestedWorkerMode`。 |

### PerPlatformProjectPath {: #perplatformprojectpath }

專案 YAML 路徑的跨平台個別映射。

| 欄位 (Field) | 型別 (Type) | 預設值 (Default) | 說明 (Description) |
| :--- | :--- | :--- | :--- |
| `linux` | string | `null` | 於 Linux 作業系統上使用的數值 |
| `darwin` | string | `null` | 於 macOS 作業系統上使用的數值 |
| `windows` | string | `null` | 於 Windows 作業系統上使用的數值 |
| `default` | string | `null` | 當活動平台的專屬鍵未設定時的備援回退值 |

### MCPServerConfig {: #mcpserverconfig }

單一 MCP 伺服器的組態設定。

| 欄位 (Field) | 型別 (Type) | 預設值 (Default) | 說明 (Description) |
| :--- | :--- | :--- | :--- |
| `name` | string | 必要欄位 (required) | MCP 伺服器的唯一名稱/識別碼 |
| `enabled` | boolean | `true` | 是否啟用此 MCP 伺服器 |
| `transport` | string | `"stdio"` | 傳輸通訊協定類型：stdio、sse、streamable_http 或 websocket |
| `command` | string | `null` | 啟動 MCP 伺服器的指令（stdio 模式下為必填） |
| `args` | array of string | `[]` | 傳遞給 MCP 伺服器啟動指令的引數參數（適用於 stdio） |
| `env` | object | `{}` | MCP 伺服器的專屬環境變數（適用於 stdio） |
| `cwd` | string | `null` | MCP 伺服器的工作目錄（適用於 stdio） |
| `encoding` | string | `"utf-8"` | stdio 通訊採用的文字編碼 |
| `encoding_error_handler` | string | `"strict"` | stdio 通訊的編碼錯誤處理器 |
| `url` | string | `null` | HTTP 基礎連線的 URL（適用於 sse、streamable_http、websocket） |
| `headers` | object | `null` | HTTP 基礎連線的標頭 (Headers) |
| `timeout` | number | `null` | HTTP 逾時秒數 |
| `sse_read_timeout` | number | `null` | SSE 讀取逾時秒數 |
| `terminate_on_close` | boolean | `true` | 是否在連線關閉時終止工作階段（適用於 streamable_http） |
| `description` | string | `null` | 說明此 MCP 伺服器所提供服務的選填描述字串 |
| `capabilities` | array of string | `[]` | 此 MCP 伺服器所提供的功能能力清單 |
| `rules` | string | `null` | 此 MCP 伺服器的選填規則，表示為單一字串。 |
