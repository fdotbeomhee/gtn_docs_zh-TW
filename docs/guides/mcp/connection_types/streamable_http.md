# 可串流 HTTP 連線 (Streamable HTTP) {: #streamable-http-connection }

**streamable_http** 連線類型允許 Griptape Nodes 透過 HTTP 協定與 MCP 伺服器進行通訊，並完整支援**雙向串流 (Bidirectional Streaming, 用戶端 ↔ 伺服器)** 與即時資料互動。

## 何時使用 Streamable HTTP {: #when-to-use-streamable-http }

- **雙向資料通訊**：用戶端與伺服器兩端皆需自主主動發送資料串流
- **高互動性應用**：即時聊天、多人協同編輯、即時協作畫布
- **相容既有 HTTP 基礎設施**：沿用現成的 API 閘道、負載平衡器與反向代理
- **自訂細粒度串流**：需要比傳統 SSE 更強大的雙向傳輸控制力
- **工作階段持久化 (Session Management)**：需要維護跨請求持久狀態的應用

## 現成可用的 Streamable HTTP MCP 伺服器 {: #available-streamable-http-mcp-servers }

- **[Exa](../servers/exa.md)** — 高階 AI 語意搜尋與深度資料調研能力

## Streamable HTTP MCP 伺服器配置範例 {: #example-streamable-http-mcp-server-configuration }

### 即時通訊應用伺服器 {: #chat-application-server }

```json
{
  "name": "chat_app",
  "transport": "streamable_http",
  "url": "https://api.chat-service.com/mcp/stream",
  "headers": {
    "Authorization": "Bearer chat-token"
  },
  "timeout": 60,
  "sse_read_timeout": 120,
  "terminate_on_close": false,
  "description": "即時訊息傳遞與雙向通訊"
}
```

## 熱門 Streamable HTTP 應用場景 {: #popular-streamable-http-use-cases }

- **即時通訊工具 (Chat)** — 即時串流對話與多方溝通
- **多人協同編輯 (Collaborative Editing)** — 類似 Google Docs 的多人即時共享編輯
- **即時團隊協作 (Live Collaboration)** — 團隊虛擬工作區與即時共享白板
- **互動式儀表板 (Interactive Dashboards)** — 即時遙測數據視覺化與動態互動
- **客服支援系統 (Customer Support)** — 線上即時諮詢與智慧客服系統
- **連線遊戲 (Online Gaming)** — 回合制或即時多人互動機制

## 設定參數說明 {: #configuration }

### 必要欄位 {: #required-fields }

| 欄位名稱 | 型別 | 描述說明 | 範例 |
| :--- | :--- | :--- | :--- |
| `url` | string | MCP 伺服器的 HTTP 服務端點 URL | `"https://api.example.com/mcp"` |

### 選填欄位 {: #optional-fields }

| 欄位名稱 | 型別 | 描述說明 | 預設值 |
| :--- | :--- | :--- | :--- |
| `headers` | object | 用於身分驗證或自訂標頭的 HTTP Headers | `{}` |
| `timeout` | number | 一般請求逾時時限（秒） | `30` |
| `sse_read_timeout` | number | SSE 串流讀取逾時時限（秒） | `60` |
| `terminate_on_close` | boolean | 連線關閉時是否主動終止伺服器端 Session | `true` |

## 組態範例實例 {: #example-configurations }

### 基礎 Streamable HTTP {: #basic-streamable-http }

```json
{
  "name": "streamable_api",
  "transport": "streamable_http",
  "url": "https://api.example.com/mcp/stream",
  "description": "支援雙向串流 (用戶端 ↔ 伺服器) 的 HTTP API"
}
```

### 具備身分驗證的 Streamable HTTP {: #authenticated-streamable-http }

```json
{
  "name": "auth_streamable",
  "transport": "streamable_http",
  "url": "https://api.example.com/mcp/stream",
  "headers": {
    "Authorization": "Bearer your-token-here",
    "Content-Type": "application/json",
    "Accept": "application/json"
  },
  "timeout": 60,
  "sse_read_timeout": 120,
  "terminate_on_close": true
}
```

### 自訂標頭配置 {: #custom-configuration }

