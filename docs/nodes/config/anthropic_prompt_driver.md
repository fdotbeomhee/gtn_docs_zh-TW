# AnthropicPrompt {: #anthropicprompt }

## 功能概述 {: #what-is-it }

AnthropicPrompt 節點用於建立連線至 Anthropic 旗下頂尖 AI 模型（例如 Claude 系列）的專屬驅動組態。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 欲在工作流程中使用 Anthropic 的強大對話與推理模型
- 欲客製化 Agent 與 Anthropic 模型互動時的底層參數
- 欲精確微調 Anthropic 模型回應時的創造力、長度限制與重試策略

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 AnthropicPrompt 節點新增至工作流程中
1. 將其輸出連線至能夠接收模型驅動的節點（例如 Agent 節點）

### 參數說明 {: #parameters }

- **model**：欲調用的 Claude 模型名稱（預設為 `"claude-3-7-sonnet-latest"`）
- **stream**：是否啟用即時打字機式串流回應（`true` 為即時串流，`false` 為一次性回傳）
- **temperature**：控制回覆內容的隨機性與發散度（數值愈高代表愈具創意，數值愈低代表愈精準嚴謹）
- **max_attempts_on_fail**：當 API 發生瞬時網路或配額錯誤時的最大自動重試次數
- **use_native_tools**：是否啟用 Anthropic 原生工具調用 (Tool Calling) 支援
- **max_tokens**：單次回應允許產出的最大 Token 上限長度
- **top_p**：控制字詞採樣的多樣性（與 temperature 搭配使用）
- **top_k**：控制模型僅在機率最高的前 K 個 Token 中進行挑選

### 輸出埠說明 {: #outputs }

- **prompt_model_config**：配置完畢的 Anthropic 驅動組態物件，可直接供其他節點調用

## 範例展示 {: #example }

建立一個使用具備特定參數的 Claude 模型的 Agent：

1. 將 AnthropicPrompt 節點新增至工作流程中
1. 將 "prompt_model_config" 輸出連線至 Agent 節點的 "prompt_model_config" 輸入埠
1. 該 Agent 現在將完全遵循您的專屬設定調用 Claude 進行推理運算！

建議嘗試的操作：

1. 嘗試將 "model" 指定為 `"claude-3-5-sonnet-latest"`
1. 將 "temperature" 設為 `0.7`（獲取更具創意的文本回應）
1. 將 "max_tokens" 設為 `2000`（允許生成長篇結構化內容）

## 重要注意事項 {: #important-notes }

- 您的作業系統環境中必須配置有效的 Anthropic API 金鑰，環境變數名稱為 `ANTHROPIC_API_KEY`。
- 預設模型為 `"claude-3-5-sonnet-latest"`。
- 節點在初始化與執行時會自動驗證 API 金鑰的有效性。

## 常見問題排解 {: #common-issues }

- **缺少 API 金鑰**：請確認已在環境變數或應用程式設定中正確輸入了 Anthropic API 金鑰。
- **連線逾時或錯誤**：檢查本機外網連線品質，並確認 API 金鑰帳戶配額是否充裕。
