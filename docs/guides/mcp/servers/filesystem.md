# Filesystem MCP 伺服器 (Filesystem MCP Server) {: #filesystem-mcp-server }

**[Filesystem MCP 伺服器](https://github.com/modelcontextprotocol/servers/blob/main/src/filesystem/README.md)** 賦予 AI Agent 在您本機電腦上執行檔案與目錄操作的能力。它對指定授權白名單目錄提供受控、嚴格限制存取邊界的安全機制，專為自動化檔案管理、內容歸檔整理與資料批次處理而設計。

## 安裝與配置方式 {: #installation }

1. **開啟 Griptape Nodes** 並前往 **Settings** → **MCP Servers**

1. **點擊 + New MCP Server**

1. **配置伺服器連線參數**：

    - **Server Name/ID (伺服器名稱/識別碼)**：`filesystem`
    - **Connection Type (連線類型)**：`Local Process (stdio)`
    - **Configuration JSON (組態 JSON)**：

    ```json
    {
      "transport": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/path/to/allowed/directory"
      ],
      "env": {},
      "encoding": "utf-8",
      "encoding_error_handler": "strict"
    }
    ```

1. **點擊 Create Server** 完成建立

## 內建可用工具一覽 {: #available-tools }

- **`read_file`** — 讀取指定檔案的全文內容
- **`write_file`** — 將文字或數據寫入檔案
- **`list_directory`** — 列舉資料夾目錄清單與結構
- **`create_directory`** — 建立新的子資料夾目錄
- **`search_files`** — 依檔名或萬用字元正規化模式搜尋檔案
- **`move_file`** — 移動或重新命名目標檔案
- **`delete_file`** — 刪除指定檔案

## 進階配置選項 {: #configuration-options }

若欲一次向 Agent 開放多個目錄的存取授權，可將各目錄路徑作為獨立引數逐一追加至 `args` 陣列中：

```json
{
  "transport": "stdio",
  "command": "npx",
  "args": [
    "-y",
    "@modelcontextprotocol/server-filesystem",
    "/Users/username/Desktop",
    "/Users/username/Downloads",
    "/Users/username/Documents"
  ],
  "env": {},
  "encoding": "utf-8",
  "encoding_error_handler": "strict"
}
```

## 相關參考資源 {: #resources }

- [Filesystem MCP 伺服器原始碼倉庫](https://github.com/modelcontextprotocol/servers/blob/main/src/filesystem/README.md) — 官方標準存放庫與操作手冊
- [Node.js 檔案系統 (fs) API 文件](https://nodejs.org/api/fs.html) — 深入了解底層檔案系統操作規範
