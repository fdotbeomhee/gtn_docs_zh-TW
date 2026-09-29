# VideoColorMatch {: #videocolormatch }

## 功能概述 {: #what-is-it }

VideoColorMatch 節點運用先進的色彩匹配轉移演算法，將基準參考圖片的調色風格與色彩特徵精確遷移至目標視訊中。在預設情況下，它採用極速 **HALD CLUT（色彩查找表，Color Lookup Table）**技術，僅需對查找表執行一次色彩映射，即可透過 FFmpeg 在單次遍歷中將整部視訊瞬間調色完成，運算速度比傳統逐畫格 (Frame-by-frame) 處理快上 **10 至 50 倍**。這使您能夠將電影劇照、大師海報或特定照片的調色盤風格無損注入動態影片中，極為適合用於打造一致的視覺風格、還原電影質感色彩或批量影片調色。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用 VideoColorMatch 節點：

- 將基準圖片的精緻調色盤 (Color Palette) 遷移至視訊中
- 統一多段不同來源影片片段的色調風格以確保視覺連貫性
- 依據電影劇照或海報打造好萊塢電影級色調 (Cinematic Look)
- 營造契合特定情緒氛圍或美學標準的視訊調色風格
- 為現代高清錄影注入復古膠片或風格化色彩氛圍
- 將靜態攝影作品的經典色彩空間特徵轉移至動態視訊

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 VideoColorMatch 節點新增至工作流程中
1. 將參考圖片連線至 "reference_image"（提供目標色調風格的靜態圖片）
1. 將目標視訊連線至 "target_video"（欲施加調色風格的影片）
1. 挑選色彩轉移模式與演算法
1. 調節強度參數 (strength) 控制調色濃郁度
1. 執行工作流程以套用視訊色彩遷移

### 參數說明 {: #parameters }

#### 輸入端 (Inputs) {: #inputs }

- **reference_image**：提供色調特徵的基準參考圖片（相容 ImageArtifact 與 ImageUrlArtifact）
- **target_video**：欲被轉移色彩的目標視訊（相容 VideoArtifact 與 VideoUrlArtifact）

#### 色彩匹配設定 (Color Match Settings) {: #color-match-settings }

- **transfer_method**：轉碼處理架構

    - **ffmpeg-haldclut**：基於 HALD CLUT 的極速色彩轉移（**預設值，速度快 10-50 倍**）——僅對微型查找表矩陣執行一次色彩匹配，隨後調用 FFmpeg 進行單次硬體加速渲染
    - **frame-by-frame**：逐畫格解壓並單獨處理（傳統模式，耗時極長且消耗大量記憶體）——使用 color-matcher 函式庫逐格提取並計算

- **transfer_algorithm**：核心色彩轉移演算法

    - **mkl**：Monge-Kantorovich 線性化最佳傳輸（預設值，運算極快，畫質優秀，推薦首選）
    - **hm**：直方圖匹配 (Histogram Matching，運算快，色彩對比強烈)
    - **reinhard**：Reinhard 統計色彩轉移（保留原始亮度結構，畫面極其自然）
    - **mvgd**：多元高斯分佈 (Multi-Variate Gaussian Distribution，捕獲細膩色彩關係)
    - **hm-mvgd-hm**：複合雙重直方圖高斯混合模型（運算稍慢，色彩保真度極致）
    - **hm-mkl-hm**：複合雙重直方圖 MKL 模型（運算稍慢，極致畫質）

- **strength**（範圍 0.0-10.0，預設值：1.0）：色彩轉移強度

    - 0.0 = 無色彩變更（維持原始視訊色彩）
    - 1.0 = 完全標準色彩轉移（預設標準值）
    - 數值 > 1.0 = 放大並誇張化色彩飽和度與色相偏移

### 輸出埠說明 {: #outputs }

- **output**：完成色彩轉移的視訊產物 (VideoUrlArtifact)

輸出視訊會嚴格完整保留：

- 原始封裝格式（MP4、MOV 等）
- 原始解析度與畫面長寬比例
- 原始畫面更新率 (FPS)
- 原始音訊軌道（以 192kbps AAC 高保真編碼保留）

## 範例展示 {: #example }

### 基礎電影感調色 (Basic Color Grading) {: #basic-color-grading }

將經典電影海報的冷暖色調套用至空拍視訊：

