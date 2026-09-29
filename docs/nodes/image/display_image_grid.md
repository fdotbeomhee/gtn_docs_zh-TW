# DisplayImageGrid {: #displayimagegrid }

## 功能概述 {: #what-is-it }

DisplayImageGrid 節點允許您將多張影像排列為規整的網格 (Grid) 或瀑布流 (Masonry) 畫廊佈局，並具備豐富的樣式自訂選項、預設輸出解析度以及靈活的對齊排列控制。非常適合用於建立作品集畫廊、展示 AI 批次生成結果，或以極具專業質感的視覺排版匯出影像集合。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 建立圖片畫廊 (Gallery) 或個人作品集 (Portfolio)
- 同時展示多張 AI 影像生成的批次結果
- 橫向排版展示處理前後的多版本素材
- 製作精美整理的照片專輯或相簿
- 呈列多個設計方案以供審閱選拔
- 製作以圖片為主的簡報投影片畫面
- 將多張零散照片拼接為規範的網格拼圖
- 依據標準影片規格（4K、1440p、1080p、720p）直接匯出排版畫布

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 DisplayImageGrid 節點新增至工作流程中
1. 將多張影像（清單格式）連線至 "images" 輸入埠
1. 選取理想的佈局風格並配置樣式外觀
1. 選擇輸出尺寸模式（自訂數值或標準預設規格）
1. 整合排版後的精美網格影像將自動生成呈現

### 參數說明 {: #parameters }

#### 影像輸入 (Image Input) {: #image-input }

- **images**：欲在網格中排列展示的影像清單 (List of images)

#### 佈局設定 (Layout Settings) {: #layout-settings }

- **layout_style**：影像的幾何排列風格

    - **grid**：標準等高網格佈局，各儲存格單元大小均一
    - **masonry**：Pinterest 風格的非對稱瀑布流佈局，保留各圖原生高度

- **grid_justification**（僅適用於 grid 佈局）：最後一行不滿整列時的水平對齊方式

    - **left**：靠左對齊（預設值）
    - **center**：置中居中對齊
    - **right**：靠右對齊

- **columns**（範圍 1-10，預設值：4）：網格的欄數（直行數量）

- **spacing**（範圍 0-100，預設值：10）：影像之間的間隙邊距像素

#### 樣式外觀選項 (Styling Options) {: #styling-options }

- **crop_to_fit**（布林值，預設為 `true`）：是否裁切影像使其完全貼合網格儲存格，獲得整齊一致的邊界
- **transparent_bg**（布林值，預設為 `false`）：使用透明背景替代純色底色
- **background_color**（Hex 色碼，預設為 `#000000`）：網格底板的背景顏色
- **border_radius**（範圍 0-500，預設值：8）：圖片圓角的像素半徑（0 代表直角方邊）

#### 輸出設定 (Output Settings) {: #output-settings }

- **output_image_size**：選擇自訂寬度或標準預設影片尺寸

    - **custom**：使用自訂的像素寬度
    - **preset**：使用廣播級標準視訊解析度規格

- **output_image_width**（預設值：1200）：最大像素寬度（當選取 custom 模式時顯示）

- **output_preset**：標準影片解析度選項（當選取 preset 模式時顯示）

    - **4K (3840x2160)**：Ultra HD 超高清規格
    - **1440p (2560x1440)**：Quad HD 規格
    - **1080p (1920x1080)**：Full HD 全高清規格
    - **720p (1280x720)**：標準 HD 高清規格

- **output_format**：匯出的檔案格式

    - **png**：頂級保真畫質，完整支援透明背景通道
    - **jpeg**：壓縮率高檔案小，不支援透明通道
    - **webp**：現代高壓縮率影像格式

### 輸出埠說明 {: #outputs }

- **output**：排版格式化後的網格拼接影像產物 (ImageUrlArtifact)

## 範例展示 {: #example }

建立 1080p 規格的四欄作品展示拼圖：

1. 透過 GenerateImage 批次生成多張影像
1. 將所有生成的影像整合傳遞至 DisplayImageGrid 節點的 "images" 參數埠
1. 配置佈局參數：
    - 將 layout_style 選為 "grid" 以獲得整齊均勻的磁磚效果
    - 將 grid_justification 設為 "center" 使未滿一行的圖片居中排列
    - 將 columns 欄數設為 4
    - 將 spacing 間距設為 10 像素
1. 美化外觀：
    - 將 background_color 設為純黑 `"#000000"`
    - 將 border_radius 設為 8 增添柔和圓角
    - 啟用 crop_to_fit 確保各卡片邊界齊平
1. 選定輸出規格：
    - 將 output_image_size 設為 "preset"
    - 下拉選單指定 "1080p (1920x1080)"
1. 節點將以嚴格的 1920×1080 Full HD 畫布比例輸出精緻的網格拼接海報

## 重要注意事項 {: #important-notes }

- **雙佈局模式切換**：可依據影像長寬比特徵在規整的 Grid 與靈動的 Masonry 之間任意切換。
- **預設廣播規格**：支援一鍵輸出業界標準的 4K、2K、1080p 與 720p 視訊解析度畫布。
- **智慧等比縮放**：使用 preset 模式時，整體網格會嚴格按比例縮放居中，並在畫布兩側自動留白填補背景，杜絕畫面拉伸變形。
- **透明背景相容性**：勾選 `transparent_bg` 並搭配 PNG 或 WebP 格式可無損匯出透明背景圖層。

## 常見問題排解 {: #common-issues }

- **網格未顯示任何圖片**：請確認傳入 "images" 參數的是包含合法影像實體的清單。
- **圖片過小看不清**：可適度增加總畫布寬度或調小欄數 columns（例如由 4 欄改為 2 欄）。
- **預設尺寸與預期不同**：Preset 模式會將畫布嚴格鎖定在例如 1920x1080，內容依據比例縮放居中置放。
