# MergeImages {: #mergeimages }

## 功能概述 {: #what-is-it }

MergeImages 節點允許您運用多樣化的佈局排版選項，將多張影像整合拼接為單一完整的視覺畫面。它具備高度靈活的幾何排列能力，非常適合用於製作照片拼貼、規則網格、橫向/縱向長條圖以及各類多圖彙整排版。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 製作影像拼貼 (Collages) 或規範網格 (Grids)
- 將多張相關主題的照片整合為單一畫面
- 製作馬賽克視覺圖案或圖樣集合
- 搭建簡報作品集畫廊或成果展示看板
- 將多張影像拼接在一起以便橫向比對
- 製作適應社群平台規範的長條圖或拼圖
- 構建圖文並茂的視覺簡報內容

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 MergeImages 節點新增至工作流程中
1. 將多個影像來源連線至 "input_images" 輸入埠
1. 挑選佈局風格 (layout_style) 並配置間距與欄數
1. 拼接完成的複合影像產物將於 "output" 輸出埠傳回

### 參數說明 {: #parameters }

#### 影像輸入 (Image Input) {: #image-input }

- **input_images**：欲合併拼接的影像清單（可由多個上游影像節點聚合連線輸入）

#### 佈局設定 (Layout Settings) {: #layout-settings }

- **layout_style**：影像的幾何排列排版風格

    - **grid**：標準等大儲存格網格佈局
    - **horizontal**：將所有影像單行橫向並排排列
    - **vertical**：將所有影像單列縱向依序排列
    - **masonry**：非對稱瀑布流佈局，保留各圖原生高度

- **columns**（範圍 1-10，預設值：3）：網格模式下的直行欄數

- **spacing**（範圍 0-50，預設值：10）：圖片相互之間的間隙像素

#### 樣式外觀選項 (Styling Options) {: #styling-options }

- **background_color**（Hex 色碼，預設為 `#ffffff`）：合併畫布的背景底色
- **border_radius**（範圍 0-20，預設值：0）：各圖片四周的圓角像素半徑
- **padding**（範圍 0-100，預設值：20）：整體畫面外圍與畫布邊緣的留白邊距像素

### 輸出埠說明 {: #outputs }

- **output**：整合了所有輸入影像的完成拼接影像產物

## 範例展示 {: #example }

製作帶有圓角與外邊距的雙欄產品展示海報：

1. 使用多個 LoadImage 節點載入多張素材圖片
1. 將所有素材圖片連線聚合至 MergeImages 節點的 "input_images" 參數埠
1. 配置排版佈局：
    - 將 layout_style 選定為 "grid" 標準網格
    - 將 columns 欄數設為 2
    - 將 spacing 間距設為 15 像素
1. 美化視覺外觀：
    - 將 background_color 設為淡灰色 `"#f0f0f0"`
    - 將 border_radius 設為 5 增添精緻微圓角
    - 將 padding 外邊距設為 30 像素保留大器留白
1. 將 "output" 引腳連線至 DisplayImage 檢視完成海報

## 重要注意事項 {: #important-notes }

- **豐富多樣的排版風格**：支援網格、單行橫排、長圖縱排與非對稱瀑布流四種經典佈局。
- **全自動幾何排列**：節點演算法會自適應計算每張圖的最佳寬高縮放，確保構圖嚴整。
- **彈性接收任意數量**：可相容任意多張輸入圖片的即時動態組合。

## 常見問題排解 {: #common-issues }

- **畫面上未顯示圖片**：請確認 "input_images" 參數是否已成功接收到合法的影像清單物件。
- **版面過於擁擠或空曠**：適度微調 columns 欄數、spacing 間隙與 padding 邊距以達到最佳視覺平衡。
