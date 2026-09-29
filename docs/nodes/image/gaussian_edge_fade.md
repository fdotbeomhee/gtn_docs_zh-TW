# Gaussian Edge Fade {: #gaussian-edge-fade }

## 概述 {: #overview }

**Gaussian Edge Fade** 節點利用經過高斯模糊運算的 Alpha 透明度通道，在影像四周邊緣套用柔和的羽化漸變淡出效果。這能為影像邊緣營造平滑自然的透明過渡，非常適合將主體無縫疊加於其他背景素材之上，徹底消除生硬突兀的裁切邊界。

## 適用情境 {: #use-cases }

- **多圖層無縫合成 (Image Compositing)**：透過邊緣平滑過渡，將不同影像素材天衣無縫地融合
- **暈影邊框特效 (Vignette Effects)**：製作專業級暗角或邊緣柔光淡出效果
- **疊加背景準備 (Overlay Preparation)**：預先對貼圖素材進行邊緣羽化，以完美融入各類複雜背景
- **標誌圖示處理 (Logo Processing)**：為帶有透明背景的 Logo 邊緣新增羽化，同時完整保留內部鏤空細節
- **精緻相片後製 (Photo Editing)**：製作帶有圓角曲線、自然流暢的有機羽化相框

## 參數說明 {: #parameters }

### 輸入參數 (Input Parameters) {: #input-parameters }

| 參數名稱 | 資料型別 | 說明 |
| ----------------- | -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **input_image**   | ImageArtifact / ImageUrlArtifact | 欲套用邊緣羽化淡出的來源影像 |
| **fade_mode**     | String                           | 測量羽化距離的基準模式："percentage"（相對於影像解析度長寬的百分比）或 "pixels"（絕對像素距離）。預設為 "percentage" |
| **fade_distance** | Integer                          | 從影像邊緣向內羽化的距離。在 percentage 模式下：5 代表影像維度的 5%；在 pixels 模式下：5 代表 5 個像素。預設為 5 |
| **blur_radius**   | Integer                          | 用於平滑邊界的高斯模糊半徑 (0-100)。數值越高，過渡越柔和瀰漫。預設為 10 |
| **fade_curve**    | Float                            | 控制透明度過渡曲率的指數係數 (0.5-4.0)。1.0 代表線性過渡，>1.0 代表邊緣透明度更強（激進羽化），<1.0 代表邊緣保留更多不透明度（柔和羽化）。預設為 2.0 |
| **edge_shape**    | String                           | 羽化的輪廓形狀："square"（直線方邊）或 "rounded"（圓角曲線）。預設為 "square" |
| **replace_mask**  | Boolean                          | 若為 False，將邊緣羽化與影像既有的 Alpha 通道乘法混合；若為 True，則完全覆寫並取代既有 Alpha 通道。預設為 False |
| **apply_top**     | Boolean                          | 是否對頂部邊緣套用羽化。預設為 True |
| **apply_bottom**  | Boolean                          | 是否對底部邊緣套用羽化。預設為 True |
| **apply_left**    | Boolean                          | 是否對左側邊緣套用羽化。預設為 True |
| **apply_right**   | Boolean                          | 是否對右側邊緣套用羽化。預設為 True |

### 輸出參數 (Output Parameters) {: #output-parameters }

| 參數名稱 | 資料型別 | 說明 |
| ---------------- | ---------------- | ------------------------------------------------ |
| **output_image** | ImageUrlArtifact | 套用 Alpha 通道邊緣羽化淡出後的完成影像產物 |

## 核心功能特色 {: #key-features }

### 羽化模式 (Fade Modes) {: #fade-modes }

**Percentage (百分比模式)**：與解析度無關的智慧縮放羽化。無論影像尺寸是 500px 還是 4K，設定 5% 皆能保持視覺上完全一致的等比羽化邊距。

**Pixels (像素模式)**：具備絕對精準像素控制的固定距離羽化。無論原圖尺寸為何，50 像素的羽化邊距皆代表嚴格的 50px 寬度。

### 羽化曲率曲線 (Fade Curve) {: #fade-curve }

Fade Curve 參數運用冪次曲線函式 (Power Function) 精確控制透明度由邊緣向中心過渡的非線性分佈：

