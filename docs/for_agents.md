# 面向 AI 代理 (For Agents) {: #for-agents }

本頁面記載了 [docs.griptapenodes.com](https://docs.griptapenodes.com/) 為 AI 程式碼編寫代理、MCP 技能工具以及任何希望依據引擎真實 API 進行精準錨定 (Grounding) 的自動化工具所暴露的機器可讀介面規格。

驅動此官方文件網站的相同內容亦會發布為經過預處理的 Markdown 格式，供 AI 代理直接抓取解析。程式碼片段、巨集語法與 mkdocstrings 產生的 API 簽章皆已完全展開展開，這是在 `raw.githubusercontent.com` 上直接抓取所無法獲得的。

## 機器可讀端點一覽 (Surface) {: #surface }

- [`/llms.txt`](https://docs.griptapenodes.com/en/stable/llms.txt) 是遵循 [llms.txt 規範](https://llmstxt.org/) 所精心編排的導引索引檔。它將具備極高檢索價值的核心技術文件（指令碼編寫、專案系統、自訂節點開發、MCP 整合以及節點規格參考）依具名章節分組，並隨附簡短描述與絕對網址。
- [`/llms-full.txt`](https://docs.griptapenodes.com/en/stable/llms-full.txt) 是導航目錄中所有頁面合併串接後的完整預處理 Markdown 總集。當您希望將整套引擎文件一次性載入單一大型上下文視窗時，請使用此端點。
- 每個文件頁面皆在其 HTML 渲染版本旁同步提供獨立的 Markdown 原始檔案。存取網址為渲染頁面的 URL 附加 `/index.md`，例如 [`/development/custom_nodes/parameters/index.md`](https://docs.griptapenodes.com/en/stable/development/custom_nodes/parameters/index.md) 與 [`/retained_mode/index.md`](https://docs.griptapenodes.com/en/stable/development/retained_mode/index.md)。頂層頁面位於 `/<page>/index.md`；巢狀子頁面則鏡像對齊渲染後的路徑層級。

## 端點選用決策指引 {: #when-to-use-which }

- **`/llms.txt`**：適用於在不抓取整個文件庫龐大資料量的前提下快速探索技術拓撲。其檔案體積極小，各章節的摘要說明能精準引導您找到對應主題的目標頁面。
- **`/llms-full.txt`**：適用於需要一次性提供全域上下文且具備足夠 Token 預算的情境。其內容與單頁 Markdown 完全一致，並依照 llms.txt 宣告的章節邏輯依序串聯。
- **單頁獨立 `.md`**：適用於已經確切知曉目標頁面的情境（例如編寫自訂節點時查詢參數參考規範，或編寫自動化指令碼時查閱 `retained_mode.md`）。

## 核心 API 錨定高價值頁面推薦 {: #high-value-pages-for-engine-grounding }

若您欲為 AI 代理提供一組精簡的引導文件進行快速冷啟動，以下五份核心文件涵蓋了引擎絕大多數的第一方核心 API：

- [指令碼編寫 (保留模式 Retained Mode)](https://docs.griptapenodes.com/en/stable/development/retained_mode/index.md)
- [自訂節點開發綜述 (Custom Node Development Overview)](https://docs.griptapenodes.com/en/stable/development/custom_nodes/index.md)
- [節點開發快速上手 (Getting Started with Node Development)](https://docs.griptapenodes.com/en/stable/development/custom_nodes/getting_started/index.md)
- [專案系統概述 (Project System Overview)](https://docs.griptapenodes.com/en/stable/guides/projects/index.md)
- [MCP 整合架構概述 (MCP Integration Overview)](https://docs.griptapenodes.com/en/stable/guides/mcp/index.md)

## 穩定性與維護說明 {: #stability }

- 該端點透過現有的文件部署流水線持續追蹤 `main` 分支。目前僅維護單一活動中的即時端點；帶有版本號的靜態 URL（如 `/v0.40/llms.txt`）目前尚未獨立發布。
- 收錄的頁面清單受 `mkdocs.yml` 中的 `llmstxt` 外掛程式配置區塊所驅動。新增技術文件頁面不會自動納入 `/llms.txt` 或 `/llms-full.txt`，必須手動在該設定檔的章節下登記掛載。`nodes/` 下方的節點參考頁面則透過萬用字元 Glob 自動比對匹配。
