# Griptape Nodes 命令列介面 (CLI) {: #griptape-nodes-command-line-interface-cli }

如果您剛接觸命令列介面 (CLI)，CLI 是一種透過鍵入指令而非在使用者介面 (UI) 中點擊按鈕來與軟體互動的純文字方式。Griptape Nodes 提供了帶有 `griptape-nodes`（或其簡寫別名 `gtn`）指令的 CLI，讓您能夠在終端機中與 Griptape Nodes 進行互動。

`griptape-nodes`（或簡寫 `gtn`）是一項專為在您的電腦上啟動與管理 Griptape Nodes Engine 安裝而設計的命令列工具。該工具負責處理初始化工作區、管理組態設定以及啟動驅動網頁版 Griptape Nodes 編輯器的引擎等任務。工作流程的實際建立與編輯則是在執行引擎時開啟的網頁介面中進行。

## 基本用法 {: #basic-usage }

```
griptape-nodes [options] [COMMAND]
```

若未指定任何指令，該工具預設執行 `engine` 指令。

## 指令清單 (Commands) {: #commands }

### `engine`（預設指令） {: #engine-default-command }

執行 Griptape Nodes 引擎。

```
griptape-nodes engine
```

這將啟動 Griptape Nodes 引擎並在 https://nodes.griptape.ai 開啟網頁介面。

### `init` {: #init }

為 Griptape Nodes 初始化新工作區：設定您的 API 金鑰、工作區目錄、儲存後端以及（選填）額外的程式庫。再次執行將重新執行相同的提示問答，以便您日後隨時更改上述任何設定。

```
griptape-nodes init [options]
```

選項 (Options)：

- `--api-key` - 直接指定您的 Griptape API 金鑰，無需互動式提示
- `--workspace-directory` - 直接指定您的工作區目錄，無需互動式提示
- `--storage-backend` - 直接設定儲存後端（`local` 或 `gtc`），無需互動式提示
- `--bucket-name` - 設定當 `--storage-backend gtc` 時所使用的儲存貯體 (Bucket) 名稱（既有或新增）
- `--register-diffusers-library` / `--no-register-diffusers-library` - 安裝（或略過）Griptape Nodes Diffusers Library
- `--register-griptape-cloud-library` / `--no-register-griptape-cloud-library` - 安裝（或略過）Griptape Cloud Library
- `--no-interactive` - 以無互動提示模式執行 init，僅使用提供的旗標與預設值
- `--hf-token` - 設定用於下載受限存取 (Gated) 模型的 Hugging Face 權杖
- `--config key=value` - 設定任意組態鍵值；可重複此旗標以設定多組（例如 `--config log_level=DEBUG --config workspace_directory=/tmp`）
- `--secret key=value` - 設定任意機密密鑰；可重複此旗標以設定多組（例如 `--secret MY_API_KEY=abc123`）

### `config` {: #config }

管理您的 Griptape Nodes 組態設定。

```
griptape-nodes config SUBCOMMAND
```

子指令 (Subcommands)：

- `show [config_path]` - 顯示目前的組態。未帶參數時，以 JSON 格式印出完整合併後的組態；帶入點分隔路徑（例如 `workspace_directory`）時，僅印出該設定值
- `list` - 依優先權順序列出所有影響您組態的組態檔案
- `reset` - 將您的組態重設為預設值

有關可透過此方式讀取或設定的完整選項清單，請參閱 [組態設定參考手冊 (Configuration Reference)](configuration_reference.md)。

### `self` {: #self }

管理 CLI 安裝本身。

```
griptape-nodes self SUBCOMMAND
```

子指令 (Subcommands)：

