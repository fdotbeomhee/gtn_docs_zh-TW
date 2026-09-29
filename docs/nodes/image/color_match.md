# ColorMatch {: #colormatch }

## 功能概述 {: #what-is-it }

ColorMatch 節點運用多種專業色彩轉移演算法，將參考影像 (Reference Image) 的色彩風格與調色特徵遷移至目標影像 (Target Image) 上。這對於自動化影視色彩調光 (Color Grading)、統一不同畫面之間的視覺基調、修復光照不一致問題，以及為多張照片建立連貫的視覺色調風格極具價值。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 在兩張不同照片之間匹配色彩以維持視覺連貫性
- 將專業調色照片的調色風格快速套用至另一張影像
- 修復定格動畫或多角度拍攝鏡頭之間的光照色溫不一致
- 為一系列照片建立統一協調的色調調色盤
- 遷移參考影像中所蘊含的特定光影氛圍
- 為影片影格執行批次色彩匹配校正
- 模仿特定大師級攝影作品的色調質感

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 ColorMatch 節點新增至工作流程中
1. 將提供色彩風格的參考影像連線至 "reference_image" 輸入埠
1. 將欲被修改色調的目標影像連線至 "target_image" 輸入埠
1. 於下拉選單中選取合適的色彩轉移演算法 (method)
1. 依據需求調整轉移強度 (strength) 滑桿
1. 處理完成的色彩匹配影像將於 "output" 輸出埠傳回

### 參數說明 {: #parameters }

#### 影像輸入 (Image Inputs) {: #image-inputs }

- **reference_image**：提供色調調色盤以供複製遷移的參考來源影像
- **target_image**：欲套用色彩轉移的目標影像（畫面色調將被重構）

#### 色彩匹配設定 (Color Match Settings) {: #color-match-settings }

- **method**：用於執行色彩遷移的核心演算法

    - **mkl**：Monge-Kantorovich 線性化演算法（速度極快、品質優異，系統預設）
    - **hm**：長條圖直方圖匹配演算法 (Histogram Matching，速度極快、品質適中）
    - **reinhard**：Reinhard 等人提出的統計色彩轉移法（速度極快、成色自然良好）
    - **mvgd**：多元高斯分佈演算法 (Multi-Variate Gaussian Distribution，速度適中、品質優良）
    - **hm-mvgd-hm**：複合式演算法（運算稍慢、成色品質最頂級）
    - **hm-mkl-hm**：替代複合式演算法（運算稍慢、成色品質最頂級）

- **strength**（強度，範圍 0.0-10.0，預設值：1.0）：控制色彩轉移的混合強度比例

    - 0.0 = 無色彩變更（保留原始目標影像）
    - 1.0 = 100% 完整色彩轉移
    - 數值 > 1.0 誇張強調轉移效果

### 輸出埠說明 {: #outputs }

- **output**：已吸收參考影像色彩風格特徵的完成影像產物

## 範例展示 {: #example }

典型的照片色彩風格遷移工作流程：

1. 使用兩個 LoadImage 節點分別載入兩張影像
1. 將欲調整的照片連線至 "target_image"
1. 將具備理想調色質感的專業照片連線至 "reference_image"
1. 演算法選取 "mkl"，兼顧優異運算效率與高品質成色
1. 將 strength 強度設為 1.0 進行完整色彩映射
1. 將 "output" 引腳連線至 DisplayImage 檢視成果
1. 若效果過於強烈或微妙，可相應增減強度數值

### 實戰應用範例 {: #use-cases }

#### 照片色彩調光 (Photo Color Grading) {: #photo-color-grading }

將專業電影截圖或大師攝影的調色盤快速賦予自己的生活照：

1. 將自拍照片載入為目標影像 (target_image)
1. 將參考電影海報載入為參考影像 (reference_image)
1. 選用 "reinhard" 或 "mkl" 演算法
1. 將 strength 調節於 0.7-1.0 之間獲得自然精緻的電影感

#### 定格動畫校正 (Stop-Motion Correction) {: #stop-motion-correction }

修復逐格拍攝過程中由於環境微光變化造成的閃爍與色差：

1. 挑選光線最完美的標準影格作為 reference
1. 將受光線干擾的異常影格連線為 target
1. 選用快速的 "mkl" 演算法
1. 將 strength 設為 1.0 進行精準還原

#### 批次視覺風格統一 (Batch Style Transfer) {: #batch-style-transfer }

為品牌型錄中的多張產品照打造完全一致的色彩調性：

1. 載入品牌標準形象照作為色彩風格基準
1. 將各產品照片依序送入 ColorMatch 節點
1. 選用 "hm-mvgd-hm" 獲取最頂級的色彩分佈映射成果

## 重要注意事項 {: #important-notes }

- **即時響應 (Live Preview)**：調整參數時工作流程畫布會即時重新運算渲染。
- **色彩空間轉換**：影像在處理過程中會統一轉換至 RGB/Lab 等解耦色彩空間進行精確運算。
- **精準保持尺寸解析度**：輸出影像的寬高像素尺寸完全嚴格維持與 target_image 相同。
- **演算法特性差異**：不同的演算法在亮部過渡與暗部色偏上的表現各有特色，建議依據題材進行切換嘗試。

## 常見問題排解 {: #common-issues }

- **畫面毫無變化**：請確認 strength 強度數值大於 0。
- **色調過於突兀**：將 strength 強度調低至 1.0 以下（例如 0.6 至 0.8）以融入原生色彩。
- **成色異常怪異**：嘗試切換演算法為 "reinhard"，通常能提供最具相容性的自然轉移。
- **處理時間較長**：大型高解析度影像請優先使用 "mkl" 或 "hm"，避免選用運算量較大的複合型演算法。

## 技術實作細節 {: #technical-details }

節點底層整合專業的 `color-matcher` 函式庫執行高維色彩分佈匹配：

- **MKL (Monge-Kantorovich 線性化)**：運用最優傳輸理論 (Optimal Transport Theory) 求解色彩機率分佈的最短轉移路徑。
- **HM (長條圖匹配)**：分別獨立匹配 R、G、B 各色彩維度的累積機率密度分佈 (CDF)。
- **Reinhard 統計轉移**：將色彩轉換至去相關的 $l\alpha\beta$ 色彩空間，匹配兩者的均值與標準差。
- **MVGD (多元高斯分佈)**：同時匹配多維色彩分佈的一階均值向量與二階共變異數矩陣 (Covariance Matrix)。
- **強度線性插值**：強度參數透過線性插值公式實現平滑調控：`result = target + strength * (matched - target)`。
