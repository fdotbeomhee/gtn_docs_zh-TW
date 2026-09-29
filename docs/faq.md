# 常見問題 (Frequently Asked Questions) {: #frequently-asked-questions }

!!! tip "提示"

    正在排查某個具體問題或錯誤？請參閱[疑難排解 (Troubleshooting)](troubleshooting.md)，瞭解常見問題及其解決方案。

## 我的工作區在哪裡（我的檔案儲存在哪裡）？ {: #where-is-my-workspace-where-do-my-files-save }

已儲存的工作流程等檔案均保存在工作區目錄 (Workspace Directory) 中。

工作區目錄的路徑可以在 Griptape Nodes Editor 中找到：

1. 開啟 Griptape Nodes Editor。
1. 開啟現有工作流程或建立空白工作流程。
1. 點擊 **Settings (設定)**。
1. 選擇 **Configuration Editor (設定編輯器)**。
1. 如果尚未選中，請點擊最左側欄的 **Griptape Nodes Settings**。
1. 具體路徑列於 **Workspace Directory** 下方。

如果您當前未運行編輯器，可以運行以下命令，它將回報您的工作區目錄：

```bash
gtn config show | grep workspace
```

## 我可以在與編輯器不同的電腦上運行引擎嗎？ {: #can-i-run-the-engine-on-a-different-machine-than-the-editor }

引擎 (Engine) 與編輯器 (Editor) 完全可以運行在彼此獨立的電腦上。請記住，您儲存的任何檔案或註冊的程式庫都將儲存在運行引擎的電腦上。因此，如果您在尋找檔案時未能立即找到，請仔細確認引擎當前是在哪台電腦上運行的。

## Griptape Nodes 安裝在哪裡？ {: #where-is-griptape-nodes-installed }

想知道 Griptape Nodes 的確切安裝位置？以下命令將為您顯示具體路徑：

=== "macOS / Linux"

    ```bash
    dirname $(dirname $(readlink -f $(which griptape-nodes)))
    ```

=== "Windows (PowerShell)"

    ```powershell
    Split-Path (Split-Path (Get-Command griptape-nodes).Source)
    ```

## 我可以查看或編輯設定檔嗎？ {: #can-i-see-or-edit-my-config-file }

要取得該檔案的路徑，請前往編輯器頂部的 Settings 功能表，然後選擇 **Copy Path to Settings**。這會將設定檔的路徑複製到您的剪貼簿中。

如果您偏好在命令列中操作，也可以使用：

```
gtn config show
```

## 初始設定後如何安裝 Advanced Media 程式庫？ {: #how-do-i-install-the-advanced-media-library-after-initial-setup }

有關安裝和管理節點程式庫（包括非 Advanced Media 程式庫）的完整指南，請參閱[節點程式庫 (Libraries)](guides/libraries.md)。

如果您在初始設定期間選擇了略過安裝 Advanced Media 程式庫，但現在希望添加它，可以透過運行以下命令進行安裝：

```bash
gtn init
```

這將重新啟動設定流程。您可以按 Enter 鍵保留現有的工作區和 Griptape Cloud API Key 設定。當系統出現以下提示時：

```
Register Advanced Media Library? [y/n] (n):
```

按下 **y** 安裝 Advanced Media 程式庫，或按 **n** 略過安裝。

!!! note "注意"

    Advanced Media 程式庫中的某些節點需要特定模型才能正常運行。您需要單獨安裝這些模型。

    請參閱各節點的說明文件以確定哪些節點需要哪些模型；每個文檔都包含具體依賴要求的連結。

## Advanced Media 程式庫中已廢棄的節點去哪裡了？ {: #what-happened-to-deprecated-nodes-from-the-advanced-media-library }

版本 0.64.0 移除了 Advanced Media 程式庫中已廢棄的節點。這些節點先前已被標記為廢棄，並已替換為更靈活的替代節點。

如果您的工作流程使用了已廢棄的節點，請參閱 [MIGRATION.md](https://github.com/griptape-ai/griptape-nodes/blob/main/MIGRATION.md) 遷移指南。該綜合指南提供了：

- 已移除節點及其對應替代節點的完整清單
- 逐步遷移指引
- 替代節點的視覺化範例
- 關於全新 Diffusion Pipeline Builder 系統的詳細資訊

遷移指南涵蓋了所有已廢棄的圖像處理、擴散管線 (Diffusion Pipeline)、放大 (Upscaling) 以及 LoRA 節點的替換方案。

## 如何解除安裝 Griptape Nodes？ {: #how-do-i-uninstall-griptape-nodes }

請參閱[解除安裝 Griptape Nodes (Uninstalling Griptape Nodes)](uninstalling.md)。

## 如何更新 Griptape Nodes？ {: #how-do-i-update-griptape-nodes }

Griptape Nodes 每次運行時都會自動檢查是否需要更新。如果需要，系統會提示您輸入 (y/n)。輸入 y 即會自動更新至引擎的最新版本。

如果您希望*手動*更新，可以隨時使用以下任一組命令：

```bash
griptape-nodes self update
griptape-nodes libraries sync
```

或者：

```bash
gtn self update
gtn libraries sync
```

## 在哪裡可以提供反饋或提問？ {: #where-can-i-provide-feedback-or-ask-questions }

您可以透過多種管道與我們取得聯繫：

- [官方網站](https://www.griptape.ai) - 造訪我們的首頁取得通用資訊
- [Discord](https://discord.com/invite/rpWWNmgGv9) - 加入我們的社群參與提問與討論
- [GitHub](https://github.com/griptape-ai/griptape-nodes) - 提交 Issue 或為程式碼庫做出貢獻

每個文檔頁面頁尾（右下方）的三個圖示也指向相同的連結。

## 如何體驗尚未發布的預覽功能？ {: #how-can-i-test-out-unreleased-features }

如果您有興趣測試尚未發布的功能，可以安裝 Griptape Nodes 的預先發布版本 (Pre-release builds)。

!!! warning "警告"

    預先發布版本無法保證穩定性，可能包含錯誤或尚未完成的功能。請自行承擔使用風險。

如果您使用 Griptape Nodes Desktop，請在[應用程式設定 (App Settings)](guides/desktop/app_settings.md#release-channels)中將 **Release Channel (發布通道)** 切換為 **Nightly**。Nightly 版本每日構建發布，隨應用程式一起內建了預覽版引擎和編輯器。您可以隨時切換回 **Stable (穩定版)**，無需重新安裝。

如果您是自行安裝引擎，預先發布版引擎每天會向 [latest](https://github.com/griptape-ai/griptape-nodes/releases/tag/latest) 標籤發布兩次。

若要切換至預先發布更新通道，請運行以下命令：

```
uv tool uninstall griptape-nodes
uv tool install git+https://github.com/griptape-ai/griptape-nodes.git@latest --reinstall --force --python 3.12
```

這將解除安裝當前版本的 Griptape Nodes，並從 GitHub 儲存庫安裝最新的預先發布版本。

!!! info "提示"

    使用 `uv tool uninstall griptape-nodes` 解除安裝不會刪除您現有的專案或設定。它僅會移除 Griptape Nodes 引擎本體。

您可以透過運行 `gtn self version` 來確認切換是否成功。版本號中應包含 Git Commit 的引用：

```
gtn self version
v0.31.0 (git - e172e80)
```

若要切換回穩定發布通道，請運行以下命令：

```
uv tool uninstall griptape-nodes
uv tool install griptape-nodes
```
