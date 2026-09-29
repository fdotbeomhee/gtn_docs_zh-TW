# 管理伺服器 (Admin Server) {: #admin-server }

管理伺服器 (Admin Server) 運行於您的內部網路之中，作為代表所有 Griptape Nodes 應用程式執行個體與 Griptape Cloud 進行通訊的單一主機。您的各個執行個體指向 Admin Server 而非直接連線至 `cloud.griptape.ai`，伺服器會將每個請求向上游轉發。這讓您在讓個別應用程式執行個體與公用網際網路完全隔離的同時，仍能持續完成授權啟用、執行工作階段並使用 Griptape Cloud 的各項雲端功能。

## 採用效益 {: #why-use-it }

在內部部署環境中運行 Griptape Nodes 的工作室通常不希望每個應用程式執行個體都能自行存取公用網際網路。Admin Server 為您提供了集中管理該連線的單一樞紐：

- **嚴格鎖定執行個體**：應用程式執行個體僅與 Admin Server 通訊，完全無需直接對外連線至網際網路。
- **單一對外出口 (One Egress Point)**：無需為每個執行個體開啟網際網路連線，僅需在防火牆中放行單一主機——只需維護一條規則，並擁有集中審計對外流量的單一端點。
- **集中化組態管理**：您可在單一位置集中設定 Griptape Cloud 的連線端點，並可選填配置允許離開內部網路的特定 Cloud 路徑，無需逐一配置每個執行個體。

若您的應用程式執行個體本就可以直接連線至 `cloud.griptape.ai` 且在您的企業環境中是可以接受的，則您不需要部署 Admin Server。

