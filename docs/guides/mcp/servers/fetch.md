# Fetch MCP 伺服器 (Fetch MCP Server) {: #fetch-mcp-server }

**[Fetch MCP 伺服器](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch)** 提供強大的網頁內容抓取與提取能力，使 AI Agent 能夠直接自網際網路網頁中獲取並解析資訊。它會自動將 HTML 格式轉換為乾淨結構化的 Markdown 格式以便模型閱讀與分析，是資料調研、內容綜述與網頁爬蟲工作流程的必備工具。

## 安裝與配置方式 {: #installation }

1. **開啟 Griptape Nodes** 並前往 **Settings** → **MCP Servers**

1. **點擊 + New MCP Server**

1. **配置伺服器連線參數**：

    - **Server Name/ID (伺服器名稱/識別碼)**：`fetch`
    - **Connection Type (連線類型)**：`Local Process (stdio)`
    - **Configuration JSON (組態 JSON)**：

    ```json
    {
      "transport": "stdio",
      "command": "uvx",
      "args": ["mcp-server-fetch"],
      "env": {},
      "encoding": "utf-8",
      "encoding_error_handler": "strict"
    }
    ```

1. **點擊 Create Server** 完成建立

## 內建可用工具一覽 {: #available-tools }

- **`fetch`** — 抓取指定 URL 網址的網頁全文內容，並自動將其轉換解析為乾淨的 Markdown 格式

## 進階配置選項 {: #configuration-options }

您可以透過在組態中加入專屬環境變數來客製化 Fetch 伺服器的抓取行為：

```json
{
  "transport": "stdio",
  "command": "uvx",
  "args": ["mcp-server-fetch"],
  "env": {
    "FETCH_TIMEOUT": "30000",
    "FETCH_USER_AGENT": "MyApp/1.0"
  },
  "encoding": "utf-8",
  "encoding_error_handler": "strict"
}
```

可用的環境變數參數：

- **`FETCH_TIMEOUT`** — 網路請求逾時時限，單位為毫秒（預設值：`30000`，即 30 秒）
- **`FETCH_USER_AGENT`** — 發送 HTTP 請求時使用的 User-Agent 識別字串（預設值：`mcp-server-fetch`）
- **`FETCH_MAX_SIZE`** — 允許接收的最大回應資料大小，單位為位元組（預設值：`10485760`，即 10MB）

## 疑難排解 {: #troubleshooting }

### 常見問題 {: #common-issues }

- **伺服器無回應**：檢查本機網路連線狀態，確認目標 URL 是否能於常規瀏覽器中正常開啟，可嘗試更換其他公開網站網址進行交叉驗證
- **內容無法載入**：部分網站會封鎖預設爬蟲與機器人標頭，請嘗試配置自訂的 `FETCH_USER_AGENT`，並確認該頁面是否需要會員登入驗證
- **逾時錯誤 (Timeout Errors)**：在 `env` 中調高 `FETCH_TIMEOUT` 閥值，優先先以結構輕量的小型頁面進行測試，並檢查本機連線頻寬
- **無效的 URL 格式 (Invalid URLs)**：確認網址開頭包含完整的傳輸協定前綴（`http://` 或 `https://`），檢查網址拼寫是否有誤

## 相關參考資源 {: #resources }

- [Fetch MCP 伺服器原始碼倉庫](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch) — 官方標準存放庫與原始程式碼手冊
