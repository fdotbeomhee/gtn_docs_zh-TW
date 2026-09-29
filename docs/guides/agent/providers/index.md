# AI 模型服務供應商 (AI Providers) {: #ai-providers }

Griptape Nodes 支援多種 AI 模型服務供應商，可用於對話[智慧體 (Agent)](../index.md) 以及工作流程中的 **Agent 節點**。系統預設提供 Griptape Cloud。您也可以依需求自由新增本機供應商或任何相容於 OpenAI 規範的自訂 API 端點。

## 支援的供應商一覽 {: #available-providers }

| 服務供應商 | 部署類型 | 是否需要 API 金鑰 |
| ----------------------------------------- | ---------------------- | ---------------- |
| [Griptape Cloud](./griptape_cloud.md)     | 託管代理（預設） | Griptape API 金鑰 |
| [Ollama](./ollama.md)                     | 本機部署 | 否 |
| [LM Studio](./lm_studio.md)               | 本機部署 | 否 |
| [自訂端點 (相容 OpenAI)](./custom.md)     | 託管雲端或本機 | 是 |

## 新增服務供應商 {: #adding-a-provider }

前往 **Settings → Agent Settings**。在 **AI Providers** 區塊中，點擊 **+ Add Provider** 按鈕。

選取您想要新增的供應商類型，並依照其專屬頁面的指引完成設定步驟（可點擊上方表格中的對應連結）。設定精靈將依序引導您完成三個步驟：選擇類型 → 配置參數 → 連線測試。

## 管理服務供應商 {: #managing-providers }

每個供應商條目（除預設的 Griptape Cloud 外）均提供三項管理控制項：

| 控制項 | 操作功能說明 |
| -------------- | ------------------------------------------------------------------------- |
| **開關 (Toggle)** | 啟用或停用。停用的供應商不會出現在模型下拉選單中。 |
| **編輯 (鉛筆圖示)** | 修改顯示名稱、Base URL 基礎連線位址、API 金鑰或預設模型。 |
| **刪除 (垃圾桶圖示)** | 徹底移除該供應商配置。 |

## 在對話側邊欄中使用供應商 {: #using-a-provider-in-the-chat-sidebar }

供應商啟用後，其提供的所有模型將自動列入對話智慧體的 **Model (模型)** 下拉選單中，並依供應商名稱分組歸類。點擊任一模型即可瞬間完成切換。

關於對話介面的詳細說明，請參閱[對話側邊欄 (Agent Sidebar)](../index.md)。

## 在工作流程中使用供應商 {: #using-a-provider-in-a-workflow }

**Agent 節點**包含一個 `provider` 參數。將其設定為任何已啟用的供應商，即可將該節點的 AI 推理呼叫路由至該服務。`prompt model` 參數則允許您自訂挑選特定的具體模型。

這使得同一個工作流程中的不同節點能夠靈活調用不同的供應商與模型。
