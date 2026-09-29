# Web Scraper {: #web-scraper }

## 功能概述 {: #what-is-it }

Web Scraper 工具節點是一個網頁萃取工具，可掛載指派給 AI Agent 代理，賦予模型在對話過程中自主瀏覽並從特定網頁中提取資訊的能力。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 賦予 AI Agent 從外部網站擷取資料的能力
- 自指定 URL 爬取網頁文字正文內容
- 自動化從網際網路公開來源蒐集研究資料
- 構建自動化的網頁內容收集與監控代理

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 Web Scraper 工具節點新增至工作流程中
1. 將其輸出引腳連線至需要網頁爬取能力的節點（例如 Agent 節點的 tools 輸入埠）

### 輸出埠說明 {: #outputs }

- **tool**：配置完備的網頁爬蟲工具物件 (Tool Object)，可直接供其他節點引用

## 範例展示 {: #example }

建立一個能自動瀏覽使用者提供之網址並進行總結的智慧 Agent：

1. 將 Web Scraper 工具節點新增至工作流程中
1. 將 "tool" 輸出埠連線至 Agent 節點的 "tools" 輸入埠
1. 該 Agent 便能在對話中解析使用者提供的連結，自主造訪目標頁面並萃取關鍵資訊

## 實作細節 {: #implementation-details }

Web Scraper 工具底層基於 Griptape 的 `WebScraperTool` 類別封裝，提供簡潔直觀的網頁本文萃取介面。此工具專為 AI 代理設計，能將非結構化的 HTML 網頁內容精煉為結構化純文字。

## 重要注意事項 {: #important-notes }

- 本工具遵循目標網站的服務條款 (Terms of Service) 與 robots.txt 爬蟲規範。
- 爬取效能與品質會因目標網頁的結構複雜度而有所差異。
- 部分具備嚴格反爬機制的網站可能會阻擋自動化爬取請求。
- 本工具最適合處理文字為主的靜態內容；對於高度依賴客戶端 JavaScript 動態渲染的單頁應用程式 (SPA)，支援能力相對有限。
- 請遵守爬蟲道德規範，合理配置請求速率限制 (Rate Limiting)，避免對目標伺服器造成流量負擔。

## 常見問題排解 {: #common-issues }

- **存取被拒 (Access Denied)**：部分網站設有 WAF 或反爬蟲驗證，可能會封鎖請求。
- **內容未找到 (Content Not Found)**：透過前端 JavaScript 動態異步載入的資料可能無法直接擷取。
- **速率限制觸發 (Rate Limiting)**：短時間內發送過多請求可能導致 IP 被暫時限制。
- **版面結構變動 (Changing Layouts)**：目標網站 HTML 結構改版可能影響資訊擷取的準確度。
- **超大頁面處理 (Processing Large Pages)**：極度龐大的網頁可能耗費較多時間，或在進入 LLM 時超出 Token 限制。
