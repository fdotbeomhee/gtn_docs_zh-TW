# 本機進程 (stdio) 連線 {: #local-process-stdio-connection }

**stdio** 連線類型允許 Griptape Nodes 透過標準輸入與標準輸出串流 (stdin / stdout)，與作為本機進程執行的 MCP 伺服器進行直接高效通訊。

## 何時使用 stdio {: #when-to-use-stdio }

- **本機應用程式**：在同一部電腦上執行本機 MCP 伺服器
- **命令列工具**：與以 CLI 為基礎的 MCP 伺服器工具互動
- **開發除錯**：日常測試與客製化擴充套件開發
- **極簡配置**：追求最少組態設定與最低依賴負擔

## stdio MCP 伺服器精選實例 {: #example-stdio-mcp-servers }

以下列舉幾款代表性伺服器。社群中還有更多強大的 MCP 伺服器可供設定，此處提供數個最容易上手的起點：

- **[Fetch](../servers/fetch.md)** — 網頁內容抓取與解析
- **[Filesystem](../servers/filesystem.md)** — 本機檔案與目錄操作管理

## 設定參數說明 {: #configuration }

### 必要欄位 {: #required-fields }

| 欄位名稱 | 型別 | 描述說明 | 範例 |
| :--- | :--- | :--- | :--- |
| `command` | string | 用於啟動 MCP 伺服器的執行命令 | `"npx"`, `"python"`, `"uvx"` |
| `args` | array | 傳遞給命令的參數陣列 | `["-y", "@modelcontextprotocol/server-memory"]` |

### 選填欄位 {: #optional-fields }

| 欄位名稱 | 型別 | 描述說明 | 預設值 |
| :--- | :--- | :--- | :--- |
| `env` | object | 專屬環境變數鍵值對 | `{}` |
| `cwd` | string | 進程工作目錄實體路徑 | 當前目錄 |
| `encoding` | string | 文字字元編碼 | `"utf-8"` |
| `encoding_error_handler` | string | 編碼例外錯誤處理原則 | `"strict"` |

## 組態範例實例 {: #example-configurations }

### Memory 記憶體儲存伺服器 (Node.js) {: #memory-server-nodejs }

```json
{
  "name": "memory",
  "transport": "stdio",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-memory"],
  "description": "為對話提供跨工作階段持久記憶儲存"
}
```

### Filesystem 檔案系統伺服器 {: #filesystem-server }

```json
{
  "name": "filesystem",
  "transport": "stdio",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "/allowed/path"],
  "env": {
    "NODE_ENV": "production"
  },
  "cwd": "/home/user/projects"
}
```

## 建置步驟指引 {: #setup-steps }

### 1. 安裝 MCP 伺服器 {: #1-install-mcp-server }

```bash
# Node.js 伺服器
npm install -g @modelcontextprotocol/server-memory

# Python 伺服器
pip install mcp-server-git
# 或採用免安裝直接執行
uvx mcp-server-git
```

### 2. 在 Griptape Nodes 中配置 {: #2-configure-in-griptape-nodes }

1. 開啟 Griptape Nodes 設定視窗
1. 前往 **MCP Servers** 設定頁籤
1. 點擊新增伺服器並選取 `stdio` 傳輸模式
1. 填入啟動 `command` 與對應的 `args` 參數
1. 點擊儲存該連線

### 3. 在工作流程中調用 {: #3-use-in-workflow }

1. 拖曳一個 **MCPTask** 節點至畫布中
1. 在伺服器選單中指定方才建立的 stdio 伺服器
1. 輸入提示詞 (Prompt)
1. 點擊執行工作流程

## 主要優勢 {: #advantages }

- **極低延遲**：本機進程直接管道通訊，近乎零延遲
- **部署極簡**：無需維護外部網路連接埠或連線端點
- **本機完全掌控**：能完整管理與約束進程的生命週期與資源
- **資源佔用輕量**：無額外網路協定堆疊轉換開銷

## 使用限制 {: #limitations }

- **僅限本機**：無法直接跨網路連線至遠端主機
- **進程生命週期**：需確保本機環境具備相應的執行階段 (Node.js/Python 等)
- **平台相依性**：不同作業系統 (Windows / macOS / Linux) 的路徑與命令語法略有差異
- **單一連線個體**：一個進程實例通常對應單一通訊管道

## 疑難排解 {: #troubleshooting }

### 伺服器無法啟動 {: #server-wont-start }

- 確認命令可執行檔已正確加入作業系統的 `PATH` 環境變數
- 檢查檔案執行權限
- 確認伺服器所需的所有執行階段依賴套件皆已安裝齊全
- 先行在終端機中手動執行該命令以排除腳本本身語法錯誤

### 連線逾時 (Connection Timeout) {: #connection-timeout }

- 確認伺服器是否能正常回應 stdio 標準輸入輸出訊號
- 檢查文字編碼設定是否正確匹配
- 查閱伺服器是否有向 stderr 輸出錯誤記錄
- 檢查工作目錄 (`cwd`) 的存取權限

### 權限錯誤 (Permission Errors) {: #permission-errors }

- 確認作業系統檔案系統具有相應目錄的讀寫權限
- 確認目前使用者身分有權存取目標資源路徑
- 確認環境變數的授權範圍無阻礙

## 最佳實踐準則 {: #best-practices }

1. **優先使用絕對路徑**：命令與工作目錄皆建議採用絕對路徑以避免路徑解析分歧
1. **善用環境變數**：機密金鑰或動態參數應透過 `env` 欄位傳遞而非寫死於參數列
1. **完善例外處理**：為可能發生的執行階段錯誤設定明確規則
1. **監控系統資源**：留意長時間執行進程是否存在記憶體流失或高 CPU 負載問題
1. **預先本機實測**：在填入 Griptape Nodes 前先於終端機命令列獨立驗證指令

## 相關參考資源 {: #next-steps }

- [SSE 連線](./sse.md) — 基於 HTTP 的串流傳輸（已整併入 Streamable HTTP）
- [Streamable HTTP](./streamable_http.md) — 具備串流能力的現代 HTTP 連線
- [WebSocket 連線](./websocket.md) — 全雙工即時通訊協定
