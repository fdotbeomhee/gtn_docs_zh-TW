# 安裝 Griptape Nodes {: #installing-griptape-nodes }

運行 Griptape Nodes 有兩種主要方式：

- **[Griptape Nodes Desktop](#griptape-nodes-desktop-recommended)** (推薦) — 一款完全受控的桌面應用程式，內建引擎 (Engine) 與編輯器 (Editor)。它會自動處理登入、為您佈建 API 金鑰，並保持一切為最新狀態。完全無需使用終端機。
- **[手動安裝引擎](#advanced-manual-engine-install)** — 適用於希望從命令列自行安裝並運行引擎的高級使用者，搭配基於網頁瀏覽器的編輯器使用。

如果您的組織為您提供的是**授權金鑰 (License Key)** 而非個人的 Griptape 帳號，請參閱下方的[透過授權啟用](#activating-with-a-license)。

## Griptape Nodes Desktop (推薦) {: #griptape-nodes-desktop-recommended }

Griptape Nodes Desktop 是一款集成了引擎與編輯器的單一應用程式。它會自動安裝並管理引擎、協助您登入，並代為佈建 Griptape API 金鑰 — 完全無需手動設定。

### 1. 下載並安裝 {: #1-download-and-install }

從 [griptapenodes.com](https://griptapenodes.com) 下載適用於您作業系統平台的 Griptape Nodes Desktop。它支援 macOS、Windows 和 Linux。

### 2. 登入 {: #2-sign-in }

啟動應用程式並點擊 **Login or Sign-Up**，使用您的 Griptape 帳號登入。

<!-- screenshot: the desktop app login screen showing the Login or Sign-Up button and the Activate with a License button -->

> 如果您已經註冊了 [Griptape Cloud](https://cloud.griptape.ai)，您現有的認證資訊可以直接在此使用！

如果您的組織向您發放了授權金鑰而非 Griptape 帳號，請改為點擊 **Activate with a License** — 請參閱[透過授權啟用](#activating-with-a-license)。

### 3. 選擇您的工作區目錄 {: #3-choose-your-workspace }

首次運行時，系統會要求您選擇一個*工作區目錄 (Workspace Directory)*。您的工作區目錄是 Griptape Nodes 儲存您的[專案檔案](./glossary.md#project-files)和[生成資產](./glossary.md#generated-assets)的位置。您可以接受預設路徑或選擇您偏好的任何位置。

<!-- screenshot: the desktop app first-run workspace setup step with the default workspace directory shown -->

大功告成！應用程式會處理其餘一切事項 — 安裝引擎、生成 API 金鑰並開啟編輯器。您現在已準備好進入[教學 (Tutorials)](tutorials/index.md)。

您稍後可以在[應用程式設定 (App Settings)](guides/desktop/app_settings.md)中變更工作區目錄、管理授權以及選擇應用程式的更新方式。

## 高級：手動安裝引擎 {: #advanced-manual-engine-install }

更傾向於自行管理引擎？透過這種方式，您可以從命令列安裝引擎，並在網頁瀏覽器中透過 [https://nodes.griptape.ai](https://nodes.griptape.ai) 使用編輯器。

編輯器 (Editor) 與引擎 (Engine) 是解耦的，兩者透過事件服務 (Event Service) 相互通訊，因此引擎無需與瀏覽器運行在同一台電腦上。如果您希望引擎能夠存取比筆記型電腦更多的運算資源，可以將其運行在獨立的伺服器或工作站上 — 以下說明對這兩種方式均完全適用。

### 1. 註冊或登入 {: #1-sign-up-or-log-in }

首先，請造訪 [https://griptapenodes.com](https://griptapenodes.com) 並點擊登入按鈕。

> 如果您已經註冊了 [Griptape Cloud](https://cloud.griptape.ai)，您現有的認證資訊可以直接在此使用！

登入後，編輯器會開啟並顯示引擎設定對話方塊（只要未連接任何引擎，該對話方塊就會自動顯示）。選擇 **Get Started** 標籤頁並展開 **Install Engine Manually**，按照以下步驟進行操作。

<!-- screenshot: the engine setup dialog on the Get Started tab, showing the Griptape Nodes Desktop download card and the Install Engine Manually accordion expanded -->

### 2. 安裝引擎 {: #2-install-the-engine }

1. 如果尚未安裝 [uv](https://docs.astral.sh/uv/getting-started/installation/)，請先完成安裝。

1. 運行以下命令安裝 Griptape Nodes Engine：

    ```bash
    uv tool install griptape-nodes
    ```

安裝完成後，*首次*在終端機中運行 `griptape-nodes`（或簡稱 `gtn`），系統將引導您回答一系列設定問題。

### 3. 設定 {: #3-configuration }

**首先**，系統會提示您輸入 Griptape API Key。此金鑰允許引擎與編輯器通訊。

1. 返回網頁瀏覽器中的編輯器標籤頁。在引擎設定對話方塊中，於 **Get Started** 標籤頁的 **Install Engine Manually** 區塊下，開啟 **Generate an API Key** 步驟。

1. 點擊 **Generate API Key** 按鈕。您的新金鑰將取代按鈕顯示。

1. 複製該金鑰並貼到終端機提示字元中。該金鑰在瀏覽器中僅顯示一次，因此請務必在繼續之前複製儲存。

<!-- screenshot: the Generate an API Key step in the Install Engine Manually section, with a generated key shown in place of the button -->

```
╭─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Griptape API Key                                                                                                        │
│         A Griptape API Key is needed to proceed.                                                                        │
│         This key allows the Griptape Nodes Engine to communicate with the Griptape Nodes Editor.                        │
│         In order to get your key, return to the https://nodes.griptape.ai tab in your browser and click the button      │
│         "Generate API Key".                                                                                             │
│         Once the key is generated, copy and paste its value here to proceed.                                            │
╰─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
Griptape API Key (YOUR-KEY-HERE):
```

!!! info "提示"

    如果您之前運行過 `gtn init`，對話方塊可能會直接顯示您的金鑰。您可以按 Enter 鍵接受它，或根據需要輸入不同的值。

**其次**，系統會提示您設定*工作區目錄 (Workspace Directory)*。工作區目錄是引擎儲存[專案檔案](./glossary.md#project-files)和[生成資產](./glossary.md#generated-assets)的位置。它還將包含用於存放 Griptape Nodes [機密金鑰 (Secret Keys)](./glossary.md#secret-keys) 的 [.env](./glossary.md#.env) 檔案。

```
╭───────────────────────────────────────────────────────────────────╮
│ Workspace Directory                                               │
│     Select the workspace directory. This is the location where    │
│     Griptape Nodes will store your saved workflows.               │
│     You may enter a custom directory or press Return to accept    │
│     the default workspace directory                               │
│╰──────────────────────────────────────────────────────────────────╯
Workspace Directory (/Users/user/Documents/GriptapeNodes)
```

按下 Enter 鍵將使用預設路徑：`<current_working_directory>/GriptapeNodes`，其中 `<current_working_directory>` 是您運行 `gtn` 命令所在的目錄。或者，您也可以指定您偏好的任何自訂路徑。

**最後**，系統會提供幾個選用的設定步驟。每個步驟都可以透過接受預設值來略過，並且所有設定稍後都可以重新存取與修改：

- **儲存後端 (Storage backend)** — 在本機儲存靜態檔案（預設）或儲存在 Griptape Cloud 儲存貯體 (Bucket) 中。
- **Griptape Cloud 儲存貯體 (Bucket)** — 用於跨多台裝置同步工作流程和資產。
- **Hugging Face token** — 僅在您計劃從 Hugging Face Hub 下載受閘控限制 (Gated) 的模型時才需要。
- **額外節點庫 (Additional libraries)** — 可選安裝 Diffusers 和 Griptape Cloud 節點庫。

> 如果未來需要修改設定，您隨時可以使用 `gtn init` 命令重新進入此設定流程。

### 4. 啟動您的引擎 {: #4-start-your-engine }

安裝現已完成，您可以開始建立第一個工作流程或體驗範例工作流程。首先，在終端機中運行 `griptape-nodes`（或 `gtn`），然後返回瀏覽器。您在 [https://nodes.griptape.ai](https://nodes.griptape.ai) 的瀏覽器標籤頁將會更新並顯示 *Create from scratch*（從空白畫布開始），同時還會提供多個可供實驗與體驗的 Griptape Nodes 範例工作流程！

<!-- screenshot: the editor after the engine connects, showing Create from scratch and the sample workflows -->

## 透過授權啟用 {: #activating-with-a-license }

企業與組織可以使用授權金鑰 (License Key) 來運行 Griptape Nodes，而無需為每位成員註冊個別的 Griptape 帳號。如果您的組織管理員向您發送了授權金鑰，您無需註冊任何帳號：

1. 按照上述步驟安裝 [Griptape Nodes Desktop](#griptape-nodes-desktop-recommended)。
1. 在登入畫面上，點擊 **Activate with a License** 而非登入。
1. 貼上您的授權金鑰並完成啟用。

如果您的組織運行了地端私有化的[管理伺服器 (Admin Server)](enterprise/admin_server.md)，您還需要在啟用期間將應用程式指向該伺服器。有關授權啟用流程的完整逐步指南，請參閱[使用管理伺服器 (Using the Admin Server)](enterprise/using_the_admin_server.md)。

希望核發與管理授權金鑰的組織管理員應從[管理員儀表板 (Admin Dashboard)](enterprise/admin_dashboard.md)指南開始。有關任一部署模式中各組件如何協同作業的架構圖，請參閱[架構 (Architecture)](architecture.md)。

## 後續步驟 {: #next-steps }

接下來，讓我們開始學習如何在 Griptape Nodes 中實際進行創作與建構！[立即開始](tutorials/index.md)

如需移除 Griptape Nodes，請參閱[解除安裝 Griptape Nodes (Uninstalling Griptape Nodes)](uninstalling.md)。
