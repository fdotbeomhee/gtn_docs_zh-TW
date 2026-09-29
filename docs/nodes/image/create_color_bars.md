# CreateColorBars {: #createcolorbars }

## 功能概述 {: #what-is-it }

CreateColorBars 節點專門用於生成廣播電視與影視工業標準的彩條測試圖樣 (Color Bar Test Patterns)，廣泛應用於影片訊號校準、顯示器測試以及色彩精確度驗證。它支援超過 40 種專業測試圖樣，包括 SMPTE、EBU、ARIB 國際標準，以及 PLUGE、波帶片 (Zone Plates)、色度漸層 (Chroma Ramps) 等各類精確校正圖樣。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 校準專業視訊監視器與顯示器色彩
- 測試顯示設備的色彩準確度、動態範圍與黑白電平
- 為影視後期製作管線生成標準開頭彩條訊號 (Test Patterns)
- 驗證色彩空間 (Color Space) 與 Gamma 映射處理
- 為影視調光 (Color Grading) 工作流程建立基準參考影像
- 壓力測試影片編碼器與解碼器 (Codec) 的色彩重構能力
- 評估螢幕面板的清晰度、頻率響應與失真度

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 CreateColorBars 節點新增至工作流程中
1. 自 "bar_type" 下拉式選單中選取您欲生成的彩條類型
1. 設定測試圖樣的寬度 (width) 與高度 (height) 像素解析度（預設為 1920x1080）
1. 生成的彩條圖樣產物將於 "image" 輸出埠傳回

### 參數說明 {: #parameters }

#### 主要設定 (Main Settings) {: #main-settings }

- **bar_type**：欲生成的彩條測試圖樣種類（字串，預設為 "SMPTE 219-100 Bars"）

    可用選項豐富涵蓋：

    - **SMPTE 標準圖樣**：SMPTE 219-100 Bars、SMPTE 75% Bars、SMPTE Bars、SMPTE 219+i Bars
    - **全場彩條圖樣**：100% Full Field Bars、75% Full Field Bars、75% Bars Over Red
    - **國際廣播標準**：EBU Bars（歐洲標準）、ARIB 28-100、ARIB 28-75、ARIB 28+i（日本標準）
    - **高解析度標準**：HD Color Bars
    - **純色滿版**：Full Field White、Blue、Cyan、Green、Magenta、Red、Yellow
    - **光學頻率測試圖**：Zone Plate（波帶片）、Tartan Bars、Multi Burst（多重頻波）、Bowtie（領結圖）
    - **灰階階梯條**：Stair 5 Step、Stair 5 Step Vert、Stair 10 Step、Stair 10 Step Vert
    - **亮度漸層**：Y Ramp Up、Y Ramp Down、Vertical Ramp
    - **色度漸層**：Legal Chroma Ramp、Full Chroma Ramp、Chroma Ramp
    - **黑電平校準圖樣**：Pluge（附帶進階校準選項）、Pathological EG、Pathological PLL
    - **影音同步特殊圖樣**：AV Delay Pattern 1、AV Delay Pattern 2、Bouncing Box

- **width**：彩條影像的像素寬度（整數，預設為 1920）

- **height**：彩條影像的像素高度（整數，預設為 1080）

#### PLUGE 專屬進階參數 (PLUGE-Specific Parameters) {: #pluge-specific-parameters }

當 bar_type 選定為 "Pluge" 時，以下校準參數將動態顯現：

- **pluge_ire_setup**：PLUGE 圖樣校正的電平標準（字串，預設為 "NTSC 7.5 IRE"）

    - **NTSC 7.5 IRE**：標準 NTSC 黑電平設定（預設值）
    - **PAL 0 IRE**：PAL 0 IRE 絕對黑電平標準
    - **RGB Full Range**：RGB 全範圍（0-255）電腦螢幕校正

- **pluge_bar_count**：顯示的 PLUGE 電平條數量（整數，預設為 3，範圍 2-5）

    - 2 條：標準黑電平與微亮於黑電平
    - 3 條：超黑電平 (Super Black)、標準黑電平、微亮於黑電平（標準配置）
    - 4-5 條：包含黑電平之間的細部過渡級階

