# MCP 伺服器規則 (MCP Server Rules) {: #mcp-server-rules }

MCP 伺服器規則允許您在 AI Agent 調用特定 MCP 伺服器的工具時，向其注入客製化指引指令。每當 Agent 使用該伺服器的工具時，這些規則會自動作為規則集 (Ruleset) 予以載入套用，確保 Agent 行為的一致性與合規性。

## 什麼是規則？ {: #what-are-rules }

規則是用於指引 AI Agent 應如何與特定 MCP 伺服器互動的文字指令。它們直接保存在 MCP 伺服器組態中，並在以下情況自動生效：

- 在 **MCPTask** 節點中選取並使用該伺服器時
- 在配置了該伺服器的 **Agent** 節點中運作時

## 為何需要設定規則？ {: #why-use-rules }

規則能協助您：

- **精確引導 Agent 行為**：針對如何使用該伺服器的專屬工具提供具體實務指示
- **確保跨流程一致性**：確保 Agent 始終遵循您期望的呼叫模式與回應格式
- **妥善應對極端狀況 (Edge Cases)**：指引 Agent 在發生錯誤或遇到特殊邊界時該如何優雅降級
- **最佳化資源利用**：引導 Agent 以最高效、最具成本效益的方式調用外部能力

## 向 MCP 伺服器新增規則 {: #adding-rules-to-an-mcp-server }

### 建立伺服器時新增 {: #when-creating-a-server }

建立新 MCP 伺服器時，可在 **Rules** 文字方塊中直接填寫規則：

1. 前往 **Settings** → **MCP Servers**
1. 點擊 **+ New MCP Server**
1. 填寫伺服器基礎配置（名稱、連線類型、執行命令等）
1. 在 **Rules** 文字輸入區域中，鍵入您的客製化規則
1. 點擊 **Create Server**

### 編輯既有伺服器時新增 {: #when-editing-a-server }

1. 前往 **Settings** → **MCP Servers**
1. 在欲調整的伺服器條目上點擊 **Edit** 按鈕
1. 修改 **Rules** 文字區域中的內容
1. 點擊儲存變更

## 實用規則範例 {: #example-rules }

### 適用於網頁內容擷取伺服器 (Web Fetch) {: #for-a-web-fetching-server }

```
Always validate URLs before fetching. Check that URLs use HTTPS when possible. If a fetch fails, return a clear error message explaining what went wrong.
```

### 適用於檔案系統伺服器 (Filesystem) {: #for-a-file-system-server }

```
Always check if a file exists before attempting to read it. Use absolute paths when possible. Never delete files without explicit user confirmation.
```

### 適用於搜尋引擎伺服器 (Search) {: #for-a-search-server }

```
Always verify search results are relevant before returning them. If no relevant results are found, suggest alternative search terms. Format results in a clear, readable structure.
```

### 適用於資料庫伺服器 (Database) {: #for-a-database-server }

```
Always validate SQL queries before executing them. Never execute DROP or DELETE operations without explicit confirmation. Return query results in a structured format.
```

## 規則底層運作原理 {: #how-rules-work }

1. **儲存位置**：規則作為 MCP 伺服器組態的一部分持久化儲存。
1. **動態套用**：當 Agent 調用該伺服器的工具時，規則會自動轉換為 prompt ruleset 動態注入模型上下文中。
1. **嚴密作用域**：規則僅在調用該特定 MCP 伺服器時單獨生效，不污染其他無關節點。
1. **格式彈性**：規則為單一長文字字串 — 您可使用句點或換行符號組織多條循序指令。

## 撰寫最佳實踐 {: #best-practices }

### 保持具體精確 {: #be-specific }

✅ **優質**："Always validate URLs before fetching. Check for HTTPS and return clear error messages."

❌ **模糊**："Be careful with URLs."

### 專注於具體行為指導 {: #focus-on-behavior }

✅ **優質**："Return errors in JSON format with 'error' and 'message' fields."

❌ **過於籠統**："Handle errors well."

### 簡明扼要 {: #keep-it-concise }

✅ **優質**："Validate inputs before processing. Return structured JSON responses."

❌ **過於冗長**：長篇大論解釋每種微小邊界情境，導致浪費寶貴的模型上下文空間。

### 務必實測規則 {: #test-your-rules }

配置規則後，請建立測試工作流程進行實證：

1. 建立一個指定該伺服器的 MCPTask 節點
1. 傳入測試提示詞觸發運算
1. 觀察模型輸出，驗證其是否如實遵循了您指定的規則約束

## 疑難排解 {: #troubleshooting }

### 規則未生效 {: #rules-not-being-applied }

- 確認伺服器設定中的 Rules 文字方塊確實包含文字而非空白
- 檢查該 MCP 伺服器當前開關是否處於已啟用狀態
- 確認工作流程中選取的 `mcp_server_name` 與所配置的伺服器名稱完全相符

### Agent 未遵循規則約束 {: #agent-not-following-rules }

- 提高規則的具體性與可操作性
- 先以單一精簡的指令進行測試，逐步追加
- 確保規則內容符合該伺服器底層工具的實際能力邊界

## 相關參考資源 {: #next-steps }

- **[入門教學指南 (Getting Started Tutorial)](./getting_started.md)** — 建置首個 MCP 伺服器
- **[連線架構類型 (Connection Types)](./index.md#connection-types)** — 探索不同的底層連線傳輸架構
- **[精選 MCP 伺服器範例 (Example MCP Servers)](./servers/index.md)** — 各類常見 MCP 伺服器的配置實例
