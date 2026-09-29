# RescaleImage {: #rescaleimage }

## 功能概述 {: #what-is-it }

RescaleImage 節點允許您在精確掌控尺寸規格、重採樣插值演算法與畫質表現的前提下，自由縮放影像解析度。它同時支援以絕對像素為基準以及以百分比為基準的等比縮放，非常適合用於為特定發布規範預縮放圖片或從單一高清素材生成多種尺寸變體。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 將影像重置縮放至特定的像素解析度
- 依照等比百分比（例如縮小 50% 或放大 200%）縮放影像
- 為不同的社群媒體或輸出格式調整素材尺寸
- 為長文或清單製作縮圖 (Thumbnails) 或輕量化預覽圖
- 為網頁傳輸或列印輸出最佳化檔案體積與畫質
- 批次批次標準化多張照片的解析度
- 在縮放過程中精確維持原始畫面的長寬比例

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 RescaleImage 節點新增至工作流程中
1. 將影像來源連線至 "input_image" 輸入埠
1. 挑選縮放模式 (resize_mode) 並設定目標尺寸數值
1. 縮放完成的影像產物將於 "output" 輸出埠傳回

### 參數說明 {: #parameters }

#### 縮放設定 (Resize Settings) {: #resize-settings }

- **resize_mode**：計算目標新尺寸的基準模式

    - **percentage**：依據百分比等比縮放（例如 50% 代表原圖的一半尺寸）
    - **pixels**：依據絕對像素數值進行鎖定縮放

- **target_size**（像素，預設值：512）：在 pixels 模式下的目標邊長像素值

    - 依據寬度縮放：鎖定寬度為此數值，高度自動等比計算
    - 依據高度縮放：鎖定高度為此數值，寬度自動等比計算

- **percentage_scale**（範圍 1-1000，預設值：100）：在 percentage 模式下的縮放百分比

    - 50 = 原圖的一半尺寸
    - 100 = 維持原圖尺寸
    - 200 = 放大為原圖的兩倍

#### 畫質採樣設定 (Quality Settings) {: #quality-settings }

- **resample_filter**：執行重採樣所使用的插值濾波演算法
    - **lanczos**：極致清晰頂級畫質，運算略耗時（強烈推薦於絕大多數縮放場景）
    - **bicubic**：雙三次插值，畫質優良且運算速度適中
    - **bilinear**：雙線性插值，速度更快但邊緣稍顯平滑
    - **nearest**：最鄰近插值，無平滑濾波，保留強烈像素風格 (Pixelated) 且速度最快

### 輸出埠說明 {: #outputs }

- **output**：重新縮放後的完成影像產物

## 範例展示 {: #example }

將 4K 高畫質照片無失真縮小為 50% 輕量化預覽圖：

1. 使用 LoadImage 載入大圖照片
1. 將該影像連線至 RescaleImage 節點的 "input_image" 參數埠
1. 配置縮放設定：
    - 將 resize_mode 選為 "percentage"
    - 將 percentage_scale 設定為 50
    - 將 resample_filter 指定為 "lanczos" 以確保微小文字與邊緣細節依然銳利
1. 將 "output" 引腳連線至 DisplayImage 檢視縮放成果

## 重要注意事項 {: #important-notes }

- **即時預覽 (Live Preview)**：調整縮放參數時，畫面會即時動態重新計算並渲染。
- **嚴格長寬比保護**：節點一律鎖定並保護原始影像的原生長寬比 (Aspect Ratio)，杜絕畫面形變。
- **Lanczos 演算法推薦**：Lanczos 卷積濾波在縮小圖片時具備極佳的抗鋸齒與細節銳度保持特性。
- **RGBA 透明度完整保留**：帶有透明通道的 RGBA 影像在縮放過程中會完整保留平滑透明邊界。

## 常見問題排解 {: #common-issues }

- **縮放後畫面顯得模糊**：請確認 resample_filter 是否誤選為 bilinear，建議改選 "lanczos"。
- **產生馬賽克像素塊**：請確認 resample_filter 未誤選為 "nearest"（除非刻意追求復古像素藝術風格）。