- **0.5-0.9**：柔和曲線 - 邊緣保留更多不透明度，並迅速過渡至完全不透明
- **1.0**：標準線性曲線 - 從邊界完全透明到內部不透明均勻等速過渡
- **2.0（預設值）**：激進曲線 - 邊界保持大範圍的高透明度，隨後平緩過渡
- **3.0-4.0**：超激進曲線 - 邊緣極度通透透明，形成極其寬廣深遠的漸變光霧

### 邊界外型形狀 (Edge Shapes) {: #edge-shapes }

**Square (方型直線)**：四個邊界各自獨立平行向內羽化，形成平直的邊界過渡。最適合經典暗角與寬螢幕遮罩。

**Rounded (圓角幾何)**：利用距離場演算法 (Distance Field) 沿曲線輪廓向內計算，在四個角落形成圓潤自然的弧形過渡。是現代 UI 設計與照片合成的最佳首選。

### Alpha 通道容錯與合併 (Alpha Channel Handling) {: #alpha-channel-handling }

**Replace Mask = False (預設)**：完整保留來源圖層的原生透明度，並以乘法混合 (Multiplicative Blending) 疊加邊緣羽化。非常適合保留 PNG Logo 內部的透明去背鏤空。

**Replace Mask = True**：直接忽略原圖的原生 Alpha，全盤套用全新的四邊羽化遮罩。

## 使用範例展示 {: #example-usage }

### 經典暗角/邊界柔化 (Basic Vignette Effect) {: #basic-vignette-effect }

1. 將影像連線至 **input_image**
1. 將 **fade_mode** 設為 "percentage"
1. 將 **fade_distance** 設為 10 (10%)
1. 將 **blur_radius** 設為 15
1. 將 **fade_curve** 設為 2.0
1. 保持四邊 **apply\_**\* 皆為 True
1. 執行節點
結果：獲得四周邊界皆柔和自然淡出的高品質影像。

### 寬螢幕電影感遮罩 (Letterbox Effect) {: #letterbox-effect }

1. 將影像連線至 **input_image**
1. 將 **fade_mode** 設為 "pixels"
1. 將 **fade_distance** 設為 50
1. 將 **blur_radius** 設為 20
1. 將 **apply_top** 與 **apply_bottom** 設為 True
1. 將 **apply_left** 與 **apply_right** 設為 False
1. 執行節點
結果：僅在上下邊界形成柔和漸變遮罩，極具電影畫幅質感。

### 現代圓角卡片合成 (Rounded Corner Composite) {: #rounded-corner-composite }

1. 將影像連線至 **input_image**
1. 將 **edge_shape** 設為 "rounded"
1. 將 **fade_mode** 設為 "percentage"
1. 將 **fade_distance** 設為 8
1. 將 **blur_radius** 設為 12
1. 將 **fade_curve** 設為 2.5
1. 執行節點
結果：影像四角形成平滑圓潤的橢圓形柔焦邊界，完美融入現代網頁 UI。

### 保留透明 Logo 去背細節 (Preserve Logo Transparency) {: #preserve-logo-transparency }

1. 將帶有透明背景的 Logo PNG 連線至 **input_image**
1. 將 **replace_mask** 維持為 False
1. 將 **fade_distance** 設為 5
1. 將 **blur_radius** 設為 10
1. 執行節點
結果：Logo 保留所有原有去背鏤空，外圍輪廓同時獲得順滑的羽化。

## 技巧心得 {: #tips }

- **自然合成技巧**：將 fade_curve 調至 2.0-3.0 之間，使邊緣更加通透，合成時不會留下灰霧生硬邊框。
- **跨解析度規範**：跨多尺寸作業時一律選用 percentage 百分比模式。
- **方向性漸層**：關閉特定方向的 apply 選項即可建立定向單邊漸層。

## 技術實作細節 {: #technical-notes }

- 輸出格式一律為保留 Alpha 透明通道的 RGBA PNG。
- 支援 PIL 所相容的任意輸入影像格式。
- 底層運用 NumPy 矩陣運算實現超高速邊緣距離場梯度計算。
- 呼叫 PIL 的 ImageFilter 執行高精度高斯模糊卷積以確保階調過渡毫無斷層。
- 具備自動尺寸鉗位防護 (Clamping)，防止羽化數值超越影像解析度引發黑邊噪點。
