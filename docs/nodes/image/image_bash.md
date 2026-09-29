# ImageBash {: #imagebash }

## 功能概述 {: #what-is-it }

ImageBash 節點允許您將多張影像整合拼接到單一畫布上，快速建立複合影像（Photo Bashing / Collage）。它非常適合用於製作影像拼貼、多圖網格排版，或透過自訂畫布尺寸與背景底色將零散的素材聚合為協調的單一構圖。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 製作影像拼貼畫 (Collages) 或排版網格
- 將多張零散照片合併為單一複合視覺畫面
- 製作馬賽克風格圖像或概念藝術素材拼貼 (Concept Art Kitbashing)
- 快速搭建作品集畫廊或成果展示看板
- 聚合相關主題的照片以便進行並列審閱
- 製作適應各類社群媒體規範的排版封面
- 製作以視覺圖片為主的簡報畫布

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 ImageBash 節點新增至工作流程中
1. 將多個影像來源連線至 "input_images" 輸入埠
1. 設定畫布尺寸 (canvas_size) 與背景底色 (background_color)
1. 複合拼接後的影像產物將於 "output_image" 輸出埠傳回

### 參數說明 {: #parameters }

#### 畫布設定 (Canvas Settings) {: #canvas-settings }

- **canvas_size**：輸出畫布的規格模式

    - **Custom**：自訂寬度與高度像素
    - **HD**：1280x720 標準高清
    - **Full HD**：1920x1080 全高清
    - **4K**：3840x2160 超高清
    - **Square**：1024x1024 正方形
    - **Instagram Post**：1080x1080 社群貼文
    - **Instagram Story**：1080x1920 直式限時動態

- **width**（像素，預設值：1920）：當 canvas_size 選為 "Custom" 時的手動畫布寬度

- **height**（像素，預設值：1080）：當 canvas_size 選為 "Custom" 時的手動畫布高度

- **background_color**（Hex 色碼，預設為 `#ffffff`）：未被圖片覆蓋的畫布留白區域背景底色

#### 影像輸入 (Image Input) {: #image-input }

- **input_images**：欲合併拼貼的影像清單（可由多個上游影像節點聚合連線輸入）

### 輸出埠說明 {: #outputs }

- **output_image**：整合了所有輸入影像的複合拼接影像產物

## 範例展示 {: #example }

將多張素材快速整合至 1080p 畫布拼圖：

1. 使用多個 LoadImage 節點載入各自獨立的素材圖片
1. 將所有素材圖片聚合傳入 ImageBash 節點的 "input_images" 參數埠
1. 配置畫布設定：
    - 將 canvas_size 選為 "Full HD"（1920x1080 規格）
    - 將 background_color 設為淺灰色 `"#f0f0f0"`
1. 節點將自動在畫布上計算間隔並排版所有圖片
1. 將 "output_image" 引腳連線至 DisplayImage 檢視拼貼海報

## 重要注意事項 {: #important-notes }

- **自動排版佈局 (Automatic Layout)**：節點內部演算法會自動計算最佳擺放位置與行列分佈。
- **靈活的畫布預設**：內建豐富的社群媒體與視訊標準畫布比例。
- **無縫多圖支援**：能彈性接收任意數量的影像輸入並自動適配。

## 常見問題排解 {: #common-issues }

- **畫面上未顯示圖片**：請確認 "input_images" 參數是否已成功接收到合法的影像清單物件。
- **圖片顯得過於微小**：當傳入圖片數量過多時，自動排版會等比縮小素材以完整容納於畫布中，可嘗試將 canvas_size 提升為 4K。
