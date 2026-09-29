# 引擎組態配置 (Engine Configuration) {: #engine-configuration }

當在您自己的電腦本機運行 Griptape Nodes 引擎時，系統提供了一套完善的工具來管理各項組態設定。在構建更複雜的專案或與團隊成員共用專案時，深入理解組態設定的載入機制至關重要。

> 在初次安裝期間，系統已自動為您執行了 `gtn init` 初始化命令。

> 尋找特定設定？[組態配置參考手冊](../reference/configuration_reference.md)按類別完整列出了每個設定項的資料型別、預設值、對應環境變數與詳細說明。

## 在編輯器中修改設定 {: #editing-settings-in-the-editor }

變更設定最推薦的方式是使用編輯器內建的**組態編輯器 (Configuration Editor)**：

1. 開啟編輯器標頭中的 **Settings (設定)** 功能表並選擇 **All Settings**。該子功能表中亦有直達特定類別的捷徑（例如 **Engine Settings** 或 **Library Settings**）。
1. 在左側側邊欄中選取分類 — **Editor Settings**、**Engine Settings**、**File System**、**Libraries**、**Library Settings**、**MCP Servers**、**API Keys & Secrets** — 或使用頂部搜尋框依名稱快速篩選設定項目。
1. 修改目標數值。組態編輯器會自動替您將新值寫入磁碟設定檔中。

