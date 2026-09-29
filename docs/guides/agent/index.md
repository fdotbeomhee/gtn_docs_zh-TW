# 智慧體助理 (Agent) {: #agent }

Agent 是直接內建於 Griptape Nodes 編輯器中的 AI 助理，具備對目前工作流程的直接存取權限。您可以向它詢問問題、讓它替您在畫布上建立與連線節點、執行工作流程或檢查畫布上的各項內容 — 一切都在自然流暢的日常對話中輕鬆完成。

## 開啟側邊欄 {: #open-the-sidebar }

側邊欄位於 Griptape Nodes 畫布的右側。點擊 **對話氣泡 (Chat bubble)** 標籤頁（位於 Sidebar Panels 標頭的第一個圖示）即可開啟對話面板。

## 選擇模型 {: #choose-a-model }

在對話面板頂部，點擊 **Model (模型)** 下拉選單可查閱所有可用模型，這些模型依照供應商 (Provider) 進行分類：

- 最頂部群組為 **Griptape Cloud** 模型（包含 Claude、GPT、Gemini、DeepSeek、Llama 等）— 系統預設開箱即用，無需繁瑣設定。
- 您自行配置的任何其他外部供應商會以獨立帶有標籤的區塊展示在下方。
- 使用 **Search models...** 輸入方塊可依名稱即時篩選。
- 點擊底部的 **Manage providers...** 可直達[服務供應商 (Providers)](./providers/index.md)設定畫面。

## 發送訊息 {: #send-a-message }

在底部的 **Write a message...** 輸入框中打字，並按下 **Enter** 鍵或點擊發送按鈕。您也可以使用迴紋針圖示附加檔案。

智慧體會即時串流輸出其回覆。當它採取具體動作時 — 例如讀取畫布內容或觸發執行工作流程 — 您會看到該執行步驟內嵌顯示在對話歷程中，讓您隨時清楚掌握它在做什麼。

舉例來說，您可以要求它：

- **"我的工作流程中有哪些節點？"** — 它會檢視畫布並條列說明現有節點。
- **"添加一個 TextInput 節點並將其連線到 Agent 節點"** — 它會替您在畫布上建立並自動完成端點連線。
- **"執行我的工作流程並告訴我輸出結果是什麼"** — 它會調度執行流程並向您回報產出。
- **"將我的 Agent 節點上的 prompt 修改為..."** — 它會直接替您修改節點內部的參數設定。
- **"解釋這個工作流程的功能與作用"** — 它會深度剖析整體工作流程拓撲架構，並為您提供通俗易懂的摘要總結。

!!! tip "智慧體是如何控制我的工作流程的？"

    Griptape Nodes 在引擎內部隨同運行了一個內建的 **MCP 伺服器**。MCP (Model Context Protocol，模型上下文協定) 是一套開放標準，允許 AI 模型呼叫外部工具 — 在此情境下，該工具集合包含了讀取畫布狀態、建立節點、執行工作流程等一系列 API。對話智慧體會自動連線至該內建 MCP 伺服器，您完全無需進行任何額外手動配置。

## 後續導覽 {: #whats-next }

- [對話對話串 (Threads)](./threads.md) — 對話歷程如何自動儲存以及去何處查找歷史對話
- [個人化配置 (Personalization)](./personalization.md) — 客製化智慧體的溝通語氣與背景偏好設定
- [技能 (Skills)](./skills.md) — 為智慧體注入專屬的領域專業知識
- [外部工具 (MCP 伺服器)](./mcp_servers.md) — 為智慧體掛載檔案存取、網路搜尋及其他各類擴充工具
- [模型供應商 (Providers)](./providers/index.md) — 整合 Ollama、LM Studio 或自訂本機端點
- [在工作流程中使用智慧體 (Using in Workflows)](./using_in_workflows.md) — 探索用於工作流程自動化任務的 Agent 節點
