# 將外部 MCP 用戶端連線至 Griptape Nodes (Connect External MCP Clients to Griptape Nodes) {: #connect-external-mcp-clients-to-griptape-nodes }

Griptape Nodes 原生具備自帶的 MCP 伺服器，讓外部 AI Agent（如 Claude Desktop、Claude Code、Cursor、VS Code 等）能直接驅動並調用節點引擎。本章節探討的方向與本單元其餘內容剛好相反：此處並非讓 Griptape Nodes 調用外部 MCP 伺服器，而是**將 Griptape Nodes 自身作為 MCP 伺服器對外暴露**。

## 伺服器端點 URL {: #url }

預設情況下，引擎監聽於以下本機端點：

```
http://localhost:8125/mcp/
```

底層傳輸協定採用 **Streamable HTTP**。強烈建議在結尾保留斜線 `/`；針對省略結尾斜線的用戶端，伺服器會自動將 `/mcp` 重新導向至 `/mcp/`。

當引擎啟動時，會在日誌中輸出實際綁定的監聽位址，例如：

```
INFO MCP server listening at http://127.0.0.1:8125/mcp/
```

## 環境變數覆寫配置 {: #overrides }

服務主機位址與連接埠可透過以下環境變數進行客製調整：

| 環境變數名稱 | 預設值 | 描述說明 |
| :--- | :--- | :--- |
| `GTN_MCP_SERVER_HOST` | `localhost` | 欲綁定的網路介面。明確指定本機請用 `127.0.0.1`，開放區域網路 (LAN) 請用 `0.0.0.0`。 |
| `GTN_MCP_SERVER_PORT` | `8125` | 監聽的 TCP 連接埠。若設為 `0` 則由作業系統自動分配閒置埠號。 |
| `GTN_MCP_SERVER_LOG_LEVEL` | `ERROR` | MCP 伺服器底層 uvicorn 的日誌等級。 |

若設定的連接埠已被其他程式佔用，引擎會自動備援並由作業系統指派空閒埠。請查閱啟動記錄確認實際的 URL。

!!! warning "預設僅限本機存取 (Local-only by default)"

    引擎預設僅綁定 `localhost`，這意味著只有執行在同一部實體電腦上的進程才能存取。內建的 MCP 伺服器**不包含任何身分驗證機制**。除非您能 100% 信任區域網路內的所有裝置，否則切勿將其綁定至 `0.0.0.0` 或暴露於公網環境。

## 用戶端設定指引 {: #client-configuration }

### Claude Code {: #claude-code }

將以下設定新增至 `~/.claude.json`（或透過 `claude mcp add` 指令新增）：

```json
{
  "mcpServers": {
    "griptape-nodes": {
      "type": "streamable-http",
      "url": "http://localhost:8125/mcp/"
    }
  }
}
```

### Cursor {: #cursor }

建立全域 `~/.cursor/mcp.json`，或在特定工作區中建立 `.cursor/mcp.json`：

```json
{
  "mcpServers": {
    "griptape-nodes": {
      "url": "http://localhost:8125/mcp/"
    }
  }
}
```

### VS Code {: #vs-code }

在您的工作區建立 `.vscode/mcp.json`，或透過指令面板點選 **MCP: Open User Configuration** 開啟使用者設定檔：

```json
{
  "servers": {
    "griptape-nodes": {
      "type": "http",
      "url": "http://localhost:8125/mcp/"
    }
  }
}
```

請注意：VS Code 使用 `servers` 鍵名（而非 `mcpServers`），且型別設定為 `"type": "http"`。

### Claude Desktop {: #claude-desktop }

Claude Desktop 的 `claude_desktop_config.json` 原生僅支援 `stdio` 伺服器。若要連線至遠端或 HTTP MCP 伺服器，可採用以下任一方式：

- 在軟體內部前往 **Settings → Connectors → Add custom connector** 並貼上 `http://localhost:8125/mcp/`，或
- 在組態檔案中透過 [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) 進行轉發包裝：

```json
{
  "mcpServers": {
    "griptape-nodes": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "http://localhost:8125/mcp/"]
    }
  }
}
```

