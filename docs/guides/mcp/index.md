# 讓您的 AI Agent 連接外部世界 (Connect Your Agent to the World) {: #connect-your-agent-to-the-world }

將 **MCP (Model Context Protocol，模型上下文協定)** 想像為一套通用翻譯機，能讓您的 AI Agent 與各種應用程式和外部服務直接對話。Agent 不再受困於孤立封閉的對話框中，MCP 賦予它調用真實工具、存取真實數據並操作實體應用程式的能力。

## Agent 借助 MCP 能做什麼？ {: #what-can-your-agent-do-with-mcp }

透過 MCP，Agent 現已具備以下能力：

- **操作本機檔案** — 讀取、寫入並自動整理電腦中的檔案文件
- **調用專業創作軟體** — 直接控制 Blender 與 Maya 等 3D 建模工具
- **搜尋網際網路** — 透過 Brave 等搜尋引擎即時探測最新資訊
- **發送即時訊息** — 將最新進度發布至 Slack 頻道，與團隊成員無縫溝通
- **管理資料庫** — 向資料庫儲存或擷取各類結構化資訊
- **維持持久記憶** — 在不同對話工作階段之間記錄關鍵上下文資訊

## 核心價值與重要性 {: #why-this-matters }

在引入 MCP 之前，您的 Agent 就像被反鎖在無窗密室裡的智慧助理。而現在，宛如給了該助理整座大樓的通行鑰匙 — 它能自由前往各處、調用任何專業工具，協助您處理真實世界的具體工作。

**真實應用場景：**

- 對 Agent 說：「將我 Blender 場景中的所有模型重新命名為...」，它會真正連線至本機 Blender 並完成批次重命名。
- 交代它：「搜尋關於 AI 的最新即時新聞」，它會自主聯網搜尋並傳回真實結果。
- 交代它：「向團隊發布專案更新進度」，它會直接推播至指定的 Slack 頻道。
- 要求它：「整理我的輸入文件」，它會直接在磁碟上搬移、分類並整理您的實體檔案。

## 連線架構類型 {: #connection-types }

Agent 與外部應用程式連線的兩種官方標準傳輸方式：

1. **[本機應用程式 (Local Apps)](./connection_types/stdio.md)** — 連線至安裝於本機電腦中的軟體

    **具體範例**：[Fetch](./servers/fetch.md)（抓取網頁內容）、[Filesystem](./servers/filesystem.md)（操作本機檔案）、[Time](./servers/time.md)（取得日期與排程管理）

    **最佳適用**：已實體安裝於個人電腦中的本機程式

1. **[雲端 Web 服務 (Web Services)](./connection_types/streamable_http.md)** — 連線至 Web 應用程式與雲端 API 服務

    **具體範例**：[Exa](./servers/exa.md)（AI 語意網路搜尋）、即時數據串流、即時狀態更新

    **最佳適用**：雲端 Web 應用、遠端託管服務與即時通訊服務

### 其他相容選項 {: #additional-options }

- **[伺服器發送事件 (SSE)](./connection_types/sse.md)** — ⚠️ 已淘汰（現已整併為 Streamable HTTP 的一部分）
- **[WebSocket](./connection_types/websocket.md)** — 自訂雙向傳輸協定（由特定用戶端/伺服器實作）

## 準備開始動手實踐？ {: #ready-to-get-started }

1. **[入門教學指南 (Getting Started Tutorial)](./getting_started.md)** — 使用 Fetch MCP 伺服器的循序漸進實戰教學
1. **[在 Agent 中使用 MCPTask (Using MCPTask with Agents)](./mcp_task_agents.md)** — 如何讓 AI Agent 調用這些外部工具
1. **[MCP 伺服器規則 (MCP Server Rules)](./rules.md)** — 為每個 MCP 伺服器配置自訂規則以引導 Agent 行為
1. **[在 Agent 中使用本機模型 (Local Models with Agents)](./advanced_local_models.md)** — 使用本機 AI 模型處理高敏感機密資料
1. **[精選 MCP 伺服器範例 (Example MCP Servers)](./servers/index.md)** — 主流範例伺服器的完整配置指引

## 更多進階資訊 {: #need-more-information }

- [官方 MCP 技術文件](https://modelcontextprotocol.io/docs/getting-started/intro) — 深入了解 MCP 背後的技術架構
- [MCP 伺服器開源倉庫](https://github.com/modelcontextprotocol/servers) — 探索更多由社群開發的連線伺服器
- [MCP 協定技術規範](https://modelcontextprotocol.io/specification/2025-06-18) — 面向開發者的完整技術細節

## 準備好連線您的第一個應用程式了嗎？ {: #ready-to-connect-your-first-app }

立即前往我們的[快速入門指南](./getting_started.md) — 只需幾分鐘即可帶您完成第一個應用程式的連線配置！
