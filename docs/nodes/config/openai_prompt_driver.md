# OpenAiPrompt {: #openaiprompt }

## 功能概述 {: #what-is-it }

OpenAiPrompt 節點用於建立連線至 OpenAI 旗下旗艦對話與推理模型（例如 GPT-4o 系列）的直連專屬驅動組態。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 欲在工作流程中直接調用 OpenAI 的頂尖大語言模型
- 欲客製化 Agent 與 GPT-4o 等模型互動時的底層超參數
- 欲精確微調 OpenAI 模型回應時的創造力、長度上限與自動重試策略

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 OpenAiPrompt 節點新增至工作流程中
1. 將其驅動輸出連線至能夠接收模型驅動的節點（例如 Agent 節點）

### 參數說明 {: #parameters }

- **model**：欲使用的 OpenAI 模型名稱（預設為 `"gpt-4o"`）
- **stream**：是否啟用即時打字機式串流回應（`true` 為即時串流，`false` 為一次性輸出）
- **temperature**：控制回覆文本的隨機度（數值愈高愈具創意，數值愈低愈精準嚴謹）
- **use_native_tools**：是否啟用 OpenAI 原生 Tool Calling 功能
- **max_tokens**：單次回應允許產出的最大 Token 數量上限
- **max_attempts_on_fail**：遭遇瞬時網路或速率例外時的最大自動重試次數
- **top_p**：控制字詞採樣多樣性（引擎內部會自動進行適當正規化轉換）

### 輸出埠說明 {: #outputs }

- **prompt_model_config**：配置完畢的 OpenAI 驅動組態物件，可直接供其他節點調用

## 範例展示 {: #example }

建立一個透過專屬設定調用 GPT-4o 的 Agent：

1. 將 OpenAiPrompt 節點新增至工作流程中
1. 將其 "prompt_model_config" 輸出連線至 Agent 節點的 "prompt_model_config" 輸入埠

推薦嘗試：

1. 嘗試將 "model" 指定為其他最新模型
1. 將 "temperature" 設為 `0.2`（獲取高度聚焦、確定性更強的回覆）
1. 將 "max_tokens" 設為 `2000`（允許生成長篇結構化分析）

## 重要注意事項 {: #important-notes }

- 您的作業系統環境中必須配置有效的 OpenAI API 金鑰，環境變數名稱為 `OPENAI_API_KEY`。
- 預設模型為 `"gpt-4o"`。
- 內部若傳入 `min_p` 參數，引擎會自動將其映射為 `top_p`（`top_p = 1 - min_p`）。
- 與其他部分驅動不同，OpenAI 原生 API 不支援 `top_k` 採樣參數。
