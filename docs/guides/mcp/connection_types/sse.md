# 伺服器發送事件連線 (Server-Sent Events, SSE) {: #server-sent-events-sse-connection }

⚠️ **已淘汰 (Deprecated)** — 作為獨立傳輸機制的 SSE 現已正式淘汰，統一由 **Streamable HTTP** 取代。

## SSE 發生了什麼變化？ {: #what-happened-to-sse }

根據 [官方 MCP 技術規範 (Official MCP Specification)](https://modelcontextprotocol.io/specification/2025-06-18/basic/transports)，獨立的 SSE 傳輸方式已在 2024-11-05 協定版本中正式宣告淘汰，並全面由 **Streamable HTTP** 取代。

## 當前官方標準 MCP 傳輸協定 {: #current-mcp-transport-options }

現行 MCP 技術規範僅正式定義了**兩種標準傳輸機制**：

1. **[stdio](./stdio.md)** — 透過標準輸入 (stdin) 與標準輸出 (stdout) 的本機進程通訊
1. **[Streamable HTTP](./streamable_http.md)** — 基於 HTTP 架構且可選支援 SSE 串流的通訊協定

## 這代表什麼意義？ {: #what-this-means }

- **SSE 串流功能依然存在** — 但其已被納入 Streamable HTTP 的一部分，而非作為孤立傳輸層存在
- **Streamable HTTP 可視需求調用 SSE** — 於需要時進行伺服器至用戶端 (Server-to-Client) 的資料串流
- **不再有獨立的 SSE 伺服器類型** — SSE 現已完全深度整合至 HTTP 傳輸架構中

## 遷移升級指南 {: #migration-path }

若您原先計畫使用 SSE，建議改採以下方案：

- **[Streamable HTTP](./streamable_http.md)** — 涵蓋 SSE 串流能力的現代標準替代方案
- **[本機進程 (stdio)](./stdio.md)** — 適用於在本機電腦執行的伺服器連線
- **[WebSocket](./websocket.md)** — 適用於全雙工即時通訊情境（自訂傳輸協定）

## 相關參考資源 {: #next-steps }

- **[Streamable HTTP](./streamable_http.md)** — 具備 SSE 串流能力的現代 HTTP 傳輸
- **[本機進程 (stdio)](./stdio.md)** — 包含豐富實例的本機伺服器連線
- **[精選 MCP 伺服器範例](../servers/index.md)** — 現成可用的各類 MCP 伺服器配置