```json
{
  "name": "custom_streamable",
  "transport": "streamable_http",
  "url": "https://mcp.example.com/stream",
  "headers": {
    "X-API-Key": "your-api-key",
    "X-Client-Version": "1.0.0",
    "User-Agent": "GriptapeNodes/1.0"
  },
  "timeout": 90,
  "sse_read_timeout": 300,
  "terminate_on_close": false
}
```

## 建置步驟指引 {: #setup-steps }

### 1. 部署 MCP 伺服器端點 {: #1-deploy-mcp-server }

確認您的後端 MCP 伺服器具備 Streamable HTTP 串流回應能力：

```python
# 伺服器端 Streamable HTTP 端點範例 (Python/FastAPI)
@app.post("/mcp/stream")
async def mcp_stream(request: Request):
    return StreamingResponse(
        process_mcp_stream(request),
        media_type="application/json",
        headers={"Cache-Control": "no-cache", "Connection": "keep-alive"},
    )
```

### 2. 在 Griptape Nodes 中配置 {: #2-configure-in-griptape-nodes }

1. 開啟 Griptape Nodes 設定視窗
1. 前往 **MCP Servers** 設定頁面
1. 新增伺服器並將連線類型設為 `streamable_http`
1. 輸入伺服器 API 端點 URL
1. 配置必要的身份驗證標頭與逾時時間
1. 點擊測試連線

### 3. 在工作流程中調用 {: #3-use-in-workflow }

1. 拖曳一個 **MCPTask** 節點至畫布中
1. 選取方才建立的 Streamable HTTP 伺服器
1. 輸入提示詞 (Prompt)
1. 點擊執行工作流程

## 主要優勢 {: #advantages }

- **全雙工雙向串流**：支援 用戶端 ↔ 伺服器 雙向同步實時傳輸
- **良好 Web 相容性**：完美運作於現有標準 HTTP/HTTPS 基礎設施與雲端環境
- **即時狀態更新**：兩端隨時發布與接收最新事件
- **完善的工作階段管理**：內建 Session 狀態維持與回收機制
- **高度客製彈性**：可根據業務邏輯自由定義串流控制協定
- **高互動應用首選**：非常適合即時協作與大型對話情境

## 使用限制 {: #limitations }

- **HTTP 協定開銷**：相較於純本機直接進程管道，存在 HTTP 標頭等封裝開銷
- **高度依賴網路穩定度**：需要穩定、低封包遺失率的網路環境
- **架構複雜度**：相較於純單向 REST 請求需要處理更嚴謹的串流生命週期
- **資源佔用**：長時間維持開放的 HTTP 串流連線會佔用伺服器連線池配額

## Streamable HTTP 與 SSE 比較分析 {: #streamable-http-vs-sse }

| 特性面向 | Streamable HTTP | SSE (Server-Sent Events) |
| :--- | :--- | :--- |
| **資料流向** | **雙向 (用戶端 ↔ 伺服器)** | 單向 (伺服器 → 用戶端) |
| **底層協定** | 自訂 HTTP 串流封裝 | 標準化格式 (`text/event-stream`) |
| **主要場景** | 高互動應用、即時對談、協同編輯 | 推播通知、行情跳動、背景日誌監控 |
| **實作方式** | 自訂用戶端/伺服器通訊邏輯 | 瀏覽器原生內建 `EventSource` 支援 |
| **自動重連** | 需於應用層自訂重連邏輯 | 瀏覽器層級內建自動重新連線 |
| **典型範例** | 即時協同畫布、線上聊天系統 | 股票即時報價機、新聞即時推播 |

## 身分驗證配置 {: #authentication }

### Bearer Token (JWT 權杖) {: #bearer-token }

```json
{
  "headers": {
    "Authorization": "Bearer your-jwt-token"
  }
}
```

### API Key (專屬金鑰) {: #api-key }

```json
{
  "headers": {
    "X-API-Key": "your-api-key",
    "X-Client-ID": "griptape-nodes"
  }
}
```

### 自訂複合身分驗證 {: #custom-authentication }

```json
{
  "headers": {
    "X-Custom-Auth": "your-custom-token",
    "X-User-ID": "user123",
    "X-Session-ID": "session456"
  }
}
```

## 工作階段管理 (Session Management) {: #session-management }

### 斷線即終止 (Terminate on Close) {: #terminate-on-close }

```json
{
  "terminate_on_close": true
}
```

