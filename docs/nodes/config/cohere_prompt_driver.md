# CoherePrompt {: #cohereprompt }

## 功能概述 {: #what-is-it }

CoherePrompt 節點用於建立連線至 Cohere 旗下企業級 AI 語言模型的專屬驅動組態。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 欲在工作流程中調用 Cohere 的高精度文字與對話模型
- 欲充分發揮 Cohere 在長文本檢索增強 (RAG) 與多語言指令理解方面的專屬優勢

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 CoherePrompt 節點新增至工作流程中
1. 將其輸出連線至能接收提示驅動的節點（例如 Agent 節點）

### 參數說明 {: #parameters }

- **model**：欲調用的 Cohere 模型名稱（預設為 `"command-r-plus"`）
- **max_attempts_on_fail**：遭遇瞬時呼叫錯誤時的最大自動重試次數
- **use_native_tools**：是否啟用 Cohere 原生工具調用 (Tool Use) 功能
- **max_tokens**：單次回覆產出的最大 Token 數量上限
- **p**：核採樣機率閥值（控制輸出文本的多樣性）
- **k**：Top-K 採樣數量（控制模型僅自機率最高的前 K 個 Token 中挑選）
- **temperature**：控制生成文本的隨機度與發散程度
- **stream**：是否啟用即時打字機串流輸出模式

### 輸出埠說明 {: #outputs }

- **prompt_model_config**：配置完備的 Cohere 驅動組態物件，可直接供工作流程下游節點調用

## 範例展示 {: #example }

建立一個底層由 Cohere 提供算力支援的 Agent：

1. 將 CoherePrompt 節點新增至工作流程中
1. 將其 "prompt_model_config" 輸出埠連線至 Agent 節點的 "prompt_model_config" 輸入埠

推薦嘗試：

1. 嘗試切換不同的 Cohere 模型版本
1. 將 "max_tokens" 設為 `1000`（生成中等長度的緊湊摘要）

## 重要注意事項 {: #important-notes }

- 您的作業系統環境中必須配置有效的 Cohere API 金鑰，環境變數名稱為 `COHERE_API_KEY`。
- 預設模型為 `"command-r-plus"`。
- 節點會在組態初始化時自動校驗 API 金鑰的合法性。

## 常見問題排解 {: #common-issues }

- **缺少 API 金鑰**：請確認環境變數中已正確設定 `COHERE_API_KEY`。
- **連線逾時或鑑權失敗**：檢查網路連線狀態以及該 API 金鑰是否有效且具備足夠點數。