有關 Admin Server 的網路拓撲定位與流量走向圖，請參閱 [系統架構：內部部署組態](../architecture.md#on-premises-configuration)。

## 取得 Admin Server {: #getting-the-admin-server }

Admin Server 僅提供給企業客戶。請 [聯絡 Foundry](https://www.foundry.com/products/griptape/request-demo) 取得適用於您部署環境的套件檔案。

## 功能特性 (Capabilities) {: #capabilities }

- **轉發雲端請求**：將應用程式的 Griptape Cloud 請求轉發至已配置的上游端點（預設為 `https://cloud.griptape.ai`）並完整保留呼叫端的 `Authorization` 標頭，使授權驗證與工作階段得以持續正常運作。Griptape Cloud 始終是身份驗證與授權的最終權威。
- **開機啟動驗證**：在開機啟動時驗證維運人員的 Griptape Cloud API 金鑰，使錯誤配置的部署在開機時立即快速失敗 (Fail-fast)，避免日後產生隱蔽故障。
- **輸出流量路徑過濾 (Egress Filtering)**：可自訂選擇允許離開內部網路的具體 Cloud API 路徑（詳見 [`forwarding`](#forwarding)）。
- **健全度檢查端點**：本機提供 `GET /health` 端點，回傳 `{"status":"ok"}` 供活性 (Liveness) 與就緒性 (Readiness) 探針檢測。
- **結構化記錄日誌**：每個請求在完成時均會以 JSON 或文字格式記錄一次（包含方法、路徑、狀態碼、延遲時間、用戶端 IP、寫入位元組數），方便審計追蹤。

## 組態設定 (Configuration) {: #configuration }

Admin Server 從 `config.yaml` 檔案讀取其設定。組態依下列順序進行解析，後者優先覆寫前者：

1. 內建預設值
1. `config.yaml` 檔案（預設路徑；啟動時可指向其他位置）
1. 環境變數

!!! note "本文件所適用的版本"

    本頁面記載 Admin Server 0.3.0 及更新版本——請執行 `./server -version` 檢查您的版本。在較舊版本中，`read_timeout` 與 `write_timeout` 預設為 `30s`，這會在串流回覆中途強制切斷；具體設定指引請參閱 [下方警告說明](#server)。

完整的 `config.yaml` 預設值如下所示：

```yaml
server:
  host: "0.0.0.0"
  port: 8080
  read_header_timeout: "10s"
  write_stall_timeout: "60s"
  idle_timeout: "120s"
  shutdown_timeout: "10s"

upstream:
  base_url: "https://cloud.griptape.ai"
  timeout: "120s"
  # 存放 Griptape Cloud API 金鑰的環境變數名稱。
  # 金鑰數值絕不應儲存於此——僅填入變數名稱。
  api_key_env: "GT_CLOUD_API_KEY"

logging:
  level: "info"   # debug | info | warn | error
  format: "json"  # json | text

forwarding:
  mode: "allow_all"  # allow_all | allow | deny
  rules: []
```

### server 伺服器設定 {: #server }

| 鍵 (Key) | 預設值 (Default) | 說明 (Description) |
| :--- | :--- | :--- |
| `host` | `0.0.0.0` | 伺服器監聽的網路位址。 |
| `port` | `8080` | 伺服器監聽的連接埠。 |
| `read_header_timeout` | `10s` | 請求標頭送達的截止時間（時間間隔字串，例如 `10s`）。 |
| `write_stall_timeout` | `60s` | 對用戶端單次寫入允許阻塞的最大時間，超過則斷開連線。在每次寫入前重新計時，因此不會限制整個回應所能花費的總時長。設為 `0` 代表停用。 |
| `idle_timeout` | `120s` | 請求之間保持閒置 keep-alive 連線開啟的最長時間。 |
| `shutdown_timeout` | `10s` | 優雅關機期間等待處理中請求完成的截止時間。 |
| `read_timeout` | `0` (停用) | **已棄用**。讀取**整個**請求的截止時間，因而會限制上傳的最大耗時。請改用 `read_header_timeout`。 |
| `write_timeout` | `0` (停用) | **已棄用**。寫入**整個**回應的截止時間——任何大於 `0` 的數值都會在串流回應傳輸中途切斷。請改用 `write_stall_timeout`。 |

!!! warning "`write_timeout` 會中斷串流回應"

    本頁面與範本設定檔的早期版本曾隨附 `read_timeout: "30s"` 與 `write_timeout: "30s"`。若您的 `config.yaml` 包含這兩行，**請務必將其修改為 `"0"`**：

    ```yaml
    server:
      read_timeout: "0"
      write_timeout: "0"
    ```

    設定為 `"0"` 在所有版本上皆是正確的作法。*刪除* 該行僅在 0.3.0 及更新版本有效（因為其預設值已是 `0`）——在早期組建中刪除該行會回退為該組建內建的 `30s` 預設值，導致回覆依然會中斷。當所有部署皆升級至 0.3.0 或更新版本後，即可完全移除這兩行。

    `write_timeout` 是針對整個回應的截止時間（從請求到達那一刻起算），而非閒置逾時。AI 代理的回覆是逐字元串流傳輸，通常耗時超過 30 秒，因此伺服器會在超過該截止時間時將其中途掐斷。在 Griptape Nodes 中這表現為對話文字在中途突然停止，應用程式日誌中則顯示：

    ```text
    httpx.RemoteProtocolError: peer closed connection without sending complete message body
    ```

    `write_stall_timeout` 取代了該設定：它僅限制單次寫入的最大阻塞時間，因此停滯的用戶端仍會被斷開，而正常的串流則可按需執行所需的時間。

    您無需修改檔案即可測試此行為——環境變數優先於 `config.yaml`：

    ```bash
    export SERVER_WRITE_TIMEOUT=0
    # 接著重新啟動 Admin Server
    ```

### upstream 上游設定 {: #upstream }

| 鍵 (Key) | 預設值 (Default) | 說明 (Description) |
| :--- | :--- | :--- |
| `base_url` | `https://cloud.griptape.ai` | 伺服器所轉發的 Griptape Cloud 根端點 URL。 |
| `timeout` | `120s` | 等待 Griptape Cloud **開始**回應的最長時間。它絕不會限制整個回應的傳輸總長，因此串流回覆不受影響。**非串流**請求在完整答案就緒前不會發送任何資料，因此預設值較為寬鬆——若長時間的生成任務回傳 `502 Bad Gateway`，請調高此數值。 |
| `api_key_env` | `GT_CLOUD_API_KEY` | 存放 Griptape Cloud API 金鑰的環境變數**名稱**。 |

Admin Server 需要一組 Griptape Cloud API 金鑰，並在啟動時驗證以確認操作人員擁有 Griptape 組織。該金鑰**不會**用於轉發請求路徑——您的應用程式依然會發送其自有的 `Authorization` 標頭，伺服器會原樣轉發。

金鑰本體絕不儲存於 `config.yaml` 中。取而代之的是，由 `api_key_env` 指定讀取該金鑰的環境變數名稱（預設為 `GT_CLOUD_API_KEY`），並在環境中設定該金鑰：

```bash
export GT_CLOUD_API_KEY="gt-..."
```

若欲使用其他變數名稱，請設定 `api_key_env` 並將金鑰匯出至該自訂變數中。

!!! warning "切勿將 API 金鑰提交至版本控制"

    請將 API 金鑰保存在環境變數中，而非 `config.yaml`。組態檔案僅記錄讀取金鑰的環境變數名稱，因此金鑰本身絕不會進入可能提交至 Git 的檔案中。

### logging 記錄日誌 {: #logging }

| 鍵 (Key) | 預設值 (Default) | 說明 (Description) |
| :--- | :--- | :--- |
| `level` | `info` | 日誌詳細程度：`debug`、`info`、`warn` 或 `error`。 |
| `format` | `json` | 日誌輸出格式：`json` 或 `text`。 |

Admin Server 將日誌輸出至標準輸出 (stdout) 與標準錯誤 (stderr)——它不會直接寫入實體檔案。若您需要檔案，請在啟動時重導向（`./server -config config.yaml > admin-server.log 2>&1`），或交由容器執行階段或服務管理員收集。若您打算人工直接肉眼檢視日誌而非送往記錄日誌收集器，請設定 `format: "text"`。

在診斷提前結束的請求時，以下三行日誌非常具有參考價值：

| 日誌訊息 | 意涵 |
| :--- | :--- |
| `starting server` | 列出實際生效的各項逾時設定（已解析 `config.yaml` 與環境變數覆寫）。在進行任何其他排查前請先確認此行。 |
| `response stream aborted before completion` | 回應未能正常傳輸完成。包含 `bytes_out`（已發送位元組數）與 `request_id`。記錄為 warning 級別，因為常見原因是使用者在回覆中途中斷關閉分頁；另一個原因則是寫入截止時間到達。 |
| `panic recovered` | Admin Server 本身出現異常崩潰 (Panic)。呼叫端收到 `500` 錯誤。建議向原廠回報。 |

### forwarding 轉發設定 {: #forwarding }

`forwarding` 區塊控制哪些 Cloud API 路徑允許穿透 Admin Server 輸出。這是針對您內部網路的輸出流量控制層，疊加於上游本身的存取權限控管之上。

| 鍵 (Key) | 預設值 (Default) | 說明 (Description) |
| :--- | :--- | :--- |
| `mode` | `allow_all` | 轉發模式：`allow_all`、`allow` 或 `deny`（詳見下文）。 |
| `rules` | _(空清單)_ | 欲允許或拒絕的路徑清單。每條規則為絕對路徑；尾隨 `/*` 代表前綴匹配。 |

三種轉發模式：

- **`allow_all`**（預設值）— 轉發所有路徑；忽略 `rules`。
- **`deny`** — 轉發除匹配路徑**以外**的所有內容。適合排除少數特定例外。
- **`allow`** — **僅**轉發匹配的路徑。以最小化暴露面建立高度防禦態勢。

每條規則均為絕對路徑。尾隨 `/*` 代表前綴匹配（`/api/proxy/*` 匹配 `/api/proxy` 及其底下的所有子路徑）；否則採精確匹配。未獲准的路徑會在本地直接回應用戶端 `403 {"error":"path not permitted"}`，絕不會送達上游。

例如，若欲在轉發其餘所有內容的同時，將 Griptape Cloud 模型代理攔截於內部網路：

```yaml
forwarding:
  mode: "deny"
  rules:
    - "/api/proxy/*"
```

若欲建立最嚴格的鎖定態勢，請使用 `allow` 模式並僅列出應用程式在執行階段必需的路由。其他任何路徑皆無法輸出。以下為必要路由——若 `allow` 模式遺漏其中任何一項，Admin Server 將**拒絕啟動**：

```yaml
forwarding:
  mode: "allow"
  rules:
    - "/api/sessions/*"     # 工作階段配置與生命週期，包含 /api/sessions/{id}
    - "/api/session-renew"  # 維持工作階段存活
    - "/api/session-release" # 結束工作階段
    - "/api/users"          # 啟動時以及每次心跳時提取
    - "/api/organizations"  # 啟動時以及每次心跳時提取
```

這是產品運行所不可或缺的最小路由集合。僅針對您希望開放的 Cloud 功能新增額外規則（例如開放模型代理可新增 `/api/proxy/*`）。

!!! note "輸出流量控制，而非身份驗證"

    轉發規則決定了哪些路徑可以離開您的內部網路——它們不是身份驗證；Griptape Cloud 始終是決定誰能呼叫什麼資源的最終權威。應用程式執行階段所需的核心路由（工作階段生命週期、使用者與組織路由）絕不會被封鎖：若您的設定可能封鎖它們，Admin Server 會在開機時直接拒絕啟動並明確列出有問題的路由。這確保您可以安心收緊網路出口而不會意外破壞產品功能。

## 環境變數覆寫 (Environment variable overrides) {: #environment-variable-overrides }

所有設定皆可透過環境變數進行覆寫，環境變數的優先權高於 `config.yaml`：

| 環境變數 | 預設值 | 說明 |
| :--- | :--- | :--- |
| `GT_CLOUD_API_KEY` | _(必填)_ | Griptape Cloud API 金鑰（或由 `upstream.api_key_env` 指定的變數）。 |
| `SERVER_HOST` | `0.0.0.0` | 監聽位址。 |
| `SERVER_PORT` | `8080` | 監聽連接埠。 |
| `SERVER_READ_HEADER_TIMEOUT` | `10s` | 請求標頭截止時間。 |
| `SERVER_WRITE_STALL_TIMEOUT` | `60s` | 用戶端停止讀取時的單次寫入截止時間。 |
| `SERVER_IDLE_TIMEOUT` | `120s` | 閒置 keep-alive 連線截止時間。 |
| `SERVER_SHUTDOWN_TIMEOUT` | `10s` | 優雅關機截止時間。 |
| `SERVER_READ_TIMEOUT` | `0` (停用) | **已棄用**。整個請求讀取截止時間；限制上傳最大時長。 |
| `SERVER_WRITE_TIMEOUT` | `0` (停用) | **已棄用**。整個回應寫入截止時間；會中斷串流回覆。 |
| `UPSTREAM_BASE_URL` | `https://cloud.griptape.ai` | 上游 Griptape Cloud 根端點。 |
| `UPSTREAM_TIMEOUT` | `120s` | 上游回應標頭等待逾時時間。 |
| `UPSTREAM_API_KEY_ENV` | `GT_CLOUD_API_KEY` | 存放 API 金鑰的環境變數名稱。 |
| `LOG_LEVEL` | `info` | 日誌詳細程度。 |
| `LOG_FORMAT` | `json` | 日誌輸出格式。 |
| `FORWARDING_MODE` | `allow_all` | `allow_all`、`allow` 或 `deny`。 |
| `FORWARDING_RULES` | _(空字串)_ | 以逗號分隔的允許或拒絕路徑清單。 |

時間長度數值必須帶有單位——例如 `30s`、`2m`——設為 `0` 則停用逾時。未帶單位的數值（如 `SERVER_READ_TIMEOUT=30`）會導致 Admin Server 直接拒絕啟動，而非靜默採用與您預期不同的數值。

## 執行伺服器 {: #running-it }

1. 在環境中設定 Griptape Cloud API 金鑰：

    ```bash
    export GT_CLOUD_API_KEY="gt-..."
    ```

1. 提供 `config.yaml`（請參閱 [組態設定](#configuration)）並啟動 Admin Server。預設情況下，伺服器監聽 `0.0.0.0:8080` 並轉發至 `https://cloud.griptape.ai`。

1. 將您的 Griptape Nodes 應用程式執行個體指向 Admin Server 的位址，而非 `cloud.griptape.ai`。

若 API 金鑰遺失或無效，或是上游端點無法連線，Admin Server 會記錄具體原因並直接退出而不對外提供服務——因此組態問題會在啟動開機時立即浮現。

## 疑難排解 (Troubleshooting) {: #troubleshooting }

- **對話文字在中途中斷**，或應用程式日誌記錄 `peer closed connection without sending complete message body`：這代表串流回覆被切斷。這幾乎總是因為 `config.yaml` 中設定了 `write_timeout` 所致——請參閱 [上方警告說明](#server)。請將其設為 `"0"`（或設定 `export SERVER_WRITE_TIMEOUT=0` 進行覆寫）並重啟伺服器。

    Admin Server 自有的日誌能印證這一點。`starting server` 那一行日誌會印出目前實際生效的每項逾時數值，請先確認其中的 `write_timeout`；`response stream aborted before completion` 警告則顯示伺服器結束了回應並標註當時已發送的位元組數。若 `write_timeout=0s` 且回覆依然被截斷，則代表切斷發生在其他位置——請檢查位於 Admin Server 前端是否有負載平衡器、Ingress 控制器或 TLS 終結點，這些元件各自擁有獨立的回應逾時設定。

- **非串流的長任務生成回傳 `502 Bad Gateway`**：`upstream.timeout` 限制了 Griptape Cloud *開始*回應所允許的最大耗時，而非串流生成在答案完全就緒前不會發送任何資料。請調高該數值。

- **大檔案上傳中途中斷失敗**：`read_timeout` 限制了請求內容送達的總時間，在實務上限制了上傳大小。在 0.3.0 及更新版本中此設定預設為停用；若您的 `config.yaml` 有設定該項，請將其設為 `"0"`。

- **伺服器無法啟動**：確認 `GT_CLOUD_API_KEY` 是否已設定且有效；Admin Server 在開機時會驗證該金鑰，若無法通過驗證則採故障關閉 (Fail-closed)。若啟動錯誤指名環境變數問題（如 `invalid SERVER_WRITE_TIMEOUT: "30" is not a duration`），代表時間數值遺漏了單位——請改用 `30s`。

- **每個請求皆回傳 `502`**：Admin Server 無法連線至上游。請檢查向外至 `upstream.base_url` 的網路連線、DNS 解析狀態，以及是否有企業代理伺服器進行 TLS 攔截。

- **收到 `403 {"error":"path not permitted"}`**：代表被 `forwarding` 轉發規則所攔截。若該路徑應允許輸出，請調整 `forwarding.mode` 或 `forwarding.rules`。
