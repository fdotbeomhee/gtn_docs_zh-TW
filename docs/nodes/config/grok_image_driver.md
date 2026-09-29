# GrokImage {: #grokimage }

## 功能概述 {: #what-is-it }

GrokImage 節點用於建立連線至 xAI 旗下 Grok 影像生成服務的專屬驅動組態。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 欲使用 Grok 旗下的最新影像生成模型產生高品質圖像
- 欲根據自然語言提示詞敘述創造生動的視覺美術資產
- 欲將 xAI 影像生成能力整合至您的自動化工作流程中

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 GrokImage 節點新增至工作流程中
1. 將其驅動輸出連線至能接收影像驅動的節點（例如 GenerateImage 節點）

### 參數說明 {: #parameters }

- **model**：欲使用的 Grok 影像模型名稱（預設為 `"grok-2-image-1212"`）

### 輸出埠說明 {: #outputs }

- **image_model_config**：配置完畢的 Grok 影像驅動組態物件，可直接供其他節點調用

## 範例展示 {: #example }

透過 Grok 生成客製化視覺影像：

1. 將 GrokImage 節點新增至工作流程中
1. 視需求調整所選的模型版本
1. 將 "image_model_config" 輸出連線至 GenerateImage 節點的 "image_model_config" 輸入埠
1. 該影像生成節點隨後將透過 xAI 平台依提示詞產出高解析度影像

## 重要注意事項 {: #important-notes }

- 您的作業系統環境中必須配置有效的 Grok API 金鑰，環境變數名稱為 `GROK_API_KEY`，可自 [xAI 控制台 (console.x.ai)](https://console.x.ai) 申請取得。
- 此節點為 xAI 官方影像生成 API 的輕量包裝器。

## 常見問題排解 {: #common-issues }

- **缺少 API 金鑰**：請確認環境變數中已正確設定 `GROK_API_KEY`。
- **連線逾時或失敗**：檢查網路連線狀態以及該 API Key 是否具備足夠點數配額。
- **速率限制 (Rate Limits)**：留意 xAI 平台的呼叫頻率與用量限制，避免短時間內密集觸發。
