# OpenAiImage {: #openaiimage }

## 功能概述 {: #what-is-it }

OpenAiImage 節點用於建立連線至 OpenAI 旗下 DALL-E 系列影像生成服務的專屬驅動組態。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 欲使用 OpenAI 官方 DALL-E 模型產出高品質圖像
- 欲根據自然語言提示詞描繪生成生動的視覺影像
- 欲將 OpenAI 影像生成能力深度整合至自動化管線中

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 OpenAiImage 節點新增至工作流程中
1. 將其驅動輸出連線至能夠接收影像驅動的節點（例如 GenerateImage 節點）

### 參數說明 {: #parameters }

- **model**：欲使用的模型版本（預設為 `"dall-e-3"`）
- **image_size**：產出影像的解析度尺寸（預設為 `"1024x1024"`）
- **style**：藝術風格取向，可選 `natural` 或 `vivid`。`natural` 偏向擬真寫實質感，光影自然和諧；`vivid` 則偏向強烈戲劇化張力，具備更高的色彩對比度
- **quality**：生成細節等級，可選 `Standard` (標準) 或 `HD` (高畫質)
- **background**：設定影像生成的背景底色偏好
- **moderation**：安全審查敏感度，可選 `Standard` (標準) 或 `Strict` (嚴格審查)
- **output_format**：輸出影像格式，可選 `png` 或 `jpeg`

### 輸出埠說明 {: #outputs }

- **image_model_config**：配置完畢的 OpenAI 影像驅動組態物件，可直接供其他節點調用

## 範例展示 {: #example }

透過 OpenAI DALL-E 產出客製影像：

1. 將 OpenAiImage 節點新增至工作流程中
1. 依需求微調尺寸、風格或畫質參數
1. 將 "image_model_config" 輸出連線至 GenerateImage 節點的 "image_model_config" 輸入埠
1. 該影像節點隨後將透過 OpenAI API 完成高精度影像生成

## 重要注意事項 {: #important-notes }

- 您的作業系統環境中必須配置有效的 OpenAI API 金鑰，環境變數名稱為 `OPENAI_API_KEY`。
- 此節點為 OpenAI 官方影像生成 API 的輕量包裝器。
- 實際調用的 DALL-E 模型行為將依底層驅動的具體組態為準。

## 常見問題排解 {: #common-issues }

- **缺少 API 金鑰**：請確認環境變數中已正確設定 `OPENAI_API_KEY`。
- **連線逾時**：檢查外網連線狀況以及該 API 金鑰是否有效且具備足夠點數配額。
- **配額與頻率限制 (Generation Limits)**：留意 OpenAI 帳號的 Tier 等級限制與每分鐘請求次數 (RPM/TPM)。
