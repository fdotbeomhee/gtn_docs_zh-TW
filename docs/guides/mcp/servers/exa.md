# Exa MCP 伺服器 (Exa MCP Server) {: #exa-mcp-server }

**[Exa MCP 伺服器](https://github.com/exa-labs/exa-mcp-server)** 透過 Exa AI 專為大型語言模型設計的先進語意搜尋引擎，為您的工作流程提供強大的網頁搜尋、程式碼探勘與深度資料調研能力。它支援本機與遠端託管兩種連線方式，內建豐富的程式碼搜尋、網頁抓取與企業調研專屬工具。

## 安裝與配置方式 {: #installation }

接入 Exa 最簡便的方法是直接連線至其官方託管的雲端 MCP 伺服器：

1. **開啟 Griptape Nodes** 並前往 **Settings** → **MCP Servers**

1. **點擊 + New MCP Server**

1. **配置伺服器連線參數**：

    - **Server Name/ID (伺服器名稱/識別碼)**：`exa`
    - **Connection Type (連線類型)**：`Streamable HTTP`
    - **Configuration JSON (組態 JSON)**：

    ```json
    {
        "transport": "streamable_http",
        "url": "https://mcp.exa.ai/mcp",
        "headers": {},
        "timeout": 30,
        "sse_read_timeout": 300,
        "terminate_on_close": true
    }
    ```

1. **點擊 Create Server** 完成建立

## 內建可用工具一覽 {: #available-tools }

- **`get_code_context_exa`** — 在數十億個 GitHub 存放庫、技術文件與 Stack Overflow 討論串中精準檢索相關程式碼範例
- **`web_search_exa`** — 即時語意網路搜尋，傳回經 AI 最佳化清洗後的乾淨結果
- **`crawling`** — 爬取並解析指定 URL 的全文內容
- **`company_research`** — 針對特定企業或機構進行全方位背景資料聚合
- **`linkedin_search`** — 搜尋 LinkedIn 上的公司專業背景與關鍵人才資料
- **`deep_researcher_start`** — 針對複雜專題啟動長程 AI 深度自主調研任務
- **`deep_researcher_check`** — 查詢並提取深度自主調研任務的結構化完整報告

## 進階配置選項 {: #configuration-options }

您可以在伺服器組態中加入 `enabled_tools` 欄位以精確啟用特定子工具群：

```json
{
  "transport": "streamable_http",
  "url": "https://mcp.exa.ai/mcp",
  "enabled_tools": ["get_code_context_exa", "web_search_exa"]
}
```

推薦工具組合設定：

- **開發者推薦組合**：`["get_code_context_exa", "web_search_exa"]`
- **調研分析人員推薦組合**：`["web_search_exa", "deep_researcher_start", "deep_researcher_check"]`
- **商業情報 (BI) 推薦組合**：`["company_research", "linkedin_search", "web_search_exa"]`
- **啟用全功能工具群**：`["get_code_context_exa", "web_search_exa", "company_research", "crawling", "linkedin_search", "deep_researcher_start", "deep_researcher_check"]`

## 疑難排解 {: #troubleshooting }

### 常見問題 {: #common-issues }

- **連線異常**：測試遠端伺服器位址 `https://mcp.exa.ai/mcp` 是否可直接連通，確認本機網路連線正常，並確認防火牆未阻擋傳出的 HTTPS 請求
- **找不到特定工具 (Tool Not Available)**：確認該工具名稱已列入組態的 `enabled_tools` 清單中，檢查拼寫是否正確，並確認 Exa 官方服務當前是否提供該工具

### 除錯小技巧 {: #debug-tips }

1. 優先使用簡短明確的搜尋詞進行初次測試
1. 登入 Exa 控制台查閱配額用量與詳細錯誤訊息
1. 優先使用官方託管伺服器以簡化網路除錯流程
1. 建議先從基礎搜尋工具開始驗證，再逐步進階使用深度調研功能

## 相關參考資源 {: #resources }

- [Exa MCP 伺服器端點](https://mcp.exa.ai/mcp) — 官方標準服務端點
- [Exa AI 官方首頁](https://exa.ai) — Exa AI 搜尋平台與技術文件
