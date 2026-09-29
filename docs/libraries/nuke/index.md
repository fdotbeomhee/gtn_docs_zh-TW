# Nuke 程式庫 {: #nuke-library }

建構可在 [Foundry Nuke](https://www.foundry.com/products/nuke-family) 專業數位合成軟體內部執行的 AI 工作流程。
本程式庫提供了針對 Nuke 流程控制的專屬節點、可將工作流程打包為帶版本號 `.gizmo` 工具組以供 Nuke 直接調用的發布工具，以及能透過無介面模式 (`nuke -t`) 執行現有 `.nk` 腳本並將其附帶註解的節點映射為 Griptape 畫布型別連接埠的 **Nuke Script** 節點。

- **原始碼存放庫**：[griptape-ai/griptape-nodes-library-nuke](https://github.com/griptape-ai/griptape-nodes-library-nuke)
- **環境需求**：本機已安裝 Foundry Nuke 並具備合法授權
- **節點分類目錄**：節點挑選器中的 `Foundry Nuke` 目錄

## 安裝指南 {: #installation }

在編輯器中，依序開啟 **管理 (Manage) → 程式庫管理 (Library Management)**，點擊 **新增程式庫 (Add Library)** 並貼入以下網址：

```text
https://github.com/griptape-ai/griptape-nodes-library-nuke
```

或透過 CLI 命令列執行安裝：

```bash
gtn libraries download https://github.com/griptape-ai/griptape-nodes-library-nuke
```

有關通用的安裝、更新與除錯指引，請參閱 [程式庫指南](../../guides/libraries.md)。

## 節點索引參考 {: #nodes }

| 節點名稱 | 功能說明 |
| :--- | :--- |
| [Nuke Start Flow](nuke_start_flow.md) | 針對 Nuke 工作流程的專屬入口節點——將工作流程發布為 Gizmo 的必備前提 |
| [Nuke End Flow](nuke_end_flow.md) | 針對 Nuke 工作流程的終點節點；輸出 `was_successful` 狀態與 `result_details` 詳細資訊 |
| [Nuke Script](nuke_script.md) | 透過 `nuke -t` 在背景無介面執行 `.nk` 腳本，將標註的 Read/Write 節點發布為型別輸入/輸出端 |

## 配置 Nuke 本機安裝路徑 {: #configuring-nuke-installations }

### 自動掃描辨識 (Auto-discovery) {: #auto-discovery }

本程式庫在引擎啟動時會自動掃描標準預設路徑，並自動填寫 **Nuke Version** 下拉選單：

| 作業系統 | 預設掃描目錄 |
| :--- | :--- |
| macOS | `/Applications/Nuke*` |
| Windows | `%ProgramFiles%\Nuke*`、`%ProgramFiles(x86)%\Nuke*` |
| Linux | `/usr/local/Nuke*`、`/opt/Nuke*`、`~/Nuke*` |

系統同時會掃描環境變數 `PATH` 中名為 `Nuke` 或 `nuke` 的執行檔。安裝新版本後，點擊節點介面上的 **Refresh UI** 即可重新觸發掃描。

### 透過引擎設定 (Engine Settings) 手動配置 {: #manual-configuration-via-engine-settings }

若 Nuke 安裝在非標準目錄下，可於引擎設定的 `nuke.installations` 鍵下新增項目：

```json
{
  "nuke": {
    "installations": [
      {
        "display_name": "Nuke 16.0v7",
        "executable_path": "/opt/nuke/16.0v7/Nuke16.0",
        "annotator_nuke_version": 16
      }
    ]
  }
}
```

| 欄位名稱 | 必填 | 說明備註 |
| :--- | :--- | :--- |
| `display_name` | 是 | 顯示於 **Nuke Version** 下拉選單中的標籤名稱 |
| `executable_path` | 是 | Nuke 二進位執行檔的本機絕對路徑 |
| `annotator_nuke_version` | 否 | Nuke 主版本號；決定載入哪個版本的 Annotator 面板（預設值：`16`） |
| `env_overrides` | 否 | 合併注入至 Nuke 子處理程序的額外環境變數 |
| `notes` | 否 | 自由備忘註記；不影響執行階段運算 |

### 附加引擎全域設定 {: #additional-engine-settings }

| 設定鍵 | 功能說明 |
| :--- | :--- |
| `nuke.executable` | 當未選取特定安裝版本時所採用的備用 Nuke 執行檔路徑 |
| `nuke.env` | 自動注入至所有 Nuke 子處理程序中的全域環境變數 |
| `nuke.nuke_path` | 在每次調用時自動附加至 `NUKE_PATH` 的額外目錄清單 |

### Foundry 商業授權設定 {: #foundry-license }

請在 Griptape 憑據管理面板 (Secrets Panel) 中設定 `foundry_LICENSE`。該授權會自動安全注入至 Nuke 子處理程序中——請勿在 `env_overrides` 中以明文寫入授權金鑰。

## 將工作流程發布為 Nuke Gizmo 工具組 {: #publishing-a-workflow-as-a-nuke-gizmo }

1. 建構一個最上層以 [Nuke Start Flow](nuke_start_flow.md) 為起點、以 [Nuke End Flow](nuke_end_flow.md) 為終點的工作流程。
1. 觸發 **發布工作流程 (Publish Workflow)** 操作（請參閱 [工作流程發布指南](../../guides/publishing.md)）。在對話框中選擇：
    - **Nuke 安裝版本**（自動偵測）以解析外掛路徑候選項。
    - **Gizmo 安裝路徑**——可選 `~/.nuke`、`NUKE_PATH` 包含的路徑、Nuke 安裝目錄下的 plugins 資料夾或自訂路徑。
    - **更新模式**——選擇建立全新版本還是直接覆寫現有版本。
1. 在 Nuke 軟體介面中，透過節點工具列上的 `Griptape` 選單即可新增此 Gizmo；或在發布後執行頂部功能表列的 `Griptape > Refresh Griptape Gizmos`，無需重新啟動 Nuke 即可立即加載新版本。

發布操作會在選定的目錄下寫入帶版本號的 `.gizmo` 與執行腳本，並產生一個向 Nuke 節點工具列掛載 `Griptape` 子選單的 `menu.py`。同一工作流程的多個版本會自動歸整於專屬子選單下。工作流程的輸出檔案將自動儲存於目前 `.nk` 專案檔旁（位於 `griptape_outputs/<workflow_name>/...` 目錄下）。

## 支援與反饋 {: #support }

若在運作中發現程式錯誤或有全新功能建議，歡迎隨時向社群回報：
[提交 GitHub Issue](https://github.com/griptape-ai/griptape-nodes-library-nuke/issues)。