- `uninstall` - 解除安裝 CLI，移除其組態目錄、資料目錄與已安裝的執行檔
- `version` - 顯示 CLI 的目前版本
- `info` - 印出用於除錯的系統資訊報告：引擎版本與安裝來源、平台與 Python 詳細資訊、組態路徑、包含各層檔案與任何剖析錯誤的組態分層堆疊、生效中的設定值、設定了哪些 API 金鑰、每個已註冊程式庫及其版本，以及任何專案範本。可安全地貼至錯誤回報中——API 金鑰與其他憑證會被移除，家目錄路徑會替換為 `~`，您的使用者名稱會替換為 `<user>`。結尾會印出被移除項目的計數，因此看似為空的設定值可以與被隱藏的設定值區分開來
    - `env` 與 `runtime` 分層沒有獨立的實體檔案：`env` 列出所設定的 `GTN_CONFIG_*` 變數名稱（值已省略，因為下方合併後的組態已顯示各變數解析後的結果），而 `runtime` 則顯示由活動專案所鎖定的工作區目錄
    - `--show-identity` - 保留真實的家目錄路徑與您的使用者名稱，而非替換為 `~` 與 `<user>`。憑證依然會被遮蔽移除
    - 若要將相同的資訊**以及**您的日誌檔收集為單一可分享檔案，請使用 [`diagnostics collect`](#diagnostics)

### `libraries` {: #libraries }

管理本機程式庫。有關安裝與管理程式庫的完整指南，請參閱 [程式庫指南 (Libraries)](../guides/libraries.md)。

```
griptape-nodes libraries SUBCOMMAND
```

子指令 (Subcommands)：

- `sync` - 將每個已註冊的程式庫更新至最新版本
    - `--overwrite` - 在更新程式庫之前，捨棄該程式庫本機複本中任何未提交的變更
- `download <git_url>` - 自 Git 複製 (Clone) 程式庫並進行註冊
    - `--branch` - 要簽出的分支、標籤 (Tag) 或提交 (Commit)
    - `--target-dir` - 複製目的地的目錄名稱
    - `--download-dir` - 程式庫複製存放的父目錄
    - `--overwrite` - 若程式庫目錄已存在則直接覆寫

### `models` {: #models }

管理從 Hugging Face Hub 下載的 AI 模型——當您執行工作流程時，本機擴散或大型語言模型等節點即是從此處提取模型。

```
griptape-nodes models SUBCOMMAND
```

子指令 (Subcommands)：

- `download <model_id>` - 從 Hugging Face Hub 下載模型（例如 `microsoft/DialoGPT-medium`）
    - `--local-dir` - 下載模型的本機目錄
    - `--revision` - 要下載的 Git 修訂版本（預設為 `main`）
- `list` - 列出目前位於您本機快取中的所有模型檔案及其磁碟大小
- `delete <model_id>` - 從您的本機快取中刪除某個模型的檔案
- `search [query]` - 在 Hugging Face Hub 上搜尋模型
    - `--task` - 依任務類型篩選結果（例如 `text-generation`）
    - `--limit` - 回傳結果的最大數量（預設為 20，上限為 100）
    - `--sort` - 結果排序欄位（預設為 `downloads`）
    - `--direction` - 排序方向（預設為 `desc`）
- `downloads status [model_id]` - 顯示單一模型的下載進度/狀態；若未提供則顯示所有追蹤中的模型狀態
- `downloads list` - 列出所有追蹤中的模型下載項目及其狀態
- `downloads delete <model_id>` - 刪除模型的下載狀態追蹤記錄（不會刪除模型實體檔案；刪除檔案請使用 `models delete`）

### `doctor` {: #doctor }

檢查您的 Griptape Nodes 安裝狀況並印出檢查結果表格，若有需要修復之處會隨附修復指引。

```
griptape-nodes doctor
```

每項檢查的結果分為 **PASS**（通過）、**WARN**（警告：目前可用，但日後可能引發問題）或 **FAIL**（失敗：目前已有元件損壞）。檢查項目包含：

- **Workspace** - 檢查您的工作區目錄是否存在且可寫入
- **Disk Space** - 檢查工作區所在磁碟是否有足夠空間儲存工作流程與安裝程式庫
- **Libraries** - 檢查每個已註冊的程式庫是否成功載入，且未遺漏任何節點
- **Secrets** - 檢查程式庫所要求的每個 API 金鑰是否皆已設定數值
- **Log Capture** - 檢查引擎日誌是否正常寫入，以便日後收集診斷
- **Cloud Connection** - 檢查本機是否能連線至 Griptape Cloud（編輯器與引擎通訊所需）

當有檢查項目**失敗 (FAIL)** 時，指令會以非零狀態碼退出，因此可安全地用於自動化指令碼中。單純的警告狀態碼為 0。

相同的結果會以 `doctor.json` 形式儲存於診斷套件中，使閱讀您錯誤回報的人員能精確重現您所看到的資訊。

### `diagnostics` {: #diagnostics }

將疑難排解此安裝所需的所有資訊打包為單一 zip 壓縮檔，方便您附加至錯誤回報中。

```
griptape-nodes diagnostics SUBCOMMAND
```

子指令 (Subcommands)：

- `collect` - 產生診斷套件：包含您的日誌檔、生效中的設定值、每個程式庫與專案及其載入狀況，以及來自 [`doctor`](#doctor) 的健全度檢查
    - `--output`, `-o` - 診斷套件寫入路徑。若指定目錄則會自動產生檔名；預設為當前目錄
    - `--skip-logs` - 排除引擎日誌檔不納入診斷套件
    - `--skip-libraries` - 略過預先載入程式庫。速度較快，但診斷套件無法指出哪些程式庫載入失敗
    - `--show-identity` - 保留真實的家目錄路徑與您的使用者名稱，而非替換為 `~` 與 `<user>`

此指令不會將任何資料上傳至任何地方。檔案僅寫入至您的本機電腦，由您全權決定是否分享。

診斷套件由引擎產生，引擎知曉其自有的 API 金鑰，因此會從收集的每個檔案中自動搜尋並移除金鑰——以及任何形似憑證的機密資訊。套件內部的 `manifest.json` 會統計每次移除的計數，而 `README.md` 則以平實的文字解說各個檔案的用途。

有關編輯器與桌面應用程式的對應操作，請參閱 [匯出引擎日誌 (Exporting engine logs)](../troubleshooting.md#exporting-engine-logs)。

## 組態設定 (Configuration) {: #configuration }

Griptape Nodes 將其組態設定儲存於下列位置：

- 組態目錄：`~/.config/griptape_nodes` (macOS/Linux) 或 `%USERPROFILE%\.config\griptape_nodes` (Windows)
- 資料目錄：`~/.local/share/griptape_nodes` (macOS/Linux) 或 `%USERPROFILE%\.local\share\griptape_nodes` (Windows)
- 組態檔案：位於組態目錄下的 `griptape_nodes_config.json`
- 環境變數檔：位於組態目錄下的 `.env`

Griptape Nodes Desktop 將這兩個目錄保存在其專屬的應用程式資料資料夾中，因此不會與手動安裝的引擎發生衝突。詳細路徑請參閱 [解除安裝 Griptape Nodes (Uninstalling Griptape Nodes)](../uninstalling.md#griptape-nodes-desktop)。

## 典型工作流程 (Workflow) {: #workflow }

標準使用流程如下：

1. 執行 `griptape-nodes init` 設定您的工作區與 API 金鑰
1. 執行 `griptape-nodes` 啟動引擎
1. 使用開啟的網頁介面建立與管理您的節點工作流程
