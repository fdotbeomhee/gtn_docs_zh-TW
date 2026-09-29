# 解除安裝 Griptape Nodes {: #uninstalling-griptape-nodes }

請先從與您當初安裝方式相符合的小節開始：

- **[Griptape Nodes Desktop](#griptape-nodes-desktop)** — 從 [griptapenodes.com](https://griptapenodes.com) 下載的桌面應用程式。
- **[手動安裝引擎](#manual-engine-install)** — 透過 `uv tool install griptape-nodes` 自行安裝的引擎。

然後閱讀[殘留內容清理](#whats-left-behind)以完成整個解除安裝流程。

!!! warning "工作流程不會被自動刪除"

    任何解除安裝程式都不會碰觸您的工作區目錄 (Workspace Directory)，因此您的專案 (Projects)、工作流程 (Workflows) 和生成資產 (Generated Assets) 都會一直保存在磁碟上，直到您手動將其刪除。在進行到[刪除工作區目錄](#remove-your-workspace)之前，請先將您希望保留的任何內容複製備份到安全位置。

## Griptape Nodes Desktop {: #griptape-nodes-desktop }

該應用程式內建了引擎，並在其專屬的資料資料夾中管理引擎的各個目錄。

### macOS {: #macos }

1. 結束 Griptape Nodes 應用程式。
1. 在 Finder 中開啟 **Applications (應用程式)** 資料夾，將 **Griptape Nodes** 拖至垃圾桶。
1. 清空垃圾桶。

此操作會將資料資料夾留在原處。請參閱[刪除應用程式資料](#remove-the-applications-data-macos-and-linux)。

### Windows {: #windows }

1. 結束 Griptape Nodes 應用程式。
1. 開啟 **設定 → 應用程式 → 安裝的應用程式 (Settings → Apps → Installed apps)**。
1. 找到 **Griptape Nodes**，點擊其右側的 **...** 功能表，然後選擇 **解除安裝 (Uninstall)**。

Windows 解除安裝程式會移除：

- 位於 `%LOCALAPPDATA%\ai.griptape.nodes.desktop` 的應用程式本體，包括存放在其旁邊的任何已下載更新套件。
- 開始功能表與桌面捷徑。
- **安裝的應用程式** 列表中的項目。
- 應用程式的兩個資料資料夾：`%APPDATA%\Griptape Nodes` 與 `%LOCALAPPDATA%\Griptape Nodes`。這涵蓋了應用程式的設定、記錄檔 (Logs) 以及引擎寫入的所有內容（包含您的 API 金鑰）。

接著請繼續閱讀[殘留內容清理](#whats-left-behind)。

### Linux {: #linux }

Griptape Nodes Desktop 以 AppImage 格式發行：這是一個獨立的單一檔案，不會在特定位置進行實體安裝。

1. 結束 Griptape Nodes 應用程式。
1. 刪除您下載的 `.AppImage` 檔案。
1. 如果您使用 AppImageLauncher 等輔助工具註冊過該 AppImage，請改用該工具進行移除，以同時清理位於 `~/.local/share/applications` 下的啟動器項目。

接著請繼續閱讀[刪除應用程式資料](#remove-the-applications-data-macos-and-linux)。

### 刪除應用程式資料 (macOS 和 Linux) {: #remove-the-applications-data-macos-and-linux }

在 macOS 和 Linux 上，刪除應用程式本體後仍會留下其資料。請手動刪除這些資料夾（在 Windows 上，解除安裝程式已自動移除了對應內容）：

| 內容                       | macOS                                                 | Linux                           |
| -------------------------- | ----------------------------------------------------- | ------------------------------- |
| 應用程式資料 (Application data) | `~/Library/Application Support/Griptape Nodes`        | `~/.config/Griptape Nodes`      |
| 記錄檔 (Log files)         | `~/Library/Logs/Griptape Nodes`                       | `~/.config/Griptape Nodes/logs` |
| 已下載的更新套件           | `~/Library/Caches/velopack/ai.griptape.nodes.desktop` | —                               |

應用程式資料資料夾包含應用程式設定、登入工作階段 (Session)、已啟用的任何授權金鑰 (License Keys)，以及引擎寫入的檔案：`griptape_nodes_config.json`、包含 API 金鑰與其他機密資訊的 `.env` 檔案、智慧體對話執行緒、模型下載記錄，以及引擎為影片預覽下載的 ffmpeg 二進位檔案。您的節點程式庫 (Node Libraries) 不在此處，它們位於工作區目錄中，將在後續步驟[單獨處理](#remove-your-workspace)。在 Linux 上，更新是在暫存目錄中暫存的，因此無需清理更新快取。

!!! tip "從應用程式內部查找精確路徑"

    在解除安裝之前，開啟 **App Settings (應用程式設定)** 並滾動至 [App Data Locations (應用程式資料位置)](guides/desktop/app_settings.md#app-data-locations)。此處會顯示您這台電腦的應用程式資料資料夾路徑，並提供 **Open (開啟)** 按鈕可在 Finder 或檔案管理器中直接開啟。記錄檔與更新套件未在該處列出，因此請參考上表中的路徑。

接著請繼續閱讀[殘留內容清理](#whats-left-behind)。

## 手動安裝引擎 {: #manual-engine-install }

運行官方解除安裝命令：

```bash
gtn self uninstall
```

當終端機顯示 `When done, press Enter to exit.` 時，按下 Enter 鍵以完成執行檔的清除。該命令會移除：

- 引擎的**設定**與**資料**目錄：`~/.config/griptape_nodes` 與 `~/.local/share/griptape_nodes`，包含存放 API 金鑰與其他機密資訊的 `.env` 檔案。
- 透過運行 `uv tool uninstall griptape-nodes` 移除 `griptape-nodes` 和 `gtn` 命令。

如果終端機輸出了 **Caveats (注意事項)** 區塊，請務必詳細閱讀。其中列出了未被碰觸的設定檔案（位於工作區與專案資料夾內部），以及任何無法自動刪除、需要您手動清理的項目。

!!! note "Windows 上的資料夾位置"

    引擎在所有作業系統平台上採用相同的目錄配置結構，因此在 Windows 上，這兩個資料夾分別是 `%USERPROFILE%\.config\griptape_nodes` 和 `%USERPROFILE%\.local\share\griptape_nodes`。兩者都位於您的使用者根目錄下，而不是在 `AppData` 中。

如果 `gtn` 命令無法正常運行（通常是因為之前的解除安裝操作留下了損壞的虛擬環境），請手動執行相同的清除步驟：

```bash
uv tool uninstall griptape-nodes
rm -rf ~/.config/griptape_nodes ~/.local/share/griptape_nodes
```

在 Windows 上使用 PowerShell：

```powershell
uv tool uninstall griptape-nodes
Remove-Item -Recurse -Force "$env:USERPROFILE\.config\griptape_nodes"
Remove-Item -Recurse -Force "$env:USERPROFILE\.local\share\griptape_nodes"
```

接著請繼續閱讀[殘留內容清理](#whats-left-behind)。

## 殘留內容清理 {: #whats-left-behind }

### 刪除工作區目錄 {: #remove-your-workspace }

您的[工作區目錄 (Workspace Directory)](guides/projects/workspace.md)存放了專案 (Projects)、工作流程 (Workflows)、生成資產 (Generated Assets) 以及節點程式庫 (Node Libraries)，系統絕不會自動刪除它。預設路徑為：

- 如果使用桌面應用程式：`Documents/GriptapeNodes`。
- 如果自行安裝引擎：首次運行 `gtn` 指令所在目錄下的 `GriptapeNodes` 資料夾。

您當初可能已將其設定在其他位置，因此在解除安裝前請先記下實際路徑。

請複製出您希望保留的任何檔案，然後手動刪除該資料夾。

### 刪除已下載的模型 {: #remove-downloaded-models }

節點從 Hugging Face Hub 下載的模型會存放在共享的 Hugging Face 快取目錄中，而不是 Griptape Nodes 的專屬目錄。

| 平台          | 路徑                                   |
| ------------- | -------------------------------------- |
| macOS / Linux | `~/.cache/huggingface/hub`             |
| Windows       | `%USERPROFILE%\.cache\huggingface\hub` |

如果您的電腦上還有其他工具也會從 Hugging Face 下載模型，請不要直接刪除整個資料夾。對於手動安裝的引擎，您可以在解除安裝前僅清除 Griptape Nodes 所下載的模型：

```bash
gtn models list
gtn models delete <model_id>
```

在 Griptape Nodes 尚未解除安裝前，您也可以從編輯器的 **Model Management (模型管理)** 視窗中執行此操作。請參閱[管理模型與程式庫 (Managing models and libraries)](guides/editor/managing_models_and_libraries.md)。

### 移除 uv {: #remove-uv }

Griptape Nodes 使用 [uv](https://docs.astral.sh/uv/) 為各個節點程式庫構建虛擬環境，而 uv 會維護自己的快取與 Python 直譯器。如果 uv 僅僅是因為安裝 Griptape Nodes 才被安裝到您的電腦上，請按照 uv 的[官方解除安裝說明](https://docs.astral.sh/uv/getting-started/installation/#uninstallation)將 uv 本體及其快取一併移除。

這些說明不會自動修改您的 `PATH` 環境變數。uv 的安裝程式曾將其執行檔目錄（macOS/Linux 為 `~/.local/bin`，Windows 為 `%USERPROFILE%\.local\bin`）添加到了 PATH 中。在 uv 移除後，您可以刪除該項目：在 macOS 和 Linux 上刪除 uv 添加到 shell profile 中的設定行，在 Windows 上則可以在 **設定 → 系統 → 關於 → 進階系統設定 → 環境變數** 中將其移除。

## 重新安裝 {: #reinstalling }

如果未來需要重新使用，請再次參考[安裝 Griptape Nodes (Installing Griptape Nodes)](installation.md)進行安裝。如果您保留了原來的工作區目錄，只需將新安裝指向該目錄，您的專案與工作流程就會完好如初地呈現在您眼前。