1. 將 VideoColorMatch 節點新增至工作流程中
1. 使用 LoadImage 載入電影海報並連線至 "reference_image"
1. 使用 LoadVideo 載入空拍視訊並連線至 "target_video"
1. 保持預設的 "mkl" 演算法與 "ffmpeg-haldclut" 模式
1. 將 "strength" 設為 1.0
1. 執行工作流程——影片將瞬間染上電影海報的經典藍綠與暖橙色階氛圍

### 輕柔復古膠片質感 (Subtle Vintage Effect) {: #subtle-vintage-effect }

為現代數位相機影片增添 1970 年代底片色澤：

1. 載入一張泛黃復古老照片作為 reference_image
1. 連線現代視訊作為 target_video
1. 將 transfer_algorithm 選為 "reinhard" 以保護真實環境的明暗反差
1. 將 strength 調微降至 0.7 營造不突兀的淡淡懷舊感
1. 執行工作流程獲取優雅的膠片成果

## 重要注意事項 {: #important-notes }

- **轉碼處理耗時差異**：
    - **ffmpeg-haldclut（預設推薦）**：極速秒級處理——一段 10 秒 1080p 30fps 的視訊在現代硬體上僅需數秒即可渲染完畢。
    - **frame-by-frame**：極其漫長——同一段視訊可能需要耗時 3 至 5 分鐘。僅在追求特殊像素級控制時使用。
- **進度指示反饋**：
    - **ffmpeg-haldclut**：10% 生成 HALD 矩陣，20% 計算色彩映射，70% 完成視訊串流渲染。
    - **frame-by-frame**：0-90% 顯示畫格提取處理進度，90-100% 進行音畫封裝合成。
- **原始特性完整保留**：格式、長寬比、解析度與時間時長完全鎖定。
- **音訊軌道完美保留**：ffmpeg-haldclut 採用音軌直接複製 (copy)，無損保留原生聲音。

## 演算法特性深度解析 {: #algorithm-details }

- **mkl**：基於最佳傳輸理論的線性化算法，速度快且畫質優良，為全能通用標準首選。
- **hm**：基於色彩分佈直方圖對齊，容易產生強烈大膽的色相轉移。
- **reinhard**：統計學亮度與色彩空間分離算法，極佳地保留原圖光影明暗，效果自然耐看。
- **mvgd**：將色彩建模為多維高斯分佈，細膩捕捉相鄰色系的微弱漸變。
- **hm-mvgd-hm**：先進行直方圖粗對齊，再經由高斯分佈平滑，最後進行直方圖校正，頂級品質。
- **hm-mkl-hm**：結合理論最佳傳輸與直方圖匹配的高畫質複合演算法。

## 調色實踐最佳建議 {: #tips-and-best-practices }

### 挑選理想的參考圖片 {: #choosing-a-reference-image }

✓ **優質參考圖特徵：**

- 曝光適當、色彩鮮明且具備明確主色調氛圍的照片
- 光影環境（如日光、夜景、室內燈光）與目標視訊相近的圖片
- 忠實代表您心目中最終成品質感的範例圖

✗ **應避免使用的圖片：**

- 極度過曝死白或嚴重死黑缺乏色彩階調的圖片
- 光線條件與目標視訊徹底相反的圖片（例如拿正午烈日去配黑夜森林）
- 帶有嚴重偏色雜訊或人工過度飽和的圖片

### 強度 (Strength) 推薦設定 {: #strength-guidelines }

| 強度數值 | 適用情境與視覺效果 |
| :--- | :--- |
| **0.3 - 0.5** | 微弱色調校準，輕微增添情緒氛圍 |
| **0.6 - 0.8** | 自然優雅的電影色彩微調（最耐看的日常質感） |
| **0.9 - 1.1** | 完全標準的色彩風格轉移，風格鮮明深刻 |
| **1.2 - 2.0** | 強烈的藝術風格化視覺特效 |
| **> 2.0** | 實驗性、極致誇張的色彩反差 |

## 常見問題排解 {: #common-issues }

- **色調過於極端刺眼**：將 strength 降至 0.6-0.8，或選用 reinhard 演算法以保留更多原始光度。
- **色彩變化不夠明顯**：將 strength 提高至 1.2-1.5，或改選 hm-mvgd-hm 複合演算法。
- **處理速度過慢**：請確認 transfer_method 是否保持為預設的 "ffmpeg-haldclut"（切勿輕易切換至 frame-by-frame）。
- **畫面不如預期**：多嘗試不同的 transfer_algorithm（例如 mkl、reinhard 與 hm-mvgd-hm 的色彩傾向各有千秋）。
