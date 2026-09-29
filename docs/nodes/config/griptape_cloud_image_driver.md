# GriptapeCloudImage {: #griptapecloudimage }

## 功能概述 {: #what-is-it }

GriptapeCloudImage 節點用於建立連線至 Griptape Cloud 官方雲端影像生成服務的專屬驅動組態。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 欲直接使用 Griptape Cloud 內建託管服務生成高品質影像（無需額外自行註冊 OpenAI 帳號並取得個人 API Key）
- 欲透過 Griptape 雲端平台無縫調用 DALL-E 3 與各類最新影像擴散模型

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 GriptapeCloudImage 節點新增至工作流程中
1. 將其驅動輸出埠連線至需要生成影像的節點（例如 GenerateImage 節點）

### 參數說明 {: #parameters }

- **model**：欲使用的影像生成模型名稱（預設為 `"gpt-image-1-mini"`）
- **image_size**：欲產出的影像解析度尺寸（預設為 `"1024x1024"`）
- **style**：藝術呈現風格，可選 `natural` 或 `vivid`。`natural` 傾向於寫實風格，具備自然的光影與細膩質理；`vivid` 則會大幅強化色彩飽和度、對比度與畫面張力
- **quality**：生成影像的細節品質等級，可選 `Standard` (標準) 或 `HD` (高畫質)

### 輸出埠說明 {: #outputs }

- **image_model_config**：配置完畢的 Griptape Cloud 影像驅動組態物件，可直接供其他影像處理節點調用

## 範例展示 {: #example }

透過 Griptape Cloud 產出直式構圖影像：

1. 將 GriptapeCloudImage 節點新增至工作流程中
1. 將 "image_size" 設定為 `"1024x1792"` 以適配直式手機螢幕畫面
1. 將 "image_model_config" 輸出埠連線至 GenerateImage 節點的 "image_model_config" 輸入埠
1. 該影像生成節點後續即可直接調用 Griptape 雲端算力自動生成指定規格的畫面

## 重要注意事項 {: #important-notes }

- 您的作業系統環境中需具備有效的 Griptape API 金鑰，環境變數名稱為 `GT_CLOUD_API_KEY`（在 Griptape Nodes 登入環境中通常已由系統自動代入）。
- 節點會根據所選模型架構自動調整支援的長寬比尺寸。
- 值得注意的是：此驅動本身即為 GenerateImage 節點的原生預設配置。若未做特殊參數客製，直接連線至 GenerateImage 時其行為與預設值一致。