- **pluge_orientation**：PLUGE 條的排列方向（字串，預設為 "vertical" 垂直排列）

    - **vertical**：垂直排列（預設值）
    - **horizontal**：水平排列

### 輸出埠說明 {: #outputs }

- **image**：生成的高精度彩條測試影像產物 (ImageUrlArtifact)

## 範例展示 {: #example }

### 生成標準 SMPTE 廣播彩條 {: #basic-color-bars-generation }

1. 將 CreateColorBars 節點新增至工作流程中
1. 自 bar_type 下拉選單中選取 "SMPTE 75% Bars"
1. 將寬度設為 1920，高度設為 1080
1. 將 "image" 輸出埠連線至 DisplayImage 節點檢視成果
1. 視需要連線至 SaveImage 節點以無損 PNG 匯出

### PLUGE 黑電平精準校正圖樣 {: #pluge-calibration-pattern }

1. 新增 CreateColorBars 節點至工作流程中
1. 將 bar_type 選為 "Pluge"
1. 面板將自動展開 PLUGE 專屬參數區塊
1. 配置校準規範：
    - 將 pluge_ire_setup 設為符合目標標準的數值（如 "PAL 0 IRE" 或 "NTSC 7.5 IRE"）
    - 將 pluge_bar_count 設為 3 進行標準的三電平校正
    - 選定 orientation 排列方向
1. 輸出影像並在監視器上調節亮度直到超黑電平隱沒而微亮電平剛好可見

### 自訂 4K 超高解析度測試圖 {: #custom-resolution-test-pattern }

1. 新增 CreateColorBars 節點
1. 選定欲測試的圖樣型態（例如 "HD Color Bars"）
1. 輸入 4K 規格解析度：
    - width: 3840
    - height: 2160
1. 節點將以 4K 解析度數學精確繪製該圖樣

## 重要注意事項 {: #important-notes }

- **即時響應 (Live Preview)**：切換圖樣類型或解析度尺寸時，影像會立即重新運算生成。
- **嚴格標準符合度**：SMPTE 75% Bars 完全遵循 SMPTE ECR 1-1978 規範標準定義的電平比例。
- **IRE 精確電平映射**：PLUGE 校正圖樣依據 IRE (Institute of Radio Engineers) 單位精確計算 RGB 色值。
- **無損 PNG 格式**：所有測試圖樣均以無損 PNG 格式生成與儲存，保證邊緣絕無壓縮失真。

## 常見應用情境 {: #common-use-cases }

### 影視後期製作 (Video Production) {: #video-production }

- 產生符合廣播標準規範的 SMPTE 彩條前導畫面
- 為影視調光工作流程建立絕對標準色階參考
- 檢驗編碼管線的色彩保真度

### 專業顯示器校準 (Display Calibration) {: #display-calibration }

- 運用 PLUGE 條精確校正顯示器的黑電平 (Brightness) 與白電平 (Contrast)
- 透過 Zone Plate 檢測螢幕的摩爾紋 (Moiré) 與極限解析度
- 藉由灰階階梯條檢驗 Gamma 響應曲線的平滑度

### 訊號品質與硬體測試 (Quality Testing) {: #quality-testing }

- 利用病態圖樣 (Pathological Patterns) 測試 SDI 傳輸線路與接收器 PLL 鎖相穩定度
- 透過 Chroma Ramps 驗證色彩空間壓縮與裁切狀況
- 藉由 Multi-burst 條狀訊號測試視訊頻寬與高頻衰減

## 技術實作細節 {: #technical-details }

- **影像格式**：PNG（無損色彩空間映射）
- **色彩空間**：RGB 全彩空間
- **預設解析度**：1920x1080 (Full HD)
- **支援標準規範**：SMPTE、EBU、ARIB
- **IRE 轉換公式**：RGB 數值嚴格按標準電平公式計算：`RGB = (IRE / 100) * 255`
