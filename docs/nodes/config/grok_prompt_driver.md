# GrokPrompt {: #grokprompt }

!!! warning "使用 xAI API 必須預先綁定帳單 (Billing Required for xAI API Usage)"

    在 xAI API Key 正式生效前，GrokPrompt 節點要求必須先在 xAI 平台註冊帳號並綁定有效付款信用卡資訊。若未完成帳單綁定，即使填入合法的 API Key，所有調用 xAI 的節點仍會直接報錯失敗。詳細指引請參閱[整合說明手冊](../../guides/integrations/grok.md)。

## 功能概述 {: #what-is-it }

GrokPrompt 節點用於建立連線至 xAI 旗下 Grok 頂尖大語言模型的專屬驅動組態。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 欲在工作流程中使用 Grok 的對話與推理模型
- 欲客製化 Agent 與 Grok 互動時的各項超參數
- 欲精確微調 Grok 模型回應時的多樣性、溫度與最大 Token 上限

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 GrokPrompt 節點新增至工作流程中
1. 將其輸出連線至能接收提示模型的節點（例如 Agent 節點）

### 參數說明 {: #parameters }

- **model**：欲調用的模型版本。預設為 `"grok-3-beta"`，可選包含 `"grok-3-beta"`、`"grok-3-fast-beta"`、`"grok-3-mini-beta"`、`"grok-3-mini-fast-beta"`、`"grok-2-vision-1212"`
- **top_p**：控制字詞採樣的多樣性（預設為 `0.9`）
- **stream**：是否啟用即時打字機式串流回應（`true` 為即時串流，`false` 為一次性輸出）
- **temperature**：控制回覆隨機性與創造力（數值愈高愈具創意，數值愈低愈精準收斂）
- **max_attempts_on_fail**：遭遇瞬時網路或配額錯誤時的最大自動重試次數
- **use_native_tools**：是否啟用模型原生工具調用支援
- **max_tokens**：單次回覆允許產出的最大 Token 數量上限

### 輸出埠說明 {: #outputs }

- **prompt_model_config**：配置完畢的 Grok 驅動組態物件，可直接供其他節點調用

## 範例展示 {: #example }

建立一個使用具備特定參數的 Grok 模型的 Agent：

1. 將 GrokPrompt 節點新增至工作流程中
1. 將 **prompt_model_config** 輸出連線至 Agent 節點的 **prompt_model_config** 輸入埠
1. 該 Agent 隨後將完全遵循您的專屬設定調用 Grok 進行推理運算！

推薦嘗試：

1. 嘗試將 "model" 自 `"grok-3-beta"` 切換為超高速推理的 `"grok-3-mini-beta"`
1. 將 "top_p" 設為 `0.7`（獲取更精準聚焦的回答）

## 重要注意事項 {: #important-notes }

- 您的作業系統環境中必須配置有效的 Grok API 金鑰，環境變數名稱為 `GROK_API_KEY`。
- 與其他部分驅動不同，Grok 目前不支援 `seed` 與 `top_k` 參數。
- Grok 為付費 API 服務，必須先行於 xAI 官方網站綁定有效帳單與付款方式後，API Key 方具備調用權限。