- 當 HTTP 連線中斷或結束時，自動通知伺服器銷毀對應的 Session
- 適合無狀態 (Stateless) 的獨立運算任務
- 系統預設行為

### 持久化工作階段 (Persistent Sessions) {: #persistent-sessions }

```json
{
  "terminate_on_close": false
}
```

- 連線中斷後依然在伺服器端保留工作階段狀態
- 適合需要跨次連線維持上下文的有狀態 (Stateful) 運算
- 需伺服器後端支援工作階段儲存機制

## 疑難排解 {: #troubleshooting }

### 連線異常 (Connection Issues) {: #connection-issues }

- 確認伺服器 URL 可自本機直接瀏覽並回應
- 檢查本機與目標主機之間的網路防火牆
- 使用 curl 或 Postman 獨立發送 POST 請求進行測試
- 檢閱伺服器端存取日誌 (Access Log)

### 逾時問題 (Timeout Problems) {: #timeout-problems }

- 適度調高 `timeout` 或 `sse_read_timeout` 數值
- 檢查伺服器端長任務運算是否卡在資料庫或外部 API
- 監控兩端網路傳輸延遲 (Ping Latency)

### 身分驗證失敗 (Authentication Failures) {: #authentication-failures }

- 確認憑證金鑰是否正確無誤
- 檢查 Token 是否已經過期
- 確認 Header 名稱大小寫與後端規範完全吻合
- 先以獨立 HTTP 工具驗證 Headers 是否能通過鑑權

### 串流中斷或異常 (Streaming Issues) {: #streaming-issues }

- 確認後端伺服器確實支援長連線串流輸出而非一次性緩衝回傳
- 檢查 Content-Type 與 Cache-Control (`no-cache`) 標頭
- 檢查中繼 Proxy / Nginx 是否開啟了緩衝 (`proxy_buffering off`)

## 最佳實踐準則 {: #best-practices }

1. **一律採用 HTTPS**：生產環境必須使用加密連線以保障數據傳輸安全
1. **完善重連機制**：在網路震盪時實作平滑優雅的斷線重連邏輯
1. **主動回收 Session**：避免在伺服器端殘留大量孤兒 Session 消耗記憶體
1. **合理設定逾時時限**：依據任務類型配置長短相宜的超時閥值
1. **安全保護金鑰**：絕不將真實 API 權杖外洩或提交至公開版本庫

## 實用情境組態實例 {: #example-use-cases }

### 互動式對話 (Interactive Chat) {: #interactive-chat }

```json
{
  "name": "chat_interactive",
  "transport": "streamable_http",
  "url": "https://chat.example.com/stream",
  "headers": {
    "Authorization": "Bearer chat-token"
  },
  "terminate_on_close": false
}
```

### 即時團隊協同 (Real-Time Collaboration) {: #real-time-collaboration }

```json
{
  "name": "collaboration",
  "transport": "streamable_http",
  "url": "https://collab.example.com/stream",
  "headers": {
    "X-User-ID": "user123",
    "X-Workspace-ID": "workspace456"
  }
}
```

### 即時數據串流處理 (Live Data Processing) {: #live-data-processing }

```json
{
  "name": "data_processor",
  "transport": "streamable_http",
  "url": "https://processor.example.com/stream",
  "timeout": 120,
  "sse_read_timeout": 600
}
```

## 效能與最佳化考量 {: #performance-considerations }

### 逾時設定梯次建議 {: #timeout-configuration }

- **短逾時 (30-60 秒)**：適用於輕量型即時問答與快速查詢
- **中等逾時 (60-120 秒)**：適用於標準多步驟調研與資料聚合
- **長逾時 (120 秒以上)**：適用於複雜模型運算或長時間批次生成任務

### 連線池最佳化 (Connection Pooling) {: #connection-pooling }

- 盡可能重複使用底層 TCP 連線 (Keep-Alive)
- 監控併發連線上限，防止伺服器連線池耗盡
- 確保異常關閉時優雅釋放通訊埠

## 相關參考資源 {: #next-steps }

- [WebSocket 連線](./websocket.md) — 經典全雙工網路通訊協定
- [SSE 連線](./sse.md) — 單向推播串流協定回顧
- [stdio 連線](./stdio.md) — 本機直接進程通訊架構
