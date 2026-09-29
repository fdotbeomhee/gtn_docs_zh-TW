# 疑難排解 (Troubleshooting) {: #troubleshooting }

本頁面彙總了使用者最常遇到的問題與異常狀態，並分析其背後成因與解決方法。如果您遇到的問題未在此處列出，請查閱[常見問題 (FAQ)](faq.md)或透過 [FAQ 頁尾的管道](faq.md#where-can-i-provide-feedback-or-ask-questions)與我們取得聯繫。

## 圖像或影片未在編輯器中顯示 {: #images-or-videos-arent-showing-in-the-editor }

**症狀**

- Load Image、Save Image 或媒體預覽節點顯示空白區域而非實際圖像。
- 檔案確實存在於磁碟上（例如在 `{outputs}/images/...` 中），但編輯器無法顯示它。
- 上傳新圖像失敗，並出現類似以下錯誤：

    ```
    Error: CreateStaticFileUploadUrl Failed
    Description: Failed to create presigned URL for file ...: Client error
    '404 Not Found' for url 'http://localhost:8124/static-upload-urls'
    ```

**成因**

編輯器中的多媒體檔案由引擎在連接埠 `8124` 上啟動的本機**靜態檔案伺服器 (Static File Server)** 提供服務。如果該連接埠已被佔用（通常是由於**第二個或殘留的 Griptape Nodes 引擎處理程序仍在背景運行**），新引擎的靜態伺服器將回退到作業系統動態分配的其他連接埠。此時媒體請求會被分散在兩個引擎之間：殘留引擎佔用了預設連接埠，而您實際操作的引擎則在另一個連接埠上提供服務，導致預覽載入失敗並出現 `404` 上傳錯誤。

**解決方法**

1. 首先在編輯器中使用 Ctrl+R (Windows/Linux) 或 Cmd+R (macOS) 重新整理頁面。這可以清除簡單的介面顯示異常。
1. 如果媒體仍未顯示，請確保**僅運行一個引擎實例**。完全結束 Griptape Nodes，然後檢查是否有殘留的引擎處理程序：
    - **Windows**：開啟工作管理員 (Task Manager)，尋找殘留的 `Python` 處理程序並將其結束。
    - **macOS / Linux**：在終端機中運行 `pgrep -fl griptape`（或尋找運行引擎的 `python` 處理程序）並終止殘留處理程序。
1. 如果找不到或無法停止殘留處理程序，請**重新啟動電腦**。這能徹底清除佔用連接埠的殘留引擎。
1. 再次啟動 Griptape Nodes。在初始啟動時應該只有單一引擎處理程序，此時多媒體檔案將恢復正常顯示。

!!! tip "提示"

    重新啟動電腦後，在重新開啟工作流程之前，請先確認只有一個引擎處理程序在運行。特別是在版本更新之後，由先前工作階段殘留的單一孤立引擎是造成此問題最常見的原因。

!!! note "在遠端電腦上運行引擎？"

    如果您的引擎與編輯器運行在不同的電腦上（或位於通道、反向代理之後），在將編輯器指向正確位址之前，媒體遺失是正常現象。請按照[靜態檔案伺服器設定 (Static File Server Configuration)](guides/configuration.md#static-file-server-configuration)中所述設定 `static_server_base_url`。

## 匯入的圖像或影片大小為 0 位元組 (0 bytes) {: #imported-images-or-videos-are-0-bytes }

**症狀**

- 匯入的圖像和影片存入 `{inputs}/images/` 或 `{inputs}/videos/` 時大小為 **0 位元組** (檔案總管顯示 `0 KB`)。
- 媒體無法在編輯器中顯示，重新開啟工作流程也看不到任何媒體。
- 結束並重新開啟 Griptape Nodes 無法解決問題。

**成因**

將檔案引入專案分為兩個步驟：引擎首先透過建立空檔案來預留目的地檔名，然後編輯器將檔案的實際內容發送到引擎在連接埠 `8124` 上運行的本機靜態檔案伺服器。當第二個步驟未完成時，磁碟上就只會留下預留的空檔案。

此問題已定位為作業系統層級的瀏覽器故障，且僅在 Windows 10 上偶爾出現。單純重啟應用程式無法解決該問題。

**解決方法**

- 重新啟動電腦。僅重啟 Griptape Nodes 應用程式是不夠的。

## "Address already in use" / 引擎無法啟動 {: #address-already-in-use-the-engine-wont-start }

**症狀**

啟動時出現類似以下錯誤：

```
The 'websocket_direct' driver could not start: its address is already in use.
Another Griptape Nodes engine is probably already running.
Stop the other engine (or change this driver's port) and try again.
```

**成因**

另一個 Griptape Nodes 引擎實例已在運行，並佔用了當前引擎所需的連接埠。

**解決方法**

1. 停止另一個引擎。關閉其他 Griptape Nodes 視窗，並按照[上方媒體區塊](#images-or-videos-arent-showing-in-the-editor)中的步驟檢查殘留的引擎處理程序。
1. 如果您有意在同一台電腦上同時運行多個引擎，請參閱[在單一裝置上運行多個引擎](#running-multiple-engines-on-one-machine)。

## 在單一裝置上運行多個引擎 {: #running-multiple-engines-on-one-machine }

**症狀**

在同一台裝置上運行兩個或多個引擎時，您會遇到許多看似無關的奇怪錯誤：請求被錯誤的引擎回應（或被重複回應兩次）、工作流程和狀態在不同編輯器工作階段之間發生錯亂、編輯器將多個引擎辨識為同一個、出現如[上述位址被佔用錯誤](#address-already-in-use-the-engine-wont-start)的連接埠衝突，或者出現如[上述圖像問題](#images-or-videos-arent-showing-in-the-editor)的媒體載入失敗。

**成因**

兩個獨立的問題疊加在一起：

- **共享身分 (Shared identity)。** 當未設定 `GTN_ENGINE_ID` 時，本機啟動的每個引擎都會解析為同一個預設引擎身分。共享身分的引擎會監聽相同的請求並共享相同的工作階段狀態，導致它們同時爭搶回應原本只屬於其中一個引擎的請求。這就是大量詭異錯誤的根源。
- **連接埠衝突 (Port conflicts)。** 第一個啟動的引擎會取得預設連接埠（例如靜態檔案伺服器的 `8124`）；後續啟動的引擎會回退到其他隨機連接埠，導致任何仍指向預設連接埠的組件出現通訊中斷。

**解決方法**

為每個額外的引擎**分配專屬的獨立身分與連接埠**：

```bash
GTN_ENGINE_ID=second-engine STATIC_SERVER_PORT=9000 GTN_MCP_SERVER_PORT=9928 gtn engine
```

如果您無意運行多個引擎，請按照[上方媒體區塊](#images-or-videos-arent-showing-in-the-editor)中的步驟尋找並終止多餘的引擎處理程序。

## "No sessions available" — 授權使用者引擎無法啟動 {: #no-sessions-available-the-engine-wont-start-for-license-users }

**症狀**

您是透過授權金鑰啟用（而非登入 Griptape Cloud），且在啟動時引擎授權分配失敗，並提示類似 `No sessions available` 的錯誤。

**成因**

您的組織擁有固定配額的授權工作階段池（基座/席位，Seats）。只要引擎處於運行狀態就會佔用一個席位，並在引擎正常關閉時釋放該席位。`No sessions available` 表示工作階段池中的所有席位當前均已被佔用 — 可能是正常情況（所有成員都在使用），也可能是由**陳舊無效工作階段 (Stale session)** 引起：某個崩潰、被強制終止或仍在背景孤立運行的引擎一直在持續佔用（並自動續約）該席位。

**解決方法**

1. 按照[上方媒體區塊](#images-or-videos-arent-showing-in-the-editor)中的步驟檢查您本機電腦上是否存在孤立殘留的引擎處理程序（尤其是在崩潰或強制結束之後）。終止它即可釋放您的席位。
1. 如果某個席位被卡住，組織所有者可以在後台將其釋放：在[管理員儀表板 (Admin Dashboard)](enterprise/admin_dashboard.md#sessions)中，開啟 **Sessions (工作階段)** 視窗並對陳舊工作階段點擊 **Release (釋放)**。
1. 否則，陳舊工作階段在停止續約並超時過期後會自動釋放，因此稍等數分鐘後重試通常也能恢復正常。

!!! note "注意"

    另一個相關的錯誤 `No session pool configured` 表示您的組織根本未設定授權工作階段池 — 請聯繫管理您 Griptape Nodes 授權的負責人。

## 編輯器畫面全黑或空白 {: #the-editor-is-black-or-blank }

**症狀**

編輯器視窗變為全黑或空白，通常發生在電腦處於閒置/睡眠狀態之後，或短暫網路中斷之後。

**解決方法**

- 使用 Ctrl+Shift+R (Windows/Linux) 或 Cmd+Shift+R (macOS) 強制重新整理編輯器。強制重新整理會重新載入編輯器前端並重新建立與引擎的連線。

## 節點程式庫或節點遺失，或看到來自其他引擎的錯誤 {: #libraries-or-nodes-are-missing-or-you-see-errors-from-another-engine }

**症狀**

- 沒有顯示任何節點程式庫，或者您預期的某個節點（例如 Agent 節點）消失不見。
- 編輯器顯示的錯誤訊息引用了另一個引擎或完全不同的工作流程。

**成因**

通常由以下兩種原因之一引起：

- **某些因素阻止了程式庫載入。** 當程式庫載入失敗時（缺少依賴項、節點程式碼錯誤、匯入錯誤等），其內部的節點會靜默消失。**此時日誌是唯一的真理來源。** 匯出或開啟引擎記錄檔，尋找啟動時與程式庫載入相關的錯誤。
- **Libraries To Register 設定異常。** 引擎僅會載入在 **Libraries To Register** 設定中所列出的程式庫（**Configuration Editor → Libraries → Library Registration**，在 `griptape_nodes_config.json` 中對應儲存為 `app_events.on_app_initialization_complete.libraries_to_register`）。如果某個程式庫未包含在該清單中、被切換為停用狀態，或者其設定項目陳舊無效，其節點將無法顯示。
- 您可能只是單純連線到了與預期不同的另一個引擎，編輯器呈現了該引擎的程式庫與錯誤。

**解決方法**

1. **優先檢查日誌。** 尋找啟動期間程式庫載入時輸出的錯誤。回報的錯誤通常會註明程式庫名稱以及載入失敗的原因。參閱[匯出引擎日誌](#exporting-engine-logs)。
1. 確認編輯器連線的引擎目標。如果多台電腦上都運行有引擎，編輯器可能誤連到了其他實例。
1. 開啟 **Configuration Editor (設定編輯器)**，進入 **Libraries** 檢視，檢查 **Library Registration → Libraries To Register**。如果您需要的程式庫遺失、被關閉或指向陳舊路徑，請修正該項目，或透過 **Manage → Library Management → Add Library** 重新添加該程式庫。參閱[啟用與移除程式庫](guides/libraries.md#toggling-and-removing-libraries)以及[安裝程式庫](guides/editor/managing_models_and_libraries.md#installing-a-library)。
1. 在 **Libraries** 面板中將篩選條件設定為 **Errors**，以檢查安裝或載入失敗的程式庫。參閱[“我安裝了程式庫但看不到其節點”](guides/libraries.md#i-installed-the-library-but-i-dont-see-its-nodes)。
1. 確保您的節點程式庫保持最新版本。開啟 **Manage → Library Management**，展開對應程式庫並點擊 **Check for Updates**，如果有新版本則點擊 **Update**。參閱[更新程式庫](guides/editor/managing_models_and_libraries.md#updating-a-library)。如需更新引擎本體，請參閱 [FAQ](faq.md#how-do-i-update-griptape-nodes)。

## "failed to locate pyvenv.cfg" / 引擎無法啟動 {: #failed-to-locate-pyvenvcfg-the-engine-wont-start }

**症狀**

在啟動時引擎無法正常運行，並提示：

```
failed to locate pyvenv.cfg: The system cannot find the file specified.
```

**成因**

先前的解除安裝操作未完整完成，導致 Griptape Nodes 的虛擬環境處於殘損狀態。

**解決方法**

1. 再次執行解除安裝命令以清理損壞的安裝環境：

    ```bash
    griptape-nodes self uninstall
    ```

    由於 `griptape-nodes` 命令本身也是由該虛擬環境提供的，損壞的虛擬環境有時也會阻止解除安裝命令自身執行。如果在嘗試解除安裝時遇到相同錯誤，請按照[解除安裝 Griptape Nodes](uninstalling.md#manual-engine-install)中的手動清除步驟手動刪除相關目錄。

1. 按照[安裝指南](installation.md)重新安裝。

## "Attempted to create a Flow with a parent 'None'" / 通常無害 {: #attempted-to-create-a-flow-with-a-parent-none-usually-harmless }

**症狀**

在載入或構建工作流程時常會看到此錯誤：

```
Attempted to create a Flow with a parent 'None', but no parent with that name could be found.
```

**成因**

一個已知的偶發性 Bug。在絕大多數情況下它是完全無害的，不會影響您的實際工作。

**解決方法**

1. 通常可以直接忽略該提示並繼續工作。
1. 如果它阻礙了操作，重啟引擎通常即可消除。
1. 如果您可以穩定重現該問題，非常歡迎您向我們[提交 Bug 報告](https://github.com/griptape-ai/griptape-nodes/issues/new?template=bug_report.yml&title=Attempted%20to%20create%20flow%20with%20a%20parent%20%27None%27)，並提供導致該問題發生的前後背景資訊。

## "ssl.SSLCertVerificationError" / 引擎無法運行 {: #sslsslcertverificationerror-the-engine-wont-run }

**症狀**

當您嘗試運行 Griptape Nodes 時出現：

```
ssl.SSLCertVerificationError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1000)
```

**成因**

您電腦上的 Python 安裝環境無法存取受信任的 SSL 憑證鏈。

**解決方法**

1. 使用 [python.org](https://www.python.org/downloads/) 官方安裝程式重新安裝 Python。Griptape Nodes 需要 Python 3.12。
1. 在安裝結束時，選擇 **Install Certificates (安裝憑證)**。
    - 如果安裝程式未提供該選項（例如在 macOS 上），請在終端機中運行 `/Applications/Python\ 3.12/Install\ Certificates.command`。

## 匯出引擎日誌 (Exporting engine logs) {: #exporting-engine-logs }

當您回報問題（或自行深入調查問題）時，引擎日誌通常是首要排查目標。然而，僅憑日誌本身往往難以還原完整情境 — 當時載入了哪些程式庫、哪些設定生效中，以及設定了哪些 API 金鑰都至關重要。診斷打包功能可以一鍵完整蒐集這些資訊。

### 一次性蒐集所有診斷資訊 {: #collecting-everything-at-once }

**診斷封裝包 (Diagnostics bundle)** 是一個包含以下內容的 zip 壓縮檔：

| 封裝包內部檔案 | 提供的診斷價值 |
| --- | --- |
| `logs/*.log` | 磁碟上保存的日誌檔案，按時間由新到舊排序。最上方的是產生該封裝包的工作階段日誌 |
| `logs/session.log` | 引擎本次工作階段在記憶體中記錄的所有內容。僅在日誌尚未寫入磁碟檔案時生成 |
| `report.json` | 運行的引擎版本、主機規格、生效設定，以及各個程式庫和專案載入時的具體狀態 |
| `doctor.json` | [`doctor`](reference/command_line_interface.md#doctor) 健康檢查報告：包含發現的異常及針對各項問題的修復建議 |
| `workflow/` | 產生封裝包時編輯器中當前開啟的最後儲存狀態的工作流程檔案。僅由編輯器匯出時包含 |
| `manifest.json` | 上述所有檔案的清單，以及出於安全考慮已脫敏過濾的機密資訊統計 |
| `README.md` | 說明各個診斷檔案內容的導讀手冊 |

生成診斷封裝包：

```bash
gtn diagnostics collect
```

透過此命令生成的封裝包不包含 `workflow/` 資料夾：因為該命令會啟動其自身的獨立引擎，而該引擎沒有開啟任何工作流程。若要包含您當前正在編輯的工作流程，請改為在編輯器介面內部產生封裝包。

該命令會在當前目錄下生成名為 `griptape-nodes-diagnostics-<version>-<timestamp>.zip` 的檔案。若要將其儲存在便於尋找的位置：

```bash
gtn diagnostics collect --output ~/Desktop
```

您可以將該 zip 檔案附加到您的 Bug 報告中。系統絕不會自動上傳任何內容 — 封裝包完全生成並保存在您的本機，是否共享全由您自主決定。

!!! note "哪些資訊會被自動脫敏過濾，以及分享前應檢查的事項"

    該封裝包由引擎自身寫入，引擎清楚自己的 API 金鑰，因此它會掃描蒐集的每個檔案並自動將其移除 — 同時也會過濾任何符合認證資訊特徵的文字。使用者根目錄路徑會被替換為 `~`，您的使用者名稱會替換為 `<user>`；如果您希望保留原始路徑，可以傳遞 `--show-identity`。所有被過濾的內容都會標記為 `<redacted>`，且 `manifest.json` 會統計每次脫敏操作，因此您可以明確區分某個設定項原本就是空白還是出於隱私被隱藏。

    診斷工具無法識別的是它未曾獲知的非標準格式機密 — 例如手動輸入到節點文字欄位中的密碼，或是某個自訂程式庫以其專屬格式寫入日誌的 Token。在將封裝包上傳到任何公開平台之前，請先快速瀏覽 `logs/` 下的檔案與工作流程檔案。

如果您只需要查看本機健康檢查結果而無需生成檔案，請運行：

```bash
gtn doctor
```

它將以表格形式列印檢查結果，並為需要處理的項目提供修復建議。

### 從桌面應用程式匯出 {: #from-the-desktop-application }

桌面應用程式為其管理的本機引擎維護著獨立的日誌檔案，並且能夠匯出指定時間範圍內的日誌，而不僅限於當前工作階段。當問題發生在一段時間之前或跨越了引擎重啟時，這項功能特別實用。

1. 點擊頂部標題列中的 **Engine** 按鈕（顯示引擎狀態的按鈕）開啟彈出視窗。
1. 在 **Managed Engine** 下，點擊 **Logs** 開啟引擎日誌視窗。
1. 點擊 **Export (匯出)**。
1. 在 **Export Logs** 對話方塊中選擇：
    - **Current Engine Session** — 自引擎上次啟動以來的所有日誌。
    - **Time Range** — 特定時間戳記之間的日誌，可設定 **From** 起始時間，以及 **To** 結束時間或勾選 **Now**。當問題剛剛發生時，匯出過去半小時左右的日誌通常比整個工作階段更具針對性。
1. 選擇儲存 `.txt` 檔案的目的地位置。

!!! note "注意"

    匯出日誌需要啟用 **Write engine logs to file** 設定，該選項位於桌面應用程式的 [App Settings (應用程式設定)](guides/desktop/app_settings.md#logging-and-diagnostics) 中。該設定預設處於啟用狀態；如果 **Export** 按鈕為不可用狀態，請點擊其旁邊的 **Manage** 連結跳轉至該設定項進行啟用。

### 從終端機查看 {: #from-the-terminal }

如果您手動運行引擎（透過 `gtn` 或 `gtn engine`），日誌將直接輸出到該終端機視窗中。您可以直接向上滾動並複製相關內容。

引擎自身也會維護日誌檔案，因此您無需在終端機開啟的狀態下即時捕捉問題。每個引擎處理程序會在 `<XDG_DATA_HOME>/griptape_nodes/logs` 中寫入一個日誌檔案，檔案達到 10 MB 時會自動輪轉 (Roll over)，並自動清理超過一週未被修改的陳舊日誌。有三個設定控制此行為：`logging.log_to_file`、`logging.log_directory` 以及 `logging.log_retention_days`（參閱[設定參考 (Configuration Reference)](reference/configuration_reference.md)）。`gtn diagnostics collect` 命令會為您自動打包這些檔案。

如果日誌詳細程度不足，可以調高引擎的日誌記錄層級：開啟 Configuration Editor (**Settings → All Settings**)，搜尋 "log level"，將其設定為 `DEBUG`，然後重現問題（參閱[在編輯器中修改設定](guides/configuration.md#editing-settings-in-the-editor)）。在無編輯器連接的無介面 (Headless) 模式下運行時，您可以透過環境變數直接指定：

```bash
GTN_CONFIG_LOG_LEVEL=DEBUG gtn
```

!!! tip "提示"

    引擎在記憶體中始終保留最近 5,000 行日誌，因此即使關閉了磁碟日誌記錄，在問題發生後立即打包診斷資訊也能完整捕獲上下文 — 這些記錄將以 `logs/session.log` 形式存在於封裝包中。如果引擎已開啟磁碟日誌，封裝包將直接包含該日誌檔案而省略 `session.log`。無論哪種方式，日誌內容都會遵循當前的日誌等級，因此如果您需要除錯詳細資訊，請在重現問題前將日誌等級調整為 `DEBUG`。`logging.session_log_buffer_lines` 控制記憶體緩衝區保留的最大行數。
