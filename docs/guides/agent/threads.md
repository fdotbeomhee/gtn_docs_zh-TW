# 對話串 (Threads) {: #threads }

與智慧體的每一次對話都被組織為獨立的**對話串 (Thread)**。對話串會以其建立時的日期與時間自動命名。

- 點擊 **+ New** 即可開啟一段全新的對話
- 先前的歷史對話串清單展示於訊息輸入方塊上方

## 對話串存放位置 {: #where-threads-are-stored }

對話串會自動持久化儲存至本機檔案系統中，跨工作階段皆可完整存取。每個對話串由三個檔案組成：

| 檔案名稱 | 內容說明 |
| ----------------------- | -------------------------------------------- |
| `thread_{id}.json` | 完整的歷史訊息記錄內容 |
| `thread_{id}.meta.json` | 對話串標題、時間戳記、訊息總數 |
| `thread_{id}.runs.json` | 每次運行的服務供應商、AI 模型與 MCP 伺服器記錄 |

Runs 檔案詳細記錄了該對話串中每次生成回應時所採用的 AI 供應商、模型規格與 MCP 伺服器狀態。

實體存放路徑嚴格遵循 [XDG 基本目錄規格 (XDG Base Directory)](https://specifications.freedesktop.org/basedir-spec/latest/)：

| 作業系統平臺 | 預設儲存路徑 |
| ------------- | ---------------------------------------------------- |
| macOS / Linux | `~/.local/share/griptape_nodes/threads/` |
| Windows | `%USERPROFILE%\.local\share\griptape_nodes\threads\` |

Griptape Nodes Desktop 桌面版則將對話串集中存放在其應用程式專屬資料夾內的 `xdg_data_home/griptape_nodes/threads/` 目錄中。前往 **App Settings → App Data Locations** 即可在您的電腦上快速定位該資料夾。

若歷史記錄檔案意外受損損毀，Griptape Nodes 會自動將其隔離移置一旁（重新命名追加 `.corrupt-<timestamp>` 後綴），確保其餘正常對話串完全不受影響。