## 驗證連線狀態 {: #verifying-the-connection }

最便捷的非互動式驗證方式是使用 [MCP Inspector](https://github.com/modelcontextprotocol/inspector) CLI 工具。由於引擎採用 Streamable HTTP，請務必傳入 `--transport http`：

```bash
npx -y @modelcontextprotocol/inspector --cli http://localhost:8125/mcp/ \
  --transport http --method tools/list
```

若未指定 `--transport http`，Inspector 會預設回退至 SSE，進而引發 `SSE error: Non-200 status code (400)` 報錯，這是因為引擎拒絕了未建立 MCP Session 的純 SSE GET 請求。

Inspector 亦提供圖形化網頁介面：

```bash
npx -y @modelcontextprotocol/inspector
```

在 URL 欄位貼上 `http://localhost:8125/mcp/` 並在 Transport 選單中選取 **Streamable HTTP**。若連線報錯 `TypeError: NetworkError when attempting to fetch resource`，通常為跨來源資源共用 (CORS) 限制所致：引擎的 MCP 伺服器預設不發送 `Access-Control-Allow-Origin` 標頭，因此跨域瀏覽器存取會被攔截。此時請改用上述命令列 CLI 指令，或暫時以放寬安全性原則的瀏覽器開啟。

## 安裝工作流程建構技能 (Workflow-Construction Skill) {: #install-the-workflow-construction-skill }

引擎隨附了官方 [`griptape-nodes-workflows` skill](https://docs.griptapenodes.com/en/stable/skills/griptape-nodes-workflows/SKILL/)，能教會外部 AI Agent 如何精確調用上述 MCP 工具（包含冷啟動配方、`EventRequestBatch` 批次處理與常見注意事項）。Claude Code、Cursor 與 VS Code 皆原生支援 [agentskills.io](https://agentskills.io) 規範中的 `name` + `description` frontmatter 格式，只需放入特定目錄即可生效。

官方發布的 Markdown 原始檔案位於：

```
https://docs.griptapenodes.com/en/stable/skills/griptape-nodes-workflows/SKILL/index.md
```

無論您選擇何種安裝作用域，目錄名稱**必須**命名為 `griptape-nodes-workflows`（與 frontmatter 中的 `name` 完全一致），且檔案名稱**必須**命名為 `SKILL.md`。

### 各用戶端安裝路徑一覽 {: #per-client-install-paths }

| 用戶端環境 | 專案作用域 (Project Scope) | 使用者全域作用域 (User Scope) |
| :--- | :--- | :--- |
| **Claude Code** | `.claude/skills/griptape-nodes-workflows/SKILL.md` | `~/.claude/skills/griptape-nodes-workflows/SKILL.md` |
| **Cursor** | `.cursor/skills/griptape-nodes-workflows/SKILL.md` | `~/.cursor/skills/griptape-nodes-workflows/SKILL.md` |
| **VS Code (Copilot)** | `.github/skills/griptape-nodes-workflows/SKILL.md` | `~/.copilot/skills/griptape-nodes-workflows/SKILL.md` |

Cursor 與 VS Code 亦能同時識別 `.agents/skills/`（專案級）與 `~/.agents/skills/`（使用者級），且 VS Code 還能額外識別 `.claude/skills/` 與 `~/.claude/skills/`。若您希望共用單一資料夾服務多個用戶端，可將技能放置於 `~/.agents/skills/griptape-nodes-workflows/SKILL.md`。

### 單行指令快速安裝 {: #install-in-one-command }

根據上表調整目標路徑 `DEST` 後執行：

```bash
DEST="$HOME/.claude/skills/griptape-nodes-workflows"
mkdir -p "$DEST" \
  && curl -fsSL https://docs.griptapenodes.com/en/stable/skills/griptape-nodes-workflows/SKILL/index.md \
       -o "$DEST/SKILL.md"
```

安裝完成後，在對話框中輸入 `/skills`（適用於 Claude Code 或 VS Code）或前往客製化選單中的 Skills 標籤頁（適用於 Cursor）確認技能已成功載入。
