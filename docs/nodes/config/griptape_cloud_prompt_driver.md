# GriptapeCloudPrompt {: #griptapecloudprompt }

## 功能概述 {: #what-is-it }

GriptapeCloudPrompt 節點用於建立連線至 Griptape Cloud 官方雲端託管 AI 推理服務的專屬驅動組態。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 欲透過 Griptape Cloud 集中託管服務調用各類主流 AI 模型（無需分別前往 OpenAI 等各家廠商註冊與維護獨立 API Key）
- 欲為工作流程中任何接收 Prompt Driver 的下游節點（如 Agent）指定客製化模型組態

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 GriptapeCloudPrompt 節點新增至工作流程中
1. 將其驅動輸出埠連線至需要調用 AI 語言模型的節點（例如 Agent 節點）

### 參數說明 {: #parameters }

- **model**：欲使用的 AI 模型名稱（預設為 `"gpt-4o"`）
- **stream**：是否啟用即時串流打字機輸出模式（`true` 為即時串流，`false` 為運算完畢一次性回傳）
- **temperature**：控制回覆文本的發散度與隨機性（數值愈高愈具創意，數值愈低愈嚴謹收斂）
- **max_attempts_on_fail**：遭遇瞬時網路或配額例外時的最大自動重試次數
- **use_native_tools**：是否允許調用模型底層原生內建的 Tool Calling 能力
- **max_tokens**：單次回覆允許產出的最大 Token 數量上限
- **top_p**：控制字詞採樣多樣性（引擎內部會自動進行適當正規化轉換）

### 輸出埠說明 {: #outputs }

- **prompt_model_config**：配置完畢的 Griptape Cloud 提示詞驅動組態物件，可直接供其他節點調用

## 範例展示 {: #example }

建立一個透過 Griptape Cloud 調用 GPT-4o 的專屬 Agent：

1. 將 GriptapeCloudPrompt 節點新增至工作流程中
1. 將 "model" 設定為 `"gpt-4o"`
1. 將 "prompt_model_config" 輸出埠連線至 Agent 節點的 "prompt_model_config" 輸入埠
1. 該 Agent 隨後將完全透過 Griptape Cloud 平台安全呼叫 GPT-4o 進行多輪對話

推薦嘗試：

1. 將 "temperature" 設為 `0.7`（獲取更豐富多變的創意回答）
1. 切換 "stream" 參數，對比即時打字機串流與整段一次輸出的差異

## 重要注意事項 {: #important-notes }

- 您的作業系統環境中需配置有效的 Griptape API 金鑰，環境變數名稱為 `GT_CLOUD_API_KEY`。
- 預設模型為 `"gpt-4o"`。
- 內部實作中若存在 `min_p` 參數，引擎會自動將其映射轉換為 `top_p`（`top_p = 1 - min_p`）。

## 常見問題排解 {: #common-issues }

- **缺少 API 金鑰**：請確認環境變數中已正確匯入 `GT_CLOUD_API_KEY`。
- **連線逾時**：請確認本機連線至 Griptape Cloud 伺服器的網路線路暢通。
