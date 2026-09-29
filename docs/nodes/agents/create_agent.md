# Agent {: #agent }

## 功能概述 {: #what-is-it }

Agent 節點讓您能配置具備可自訂能力（如原生工具 Tools 與規則集 Rulesets）的 AI Agent。此節點既可在接收提示詞時立即執行並傳回回答，亦能將配置完備的 Agent 物件傳遞至其他節點的 `agent` 輸入埠中重複使用。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 欲從零建立一個具備高度可配置性的 AI Agent
- 欲為 Agent 綁定專屬的輕量工具與決策規則集
- 欲準備一個可在工作流程中跨多步驟重複調用的長程 Agent
- 欲透過自訂提示詞即時自 Agent 獲取文字回覆

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 Agent 節點新增至工作流程中
1. 配置 Agent 的專屬能力（連線原生工具 Tools 與規則集 Rulesets）

### 參數說明 {: #parameters }

- **agent**：既有的 Agent 組態物件（選填）。若已指定，則會在既有 Agent 的記憶與狀態基礎上進行對話。
- **provider**：欲使用的 AI 提供商（例如 Griptape Cloud、Ollama、LM Studio 或自訂端點）。詳細設定請參閱 [AI 提供商 (AI Providers)](../../guides/agent/providers/index.md)。
- **prompt model**：所選提供商旗下的具體模型名稱。
- **prompt**：您欲向 Agent 發出的具體任務指示或問題。
- **additional_context**：字串或鍵值對字典，用於向 Agent 注入額外的背景脈絡資訊。
- **tools**：欲賦予 Agent 的原生能力工具（計算機、時間、搜尋等）。
- **rulesets**：用於明確規範 Agent 行為準則的規則集，界定其能做與不能做的事項。
- **output_schema**：用於嚴格約束 Agent 回應格式的 JSON Schema 樣板（選填）。

### 輸出埠說明 {: #outputs }

- **output**：Agent 的文字回應內容（若有輸入提示詞）
- **agent**：已配置完畢的 Agent 物件，可連線傳遞給其他節點

## 範例展示 {: #example }

假設您想建立一個能根據 prompt_context 創作俳句的 Agent：

1. 新增一個 KeyValuePair 節點
1. 將 "key" 設為 "topic"，"value" 設為 "swimming"
1. 新增一個 Agent 節點
1. 將 Agent 的 "prompt" 設為 `"Write me a haiku about {{topic}}"`
1. 將 KeyValuePair 節點的 dictionary 輸出連線至 Agent 的 "prompt_context" 輸入埠
1. 執行工作流程
1. Agent 的 "output" 輸出埠將會產出關於游泳主題的優美俳句！

## 使用結構化輸出綱要 (Output Schemas) {: #using-output-schemas }

### 什麼是 Output Schema？ {: #what-is-an-output-schema }

將輸出綱要 (Output Schema) 想像為一張要求 AI 填寫的制式表單。您不再接收非結構化的自由文字段落，而是能明確指定所需的資料欄位及其組織形式。

例如，與其詢問「介紹一下這款產品」並得到一整段文字，您可以明確要求輸出：

- 產品名稱（字串）
- 產品價格（數值）
- 是否有庫存（布林值：是/否）
- 特色功能清單（文字陣列）

AI 將會嚴格遵循要求傳回與您的規格完全吻合的結構化資料。

### 為何需要使用 Output Schema？ {: #why-use-an-output-schema }

在以下情境建議採用 Output Schema：

- **確保格式一致性**：每次呼叫的回覆皆遵循完全相同的資料結構，大幅簡化後續處理邏輯
- **明確指定資料型別**：確保在需要數字處取得數值、需要清單處取得陣列
- **簡化後續自動化串接**：結構化 JSON 資料能極為輕鬆地傳遞至工作流程中的下游節點
- **資料完整性驗證**：AI 必須提供所有必要宣告欄位且格式必須完全合規

### 如何建立 Output Schema {: #how-to-create-an-output-schema }

1. 將一個 **JSON Input** 節點新增至工作流程中
1. 填入定義目標輸出格式的標準 JSON Schema（可藉由線上轉換工具輔助產生）

### Output Schema 具體範例 {: #output-schema-example }

假設您希望自餐廳評論文章中精確擷取結構化資訊：