部分設定需要重新啟動引擎後方可正式生效 — 例如 `static_server_base_url` 即為典型範例（參閱下文[靜態檔案伺服器設定](#static-file-server-configuration)）。

本頁其餘章節將詳細闡明底層的設定架構：組態編輯器所寫入的具體檔案、環境變數規範以及它們之間的合併覆寫優先順序。只有在自動化佈署、無外設環境 (Headless) 運行或跨團隊共用標準組態時，才需要直接手動編輯這些底層檔案。

## 組態載入順序與優先層級 {: #configuration-loading }

Griptape Nodes 採用了嚴謹的層次搜尋順序來從環境變數與組態檔案中載入設定值。理解此過程是管理系統環境的關鍵。

1. **環境變數檔案 (`.env`)**
    環境變數專門用於安全保存機密資訊（如 API 金鑰）。Griptape Nodes 會自動載入 env 檔案，將這些機密注入應用程式執行期。

    - 系統級主 `.env` 檔案載入自全系統使用者組態目錄：`xdg_config_home() / "griptape_nodes" / ".env"`（通常為 `~/.config/griptape_nodes/.env`）。
    - 該檔案專為 `GT_CLOUD_API_KEY`、`OPENAI_API_KEY` 等敏感憑證而設計。

    > 一般情況下無需手動編輯此檔案。Griptape Nodes 會透過 Settings 設定對話方塊自動管理您的環境變數。

1. **組態檔案 (`griptape_nodes_config.json`)**
    組態檔案保存了 Griptape Nodes 運行所需的關鍵資訊，例如節點程式庫的搜尋位置，以及自訂使用者偏好設定。

    - 若未找到任何組態檔案，Griptape Nodes 將完全採用內建的預設值運行。
    - 組態檔案均為標準 JSON 格式 (`griptape_nodes_config.json`)。設定值最多可由三個此類檔案、一個執行期覆寫層以及環境變數共同合併決定：
    - **載入順序（編號越小越先載入；編號越大覆寫優先級越高）：**
        1. **內建預設值 (Built-in defaults)** — 烘焙在應用程式底層程式碼中的基礎預設。
        1. **使用者全域組態 (User config)** — `~/.config/griptape_nodes/griptape_nodes_config.json`。代表本臺電腦的全域設定。
        1. **專案同級組態 (Project-adjacent config)** — `<project_dir>/griptape_nodes_config.json`。當某個專案被啟動時載入。用於隨同專案檔案一同分發團隊共享的預設設定。
        1. **工作區專屬組態 (Workspace config)** — `<workspace_dir>/griptape_nodes_config.json`。在解析工作區路徑後載入。用於個人專屬的自訂設定，其優先級高於專案共用設定。當工作區目錄與專案目錄相同時，該檔案等同於專案同級組態，不會重複載入。
        1. **專案級工作區覆寫 (Per-project workspace override)** — 當當前活動專案的路徑與您使用者組態中 `project_workspaces` 映射表中的某個鍵值相符時，將強制套用該工作區目錄。這僅覆寫 `workspace_directory` 單一欄位，覆寫上述所有組態檔的指定值。無實體檔案保存此項，因此在 `gtn self info` 中標記為 `runtime` 執行期層，在此狀態下透過 GUI 修改 `workspace_directory` 將無法覆寫它。專案範本自帶的 `workspace_dir` 以及繼承自父專案的工作區亦遵循相同機制。若開啟未宣告上述項目的專案，則鎖定您使用者組態中已指定的數值。詳情請參閱[工作區指南](projects/workspace.md#per-project-workspace-overrides)。
        1. **系統環境變數** — 以 `GTN_CONFIG_*` 為前綴（**擁有最高絕對優先權**）。詳見下文。
    - **覆寫優先規則：** 後載入的檔案內容覆寫先載入的同名設定。

1. **預設值與合併行為**
    Griptape Nodes 為各項參數提供了內建預設值，包含預設工作區目錄。除非在載入的組態檔案中被顯式覆寫，否則均採用內建預設值。

    - 從第一個被找到的組態檔案中載入的設定將覆寫內建預設值。
    - 若在所有搜尋路徑中皆未找到組態檔案，應用程式僅使用內建預設值。
    - 其中最關鍵的預設值是 `workspace_directory`，若在組態檔案中未明確指定，它預設採用 `<current_working_directory>/GriptapeNodes`。
    - 空值代表該檔案未配置該項設定。清空某項設定等同於將其從該組態檔案中移除，因此將依序回退至下一層級檔案的數值（或內建預設值）— 無需手動進入 JSON 檔案刪除該鍵。

1. **執行期管理模組 (`ConfigManager`)**
    在初始設定載入完畢後，`ConfigManager` 依據最終解析出的組態（尤其是工作區目錄）掌管執行期的各項運作。它負責將使用者在執行期產生的變更（如註冊的自訂工作流程）持久化寫回工作區內的組態檔案中。

    - 一旦設定載入完成，`ConfigManager` 將以最終解析出的 `workspace_directory` 為基準。
    - 執行期所做的動態變更通常會被 `ConfigManager` 自動儲存至該解析工作區目錄內的 `griptape_nodes_config.json` 檔案中。

## 組態載入案例剖析 {: #loading-examples }

以下幾種典型情境展示了組態檔案如何被定位與載入：

**情境 1：完全使用預設設定**

- 您執行了 `gtn init` 並一路按 Enter 接受所有預設選項。
- `gtn init` 在本機建立了 `~/.config/griptape_nodes/griptape_nodes_config.json` 與 `~/.config/griptape_nodes/.env`。並在 `.json` 檔案中將 `workspace_directory` 指向 `<執行init時的當前目錄>/GriptapeNodes`。
- 您稍後在 `/home/user/my_project/` 路徑下執行了 `gtn`。
- **檔案組織結構：**

    ```
    /home/user/
        my_project/          <-- 執行 'gtn' 時的工作目錄 (CWD)
            GriptapeNodes/   <-- 預設工作區目錄 (可能包含執行期儲存的組態)
            my_flow.graph.json
        .config/
            griptape_nodes/
                .env                     # 載入環境變數機密
                griptape_nodes_config.json # 包含 workspace_directory = /home/user/my_project/GriptapeNodes
    ```

- **載入解析流程：**
    1. 載入程式碼內建預設值。
    1. 載入 `~/.config/griptape_nodes/griptape_nodes_config.json`（成功找到！），將其覆寫合併至預設值之上。
    1. **最終結果：** `workspace_directory` 被成功設定為 `/home/user/my_project/GriptapeNodes`。後續由 `ConfigManager` 管理的執行期變更將寫入 `/home/user/my_project/GriptapeNodes/griptape_nodes_config.json`。

**情境 2：自訂工作區路徑**

- 您執行了 `gtn init --workspace-directory /data/gtn_work`。
- `gtn init` 建立 `~/.config/griptape_nodes/griptape_nodes_config.json`（寫入 `workspace_directory = "/data/gtn_work"`）與 `.env`。
- 您可能手動建立了 `/data/gtn_work/griptape_nodes_config.json` 來存放工作區專屬設定。
- 您在 `/home/user/some_dir/` 執行了 `gtn`。
- **檔案組織結構：**

    ```
    /home/user/
        some_dir/            <-- 執行 'gtn' 時的 CWD
        .config/
            griptape_nodes/
                .env                     # 載入環境變數
                griptape_nodes_config.json # 包含 workspace_directory = /data/gtn_work
    /data/
        gtn_work/            <-- 自訂工作區目錄
            griptape_nodes_config.json # 工作區專屬組態 (同時也是執行期儲存目標)
            project_flows/
    ```

- **載入解析流程：**
    1. 載入內建預設值。
    1. 載入使用者全域組態，將 `workspace_directory` 指定為 `/data/gtn_work`。
    1. 解析工作區位置，載入 `/data/gtn_work/griptape_nodes_config.json` 作為工作區組態，覆寫合併在使用者組態之上。
    1. **最終結果：** 工作區目錄為 `/data/gtn_work`。工作區組態中的參數將覆寫全域組態。執行期變更寫回 `/data/gtn_work/griptape_nodes_config.json`。

**情境 3：不存在使用者組態（回退至內建預設值）**

- 您從未執行過 `gtn init`，或者手動清空了 `~/.config/griptape_nodes/` 資料夾。
- 您在未啟用任何專案的情況下從 `/home/user/my_project/` 啟動了 `gtn`。
- **檔案組織結構：**

    ```
    /home/user/
        my_project/          <-- 執行時的 CWD
            GriptapeNodes/   <-- 預設回退工作區位置
            my_flow.graph.json
    ```

- **載入解析流程：**
    1. 載入內建預設值。
    1. 檢查使用者全域組態（未找到）。
    1. 無活動專案，因此不載入專案組態。
    1. **最終結果：** 應用程式完全依賴內建預設值運作。`workspace_directory` 自動回退至預設的 `<當前工作目錄>/GriptapeNodes` 即 `/home/user/my_project/GriptapeNodes`，執行期變更儲存至該目錄下新生成的 `griptape_nodes_config.json`。

## 環境變數覆寫 (Environment Variable Overrides) {: #environment-variable-overrides }

任何組態設定值皆可透過帶有 `GTN_CONFIG_` 前綴的系統環境變數進行直接覆寫。鍵名為設定項名稱的全大寫格式：

```
GTN_CONFIG_<SETTING_NAME>=<value>
```

對於巢狀物件設定（位於 `worker`、`agent` 或 `library` 等深層結構下的參數），各層級路徑之間使用**雙底線 (`__`)** 進行分隔，同樣保持全大寫：

```
GTN_CONFIG_<PARENT>__<SUB_KEY>=<value>
```

此處強制使用雙底線而非單底線，是因為許多頂層參數名稱本身就包含單底線。若使用單底線，`GTN_CONFIG_WORKER_HEARTBEAT_TIMEOUT_S` 將無法判定究竟是頂層的 `worker_heartbeat_timeout_s` 還是巢狀的 `worker.heartbeat_timeout_s`。系統中所有參數內部均不包含連續雙底線 `__`，因此可百分之百精確切分層級邊界。

環境變數覆寫享有**最高絕對優先權** — 它們會強制覆寫使用者組態檔、專案組態檔、專案工作區覆寫以及內建預設值。

對照範例：

| 設定項目 | 對應系統環境變數 |
| ---------------------------- | ---------------------------------------- |
| `workspace_directory`        | `GTN_CONFIG_WORKSPACE_DIRECTORY`         |
| `libraries_directory`        | `GTN_CONFIG_LIBRARIES_DIRECTORY`         |
| `project_file`               | `GTN_CONFIG_PROJECT_FILE`                |
| `log_level`                  | `GTN_CONFIG_LOG_LEVEL`                   |
| `storage_backend`            | `GTN_CONFIG_STORAGE_BACKEND`             |
| `worker.heartbeat_timeout_s` | `GTN_CONFIG_WORKER__HEARTBEAT_TIMEOUT_S` |
| `library.lazy_node_loading`  | `GTN_CONFIG_LIBRARY__LAZY_NODE_LOADING`  |
| `agent.system_prompt`        | `GTN_CONFIG_AGENT__SYSTEM_PROMPT`        |

這在自動化腳本、Docker 容器以及 CI/CD 管線中極具價值，無需變更任何實體磁碟設定檔即可注入組態：

```bash
GTN_CONFIG_PROJECT_FILE=/shared/studio-project.yml gtn
GTN_CONFIG_WORKER__HEARTBEAT_TIMEOUT_S=30 gtn
GTN_CONFIG_LIBRARY__LAZY_NODE_LOADING=false gtn
GTN_CONFIG_AGENT__SYSTEM_PROMPT="Answer tersely." gtn
```

傳入的字串值在正式生效前會自動轉型為該設定項宣告的資料型別，因此布林設定傳入 `false` 會被解析為布林值 `False`（而非具有真值的字串 `"false"`），數值設定傳入 `30` 會被轉為數字 `30`。注意：此型別自動轉換不適用於映射型別 (Mapping) 設定（如 `GTN_CONFIG_ARTIFACTS__SOME_KEY`），其數值保持原樣傳遞，若需精確的布林或數字型別映射項請改用設定檔。

> **兩項重要限制：**
>
> 1. **清單型設定與區分大小寫的鍵**：清單結構無法在字串環境變數中被 `Settings` 模型解析，因此保存清單的參數（如 `mcp_servers` 或 `app_events.on_app_initialization_complete.libraries_to_register`）無法透過環境變數設定。而映射型設定雖然可透過 `GTN_CONFIG_<NAME>__<KEY>=<value>` 指定，但由於整個變數名在轉換為鍵時會被自動轉為全小寫，因此僅對本身即為全小寫的鍵有效。這使得 `artifacts` 能夠正常使用，但 `project_workspaces`（鍵為區分大小寫的專案 ID 或檔案路徑）與 `secrets_to_register`（大寫密鑰名稱）在實踐中無法可靠傳遞。遇到此類情境，請直接編輯 `griptape_nodes_config.json` 檔案。
> 1. **無法解析的異常數值回退行為**：當傳入無法轉型的錯誤格式（如 `GTN_CONFIG_MAX_NODES_IN_PARALLEL=not-a-number`）時，系統通常會發出警告並回退至組態檔案所定義的值。但有四個特殊設定例外：`log_level`、`workflow_execution_mode`、`thread_storage_backend` 與 `library.dependency_install_behavior`。這四個參數若傳入無法識別的字串，將直接靜默回退至程式碼內建預設值，不發出警告亦不回退至組態檔。
>
> [組態配置參考手冊](../reference/configuration_reference.md)詳盡列出了每個參數所對應的精確環境變數名稱（包含巢狀 `__` 完整路徑）。

### 遞迴探測深度上限 (`discovery_max_depth`) {: #recursive-discovery-depth-discovery_max_depth }

當 `projects_to_register`、`libraries_to_register` 或 `workflows_to_register` 指向某個目錄時，引擎在開機啟動時會對該目錄進行遞迴掃描，尋找相應的專案檔、程式庫資訊清單與工作流程檔案。為防止惡意深層樹狀結構或符號連結 (Symlink) 迴圈卡死開機程序，系統對掃描深度進行了嚴格限制。`discovery_max_depth` 參數控制此深度上限；預設值為註冊目錄下方 **5** 層目錄，已能完全涵蓋一般標準專案結構。

作為一項標準設定，它可以在任何組態檔中配置，或透過環境變數覆寫：

```bash
GTN_CONFIG_DISCOVERY_MAX_DEPTH=20 gtn   # 支援掃描更深層的目錄結構
```

若設定為 `0`，則僅掃描頂層目錄本身（不深入任何子目錄）。

## 工作區目錄 (Workspace Directory) {: #workspace-directory }

在執行 `gtn init` 期間，您會指定一個工作區目錄 (Workspace Directory)。它是存放您所有專案、已儲存工作流程檔案以及專案專屬設定的根目錄。

雖然 `gtn init` 建議使用 `<當前工作目錄>/GriptapeNodes` 作為預設路徑，但您可以自由挑選任何實體路徑。Griptape Nodes 會精確使用您提供的完整路徑，並將其記錄於全域 `griptape_nodes_config.json` 中。

系統**不會**盲目在寫死的固定目錄中搜尋；它完全遵循組態所指定的明確路徑。

### 將程式庫與工作區實體隔離 {: #separating-libraries-from-the-workspace }

預設情況下，下載的擴充程式庫存放在工作區內部的 `libraries/` 資料夾中，因為 `libraries_directory` 預設為相對於 `workspace_directory` 的相對路徑。若您的工作區位於網路共享磁碟機或掛載磁碟等慢速儲存媒體上，將程式庫置於其中可能會降低效能，因為引擎需要頻繁從磁碟讀取程式庫代碼。

您可以將 `libraries_directory` 重新導向至快速本機快閃記憶體儲存 (NVMe SSD) 的**絕對路徑**，同時保持工作區（專案、工作流程與輸出資產）依然存放在遠端網路磁碟機上。當 `libraries_directory` 被設定為絕對路徑時，系統將直接採用該路徑，不再與工作區相對定位：

```json
{
    "workspace_directory": "/Volumes/team-share/GriptapeNodes",
    "libraries_directory": "/Users/me/.griptape-nodes-libraries"
}
```

等效的環境變數設定指令：

```bash
GTN_CONFIG_LIBRARIES_DIRECTORY=/Users/me/.griptape-nodes-libraries gtn
```

相同的相對與絕對路徑解析規則亦適用於 `sandbox_library_directory` 與 `static_files_directory`，讓您能夠將它們各自獨立移轉至本機儲存空間。

此處的 `libraries_directory` 屬於全系統全域預設值。個別專案可以在其專案檔內部透過自訂 [`libraries_dir`](projects/projects.md#libraries-directory) 欄位進行獨立覆寫，專案檔內的宣告具有更高優先權，且隨專案一同流轉（並會被子專案繼承）。

## 引擎主動發布的環境變數 {: #variables-the-engine-publishes }

上述提及的所有環境變數均由**您**負責設定。而引擎在開機啟動時，也會在其自身的進程環境中發布一個專門供您**讀取使用**的環境變數：

| 變數名稱 | 變數值含義 |
| ---------------------------- | ----------------------------------------------------------------- |
| `GTN_DEFAULT_LIBRARIES_ROOT` | 擴充程式庫在預設情況下所安裝目錄的本機絕對路徑 |

您無需手動設定此變數。其設計宗旨是讓專案檔案能夠動態引用此台機器引擎存放程式庫的實體目錄，而無需為每台電腦硬編碼寫死絕對路徑：

```yaml
libraries_dir: "${GTN_DEFAULT_LIBRARIES_ROOT}/shared"
```

該變數會精準解析為 `griptape-nodes init` 安裝標準程式庫時的實體路徑，使專案能夠直接共享既有的程式庫目錄，避免重複下載。由於該值始終為絕對路徑，因此可以安全地在 `libraries_dir` 中引用（`libraries_dir` 會將相對路徑錨定至專案檔所在目錄，而非工作區）。

該發布值真實反映了本機引擎存放程式庫的各項設定（包含 `libraries_directory` 與 `GTN_CONFIG_LIBRARIES_DIRECTORY`）。它刻意**不跟隨**專案內部的 `libraries_dir`，因為正是該欄位在讀取它。該變數僅在啟動時計算一次。

!!! warning "版本相容性警告"

    若專案檔案中引用了任何尚未定義的環境變數，該專案在**載入階段將被系統明確拒絕**且無法開啟。因此，使用 `${GTN_DEFAULT_LIBRARIES_ROOT}` 語法的專案檔要求運行的引擎版本必須支援發布該變數。

## 靜態檔案伺服器設定 (Static File Server Configuration) {: #static-file-server-configuration }

在運行 Griptape Nodes 時，本地靜態檔案伺服器負責託管工作流程產生的多媒體資產（圖像、影片、音訊）。`static_server_base_url` 設定控制在生成這些資源的存取連結時所採用的基礎 Base URL。預設情況下使用 `http://localhost:8124`，但在使用穿透穿透通道 (Tunnels)、反向代理 (Reverse Proxy) 或容器化部署時，您可能需要進行覆寫。

### 何時需要覆寫此設定 {: #when-to-override-this-setting }

在以下幾種架構情境中，您需要明確配置 `static_server_base_url`：

- **穿透通道服務 (Tunneling Services)**：使用 ngrok、Cloudflare Tunnels 或類似工具將本機伺服器暴露至網際網路時。
- **Docker / Kubernetes 容器化環境**：在容器內運行，且內部監聽位址與外部實際存取端點不同時。
- **反向代理伺服器**：架設在 NGINX、Apache 或其他反向代理伺服器後方時。
- **遠端主機開發**：在遠端雲端伺服器上執行引擎，並透過本機瀏覽器跨網段存取 UI 介面時。
- **團隊協同工作**：將您本機運行的實例即時分享給其他需要預覽已生成多媒體的團隊成員時。

### 如何配置 {: #how-to-configure }

靜態伺服器基礎 URL 可直接在 Griptape Nodes GUI 設定對話方塊中的 `static_server_base_url` 欄位進行配置。若未顯式指定，預設採用 `http://localhost:8124`（若已定義 `STATIC_SERVER_HOST` 與 `STATIC_SERVER_PORT` 環境變數，則優先依循之）。若要還原為預設值，只需將欄位清空即可。

修改此設定後，必須重新啟動 Griptape Nodes 引擎以使變更正式生效。

### 實戰應用範例 {: #example-scenarios }

**情境 1：搭配 ngrok 進行本機聯調**

您正在測試需要存取生成多媒體檔案的外部 Webhook 整合：

1. 啟動 ngrok 穿透通道：`ngrok http 8124`
1. 複製產生的公開 URL（例如：`https://abc123.ngrok.app`）
1. 開啟 Griptape Nodes GUI 設定對話方塊
1. 將 `static_server_base_url` 設定修改為您的 ngrok URL
1. 重啟 Griptape Nodes 引擎：`gtn`

此時當工作流程生成多媒體時：

- 本機存取：直接透過通道 URL 順暢載入。
- 外部服務：可透過 ngrok 公開 URL 擷取多媒體。
- CORS 跨域資源共用：自動針對該穿透通道 URL 完成配置。
