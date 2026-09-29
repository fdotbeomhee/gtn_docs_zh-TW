# 精選 MCP 伺服器範例 (Example MCP Servers) {: #example-mcp-servers }

本專題提供專門針對 Griptape Nodes 最佳化測試過的精選 MCP 伺服器建置指引與詳細文件。這些僅是活躍蓬勃的 MCP 開源社群中極少數的代表性範例。

## 如何發掘更多 MCP 伺服器？ {: #where-to-find-more-mcp-servers }

[MCP 伺服器官方開源存放庫 (Official MCP Servers Repository)](https://github.com/modelcontextprotocol/servers) 收錄了大量參考實作與社群貢獻的優質伺服器。該存放庫充分展示了 MCP 協定的靈活擴展能力，具體體現如何安全、受控地為大型語言模型接入外部工具與真實資料來源。

### 官方存放庫特點： {: #official-repository-features }

- 涵蓋常見核心應用情境的**官方標準參考實作**
- 來自全球開發者社群的豐富貢獻
- 支援**多種主流程式語言**（TypeScript、Python、JavaScript 等）
- 透過 `npx`、`uvx` 與 `pip` 提供極致簡捷的免安裝或一鍵安裝體驗
- 每個伺服器具備詳盡獨立的參數設定手冊

## 現成伺服器手冊導覽 {: #available-server-documentation }

### 本機應用程式 (stdio 傳輸) {: #local-applications-stdio }

此類伺服器直接執行於您的本機電腦上，提供對本機專業軟體與本機檔案系統的直接存取能力：

- **[Blender](./blender.md)** — 3D 建模、場景搭建與渲染自動化（第三方擴充）
- **[Maya](./maya.md)** — Autodesk Maya 3D 建模、骨架動畫與場景管理（第三方擴充）
- **[Fetch](./fetch.md)** — 網頁全文抓取與自動轉 Markdown
- **[Filesystem](./filesystem.md)** — 本機檔案與資料夾結構讀寫管理
- **[Time](./time.md)** — 實體日期、時間感知與排程換算

### 雲端 Web 服務 (streamable_http 傳輸) {: #web-services-streamable_http }

此類伺服器透過網路連線至遠端 Web 應用與雲端 API 服務端點：

- **[Exa](./exa.md)** — 高階語意網路搜尋與深度自動化調研能力

## 尋求更多連線選項？ {: #need-more-options }

- **瀏覽 [MCP 官方開源存放庫](https://github.com/modelcontextprotocol/servers)**，探索數百款社群維護的伺服器
- **探索 [MCP Servers 導覽目錄](https://mcpservers.org/)** — 收錄官方與精選認證伺服器的彙整網站
- **查閱第三方社群目錄**（如 MCPHub 等精選清單）
- **自主打造客製伺服器** — 利用官方提供的 Python、TypeScript 等語言 SDK 快速開發私有伺服器

MCP 生態系統正在飛速成長，社群每日皆在貢獻全新的能力擴充套件！
