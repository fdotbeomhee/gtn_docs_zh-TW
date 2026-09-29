# WebSocket 連線 {: #websocket-connection }

**websocket** 連線類型允許 Griptape Nodes 透過 WebSocket 協定與 MCP 伺服器進行全雙工 (Full-Duplex) 即時通訊。

> **注意事項**：WebSocket 並非 MCP 官方現行標準傳輸類型，但根據 [MCP 技術規範 (MCP Specification)](https://modelcontextprotocol.io/specification/2025-06-18/basic/transports)，它可以作為自訂傳輸層 (Custom Transport) 進行延伸實作。

## 什麼是 WebSocket？ {: #what-websocket-is }

WebSocket 是一種在單一 TCP 連線上提供全雙工雙向通訊管道的網路協定。與傳統 HTTP 請求/回應模式不同，WebSocket 允許用戶端與伺服器在任何時刻雙向自由收發資料。

## 支援現況與發展中特性 {: #coming-soon }

MCP 伺服器對 WebSocket 的官方支援仍在持續演進中。雖然連線核心層已可實作為自訂傳輸機制，但目前開源社群中尚無廣泛普及的純 WebSocket 版 MCP 伺服器作為官方範例。

**目前建議優先考慮：**

- **[Streamable HTTP](./streamable_http.md)** — 擁有豐富成熟範例的現代 HTTP 串流通訊方案
- **[本機進程 (stdio)](./stdio.md)** — 擁有大量現成套件支援的本機伺服器直連方案

## 組態配置預覽 {: #configuration }

當社群或專用 WebSocket MCP 伺服器就緒後，其組態配置將類似如下格式：

```json
{
  "transport": "websocket",
  "url": "ws://your-websocket-server.com/mcp",
  "headers": {
    "Authorization": "Bearer your-api-key"
  }
}
```

## 相關參考資源 {: #next-steps }

- **[Streamable HTTP](./streamable_http.md)** — 具備完整實例的現代 HTTP 雙向串流連線
- **[本機進程 (stdio)](./stdio.md)** — 支援豐富的本機進程通訊連線
- **[精選 MCP 伺服器範例](../servers/index.md)** — 各領域精選伺服器一覽