1. 定義綱要欄位：
    - 欄位 `"restaurant_name"` (型別: string)
    - 欄位 `"rating"` (型別: integer)
    - 欄位 `"price_range"` (型別: string)
    - 欄位 `"cuisine_type"` (型別: string)
    - 欄位 `"recommended_dishes"` (型別: array, 陣列元素型別: string)

1. 將設定連線至 Create Schema 節點

1. 將產出的綱要連線至 Agent 節點的 `output_schema` 輸入埠

1. 設定 Agent 的提示詞：`"Extract restaurant information from this review: [評論全文]"`

1. Agent 此時將回傳結構化 JSON 資料而非雜亂的純文字，內容精準包含上述欄位

1. 生成的標準 Schema 樣板如下：

```json
{
  "type": "object",
  "properties": {
    "restaurant_name": { "type": "string" },
    "rating": { "type": "integer" },
    "price_range": { "type": "string" },
    "cuisine_type": { "type": "string" },
    "recommended_dishes": {
      "type": "array",
      "items": { "type": "string" }
    }
  },
  "required": ["restaurant_name", "rating", "price_range", "cuisine_type", "recommended_dishes"]
}
```

### 套用 Schema 後的行為轉變 {: #what-changes-when-using-a-schema }

- **輸出型別**：Agent 的輸出從一般純文字轉變為嚴謹的結構化資料 (JSON 格式)
- **自動驗證**：若 AI 初次嘗試未能產生符合格式的資料，它會自動重試修正或傳回明確錯誤

## 重要注意事項 {: #important-notes }

- 若未輸入任何提示詞，此節點僅會完成 Agent 實例的初始化而不會觸發推理，此時輸出埠將包含純字串 `"Agent Created"`。
- 此節點同時相容支援串流 (Streaming) 與非串流的 Prompt Drivers。
- Tools（工具）與 Rulesets（規則集）支援以單一項目或清單陣列形式接入。
- `additional_context` 參數允許您以純文字字串或鍵值對字典形式向 Agent 補充動態背景知識。
- 預設情況下節點採用 Griptape Cloud 作為運算引擎，需具備有效的 `GT_CLOUD_API_KEY`。您可透過 **provider** 參數自由切換至本機 Ollama、LM Studio 或客製端點 — 詳閱 [AI 提供商 (AI Providers)](../../guides/agent/providers/index.md)。
- 當您透過 `agent` 輸入/輸出引腳在多個節點之間傳遞 Agent 物件時，**對話記憶將被完好保留**：
    - Agent 深度記住同一個工作流程先前的所有對話互動
    - 先前提示詞所建立的上下文會實質影響 Agent 對新提示詞的理解
    - 您可輕鬆橫跨多個節點搭建複雜的多輪對話 (Multi-turn Conversation)
    - Agent 能精準引用流程早期步驟所獲取的成果數據
- 不熟悉如何編寫 JSON Schema 嗎？可利用 [JSON Schema Builder](https://transform.tools/json-to-json-schema) 等線上視覺化工具快速產出，亦可直接給 Agent 一個範例 JSON 並讓它為您反向生成 Schema 規格！

## 在工作流程中規範 Agent 行為 {: #controlling-agent-behavior-in-workflows }

若欲在工作流程中引導與約束 Agent 的決策模式，請使用 **Rulesets 規則集**（或直接將常規 **TextInput** 節點連線至 `Behaviors (Rulesets)` 輸入埠）。這能讓您明確規範 Agent 的能力邊界、設定角色人設或嚴格約束輸出風格。

!!! note "Agent 節點不支援 Skills 技能資料夾"

    [對話側邊欄 (Chat Sidebar)](../../guides/agent/skills.md) 所使用的 `.agents/skills` 資料夾機制不適用於工作流程畫布中的 Agent 節點。在工作流程中，請統一使用 **Ruleset 節點**或將文字輸入連線至 Agent 的 `Behaviors (Rulesets)` 輸入埠以控制行為。

## 常見問題排解 {: #common-issues }

- **未配置 AI 提供商**：若未特別指定提供商，節點預設連線 Griptape Cloud，此時必須在環境中設定有效之 `GT_CLOUD_API_KEY`。參閱 [AI 提供商 (AI Providers)](../../guides/agent/providers/index.md) 以切換為免費的 Ollama 或 LM Studio 本機引擎。
- **串流異常**：若採用串流驅動模式，請確保下游節點具備相應的串流接收與展示支援。
