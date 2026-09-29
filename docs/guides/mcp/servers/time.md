# Time MCP 伺服器 (Time MCP Server) {: #time-mcp-server }

[**Time MCP 伺服器**](https://github.com/modelcontextprotocol/servers/tree/main/src/time) 為 AI Agent 提供即時日期與時間感知能力，使其能夠精準處理時序數據、排程規劃與時區跨區換算。任何涉及日期解析、週期性推算或跨國時區排程的工作流程皆可無縫整合該伺服器。

## 安裝與配置方式 {: #installation }

1. **開啟 Griptape Nodes** 並前往 **Settings** → **MCP Servers**

1. **點擊 + New MCP Server**

1. **配置伺服器連線參數**：

    - **Server Name/ID (伺服器名稱/識別碼)**：`time`
    - **Connection Type (連線類型)**：`Local Process (stdio)`
    - **Configuration JSON (組態 JSON)**：

    ```json
    {
        "transport": "stdio",
        "command": "uvx",
        "args": ["mcp-server-time"],
        "env": {},
        "cwd": null,
        "encoding": "utf-8",
        "encoding_error_handler": "strict"
    }
    ```

1. **點擊 Create Server** 完成建立

## 內建可用工具一覽 {: #available-tools }

- **`get_current_time`** — 取得當前的精確日期與時間戳記
- **`parse_date`** — 將各類非結構化日期字串解析為標準結構化物件
- **`format_date`** — 將日期時間格式化轉換為特定字串格式（ISO 8601 等）
- **`add_time`** — 針對指定日期加上特定時長（天數、小時、分鐘等）
- **`subtract_time`** — 針對指定日期減去特定時長
- **`compare_dates`** — 比較兩個日期的先後順序或計算間隔差異
- **`get_timezone_info`** — 取得特定時區的詳細屬性與日光節約時間 (DST) 資訊
- **`convert_timezone`** — 進行不同地理時區之間的時間轉換

## 相關參考資源 {: #resources }

- [Time MCP 伺服器原始碼倉庫](https://github.com/modelcontextprotocol/servers/tree/main/src/time) — 官方標準存放庫與操作手冊
- [JavaScript Date 物件標準規範](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) — 深入了解底層時間處理機制
- [IANA 時區資料庫 (Timezone Database)](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) — 全球標準時區列表與代碼
