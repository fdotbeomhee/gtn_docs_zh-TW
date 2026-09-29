# ExtendCanvas {: #extendcanvas }

## 功能概述 {: #what-is-it }

ExtendCanvas 節點允許您在既有影像周圍向外延伸畫布，以符合特定的目標長寬比例 (Aspect Ratio) 或自訂像素尺寸。它是在不裁切原始畫面的前提下，新增邊距邊框、適應社群媒體格式或為排版預留空間的理想工具。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 在影像周圍新增填補邊距 (Padding) 或裝飾邊框
- 將影像無損延伸至特定的長寬比例（16:9 寬螢幕、4:3 標準、1:1 正方形等）
- 為不同社群媒體規格（如 Instagram 貼文、YouTube 封面）調整畫布版型
- 在畫面外圍預留疊加標題文字、商標或圖形元素的空間
- 在整套工作流程中統一輸出影像的長寬比規格
- 為印刷排版預留出血位或襯墊邊距

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 ExtendCanvas 節點新增至工作流程中
1. 將影像來源連線至 "input_image" 輸入埠
1. 設定目標長寬比例或自訂寬高像素尺寸
1. 延伸擴展後的影像產物將於 "output" 輸出埠傳回

### 參數說明 {: #parameters }

#### 畫布設定 (Canvas Settings) {: #canvas-settings }

- **target_aspect_ratio**：輸出的目標長寬比規格

    - **16:9**：現代寬螢幕標準格式
    - **4:3**：傳統標準電視格式
    - **1:1**：正方形貼文格式
    - **3:2**：經典相片印刷規格
    - **21:9**：超寬電影螢幕比例
    - **Custom**：自訂寬高數值

- **custom_width**（像素）：選用 "Custom" 模式時手動指定的畫布寬度

- **custom_height**（像素）：選用 "Custom" 模式時手動指定的畫布高度

#### 延伸與填補設定 (Extension Settings) {: #extension-settings }

- **extension_method**：原始影像在新畫布上的錨定定位方式

    - **center**：將原始影像居中置於新畫布中央（預設值）
    - **top_left**：將原始影像靠左上角對齊
    - **top_right**：將原始影像靠右上角對齊
    - **bottom_left**：將原始影像靠左下角對齊
    - **bottom_right**：將原始影像靠右下角對齊

- **background_color**（Hex 色碼，預設為 `#ffffff`）：延伸擴展區域所填充的背景底色

- **background_opacity**（範圍 0.0-1.0，預設值：1.0）：背景填充顏色的不透明度

### 輸出埠說明 {: #outputs }

- **output**：已完成畫布向外延伸擴展的影像產物

## 範例展示 {: #example }

將正方形相片無損擴展為 16:9 寬螢幕桌面：

1. 使用 LoadImage 節點載入 1:1 相片
1. 將該影像連線至 ExtendCanvas 節點的 "input_image" 參數埠
1. 配置畫布設定：
    - 將 target_aspect_ratio 選定為 "16:9"
    - 將 extension_method 設為 "center" 使原圖居中
    - 將 background_color 設為純黑 `"#000000"` 以形成經典寬螢幕黑邊
1. 將 "output" 引腳連線至 DisplayImage 檢視完成的寬螢幕畫面

## 重要注意事項 {: #important-notes }

- **非破壞性擴展 (Non-destructive)**：原始影像的像素內容絕不會被裁切或變形拉伸。
- **靈活對齊控制**：支援 5 種經典錨點方位，隨心所欲決定原圖在新生畫布中的居留位置。
- **RGBA 透明度相容性**：若將 background_opacity 設為 0.0，延伸區域將維持完全透明（輸出 RGBA PNG）。

## 常見問題排解 {: #common-issues }

- **影像未置中**：請檢查 extension_method 設定是否選中為 "center"。
- **背景看不見**：請確認 background_opacity 不透明度大於 0.0。
- **長寬比不吻合**：請確認 target_aspect_ratio 下拉清單的選取項目。
