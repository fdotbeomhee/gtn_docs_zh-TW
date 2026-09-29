# Web Search {: #web-search }

## 功能概述 {: #what-is-it }

Web Search 工具節點是一個網路搜尋工具，可掛載指派給 AI Agent 代理，賦予模型在對話過程中自主檢索網際網路的能力。它支援包含 DuckDuckGo、Google 與 Exa 等多種主流搜尋引擎驅動。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 賦予 AI Agent 在網際網路上檢索資訊的能力
- 存取即時最新網路數據，克服模型訓練截止期限
- 執行自動化調查與研究任務
- 從網際網路上獲取各類動態即時資訊

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 Web Search 工具節點新增至工作流程中
1. 將其輸出引腳連線至需要網路搜尋能力的節點（例如 Agent 節點的 tools 輸入埠）

### 參數說明 {: #parameters }

- **search_engine**：欲使用的搜尋引擎服務（預設為 "DuckDuckGo"）

    - 選項：

        - DuckDuckGo：免費使用，無需任何 API 金鑰
        - Google：需配置 Google API 金鑰與 Custom Search Engine ID
        - Exa：需配置專屬 Exa API 金鑰（針對 AI Agent 優化的語意搜尋）

### 輸出埠說明 {: #outputs }

- **tool**：配置完備的網路搜尋工具物件 (Tool Object)，可直接供其他節點引用

## 範例展示 {: #example }

建立一個能即時上網檢索最新科技資訊的 AI Agent：

1. 將 Web Search 工具節點新增至工作流程中
1. 將 "tool" 輸出埠連線至 Agent 節點的 "tools" 輸入埠
1. 該 Agent 便能在回答使用者時自主判斷是否呼叫搜尋引擎獲取即時資料

## 實作細節 {: #implementation-details }

Web Search 工具底層基於 Griptape 的 `WebSearchTool` 類別封裝，支援多種可替換的搜尋引擎驅動程式：

- `DuckDuckGoWebSearchDriver`：完全免費開箱即用，無需註冊金鑰
- `GoogleWebSearchDriver`：需提供 Google Search API Key 與 Search ID
- `ExaWebSearchDriver`：需提供 Exa API Key

當選用 Google 或 Exa 搜尋引擎時，請務必於環境設定中預先配置對應的 API 金鑰。該工具會自動處理相應服務的身分驗證與搜尋流程。
