# Groq 提示詞驅動器 (Groq Prompt Driver) {: #groq-prompt-driver }

Groq Prompt Driver 節點讓您能在 Griptape Nodes 架構中配置並調用 Groq 旗下的開源大型語言模型。該節點提供對 Groq 極速 LPU 推理引擎以及各類熱門語言模型的無縫存取能力。

## 設定參數說明 {: #configuration }

### API 金鑰配置 {: #api-key }

使用 Groq 提示詞驅動器前，您需要提供 Groq API 金鑰。您可自 [Groq Console (控制台)](https://console.groq.com/keys) 免費申請取得。

### 支援模型清單 {: #available-models }

Groq 提示詞驅動器支援以下兩大類主流模型：

#### 生產就緒模型 (Production Models) {: #production-models }

- [gemma2-9b-it](https://huggingface.co/google/gemma-2-9b-it) — Google Gemma 2 9B 輕量推理模型
- [meta-llama/llama-guard-4-12b](https://console.groq.com/docs/model/llama-guard-4-12b) — Meta 專用於內容安全審核的防護模型
- [llama-3.3-70b-versatile](https://console.groq.com/docs/model/llama-3.3-70b-versatile) — Meta 具備卓越通用表現的 70B 參數大模型
- [llama-3.1-8b-instant](https://console.groq.com/docs/model/llama-3.1-8b-instant) — Meta 具備極致低延遲的 8B 高速模型
- [llama3-70b-8192](https://console.groq.com/docs/model/llama3-70b-8192) — 支援 8K 上下文長度的 Meta 70B 模型
- [llama3-8b-8192](https://console.groq.com/docs/model/llama3-8b-8192) — 支援 8K 上下文長度的 Meta 8B 模型
- [meta-llama/llama-4-scout-17b-16e-instruct](https://console.groq.com/docs/model/llama-4-scout-17b-16e-instruct) — Meta 具備 16K 上下文的視覺語言模型，相容於 Image Description 節點
- [meta-llama/llama-4-maverick-17b-128e-instruct](https://console.groq.com/docs/model/llama-4-maverick-17b-128e-instruct) — Meta 具備 128K 超長上下文的視覺語言模型，相容於 Image Description 節點

#### 預覽測試模型 (Preview Models) {: #preview-models }

- [allam-2-7b](https://ai.azure.com/explore/models/ALLaM-2-7b-instruct/version/2/registry/azureml) — 沙烏地數據與人工智慧管理局 (SDAIA) 開發的 7B 模型
- [deepseek-r1-distill-llama-70b](https://console.groq.com/docs/model/deepseek-r1-distill-llama-70b) — 基於 DeepSeek-R1 蒸餾的 70B 深度推理模型

### 參數詳細規格 {: #parameters }

Groq 提示詞驅動器支援以下組態參數：

| 參數名稱 | 資料型別 | 預設值 | 描述說明 |
| :--- | :--- | :--- | :--- |
| `model` | string | `gemma2-9b-it` | 用於文字生成的 Groq 模型名稱 |
| `temperature` | float | `0.7` | 控制輸出的隨機度（範圍 0.0 至 1.0） |
| `top_p` | float | `0.9` | 核採樣多樣性閥值（範圍 0.0 至 1.0） |
| `max_tokens` | integer | `2048` | 單次允許產出的最大 Token 數量上限 |
| `stream` | boolean | `false` | 是否啟用打字機串流即時輸出 |
| `max_attempts` | integer | `3` | 請求失敗時的最大自動重試次數 |

## 使用方式說明 {: #usage }

1. 將 Groq Prompt Driver 節點新增至工作流程畫布中
1. 前往 [Groq API Keys](https://console.groq.com/keys) 取得金鑰
1. 在 Griptape Nodes 設定或環境變數中填入 `GROQ_API_KEY`
1. 自下拉選單中挑選欲使用的模型
1. 視需求調整文本生成參數
1. 將其輸出埠連線至流程中的下游節點（如 Agent）

## 重要注意事項 {: #notes }

- Groq 驅動底層直接對接相容於 OpenAI 規範的標準端點：`https://api.groq.com/openai/v1`
- 預覽模型 (Preview Models) 主要用於評估測試，官方可能會在短暫通知後調整或下架
- 生產環境建議一律選用穩定度更高的生產就緒模型 (Production Models)
- 節點在工作流程啟動前會自動對 API 金鑰合法性進行預先檢查
