# AddBoundingBoxes {: #addboundingboxes }

## 功能概述 {: #what-is-it }

AddBoundingBoxes 節點能夠依據傳入的座標字典資料，在影像上繪製邊界框 (Bounding Boxes)。此節點專門用於將物件偵測 (Object Detection) 結果視覺化，在影像上疊加帶有選用文字標籤的彩色邊界矩形框。非常適合用於展示 YOLO、人臉辨識或各類電腦視覺偵測系統的輸出成果。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 視覺化展示 YOLO、SSD 或其他目標偵測模型的預測結果
- 標記人臉偵測邊界框並附帶置信度分數 (Confidence Scores)
- 在影像上標註特定座標範圍的感興趣區域 (Regions of Interest, ROI)
- 為電腦視覺工作流程提供直觀的即時除錯反饋
- 驗證並微調偵測演算法模型的定位準確度
- 在單一影像上整合呈現多組偵測結果

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 AddBoundingBoxes 節點新增至工作流程中
1. 將影像來源連線至 "input_image" 輸入埠
1. 將邊界框資料（單一 dict 字典或 dict 清單）連線至 "bounding_boxes" 輸入埠
1. 視需要自訂邊界框色彩、標籤樣式與線條粗細
1. 已標註完成的影像將於 "output" 輸出埠傳回

### 參數說明 {: #parameters }

#### 必要輸入 (Required Inputs) {: #required-inputs }

- **input_image**：欲繪製邊界框的來源影像（ImageUrlArtifact 或 ImageArtifact）

- **bounding_boxes**：包含邊界框座標的單一字典或字典清單

    - 每個字典必須具備：`x`、`y`、`width`、`height`（整數或可轉換為數值的字串）
    - 可包含其他自訂鍵值以供標籤範本套用（例如 `confidence`、`class` 等）
    - 範例：`{"x": 100, "y": 50, "width": 200, "height": 150, "confidence": 0.95}`

#### 樣式外觀參數 (Styling Parameters) {: #styling-parameters }

- **box_color**（十六進位顏色碼 Hex Color，預設為 `"#FF0000"`）：邊界框外框的線條顏色

    - 整合 ColorPicker 調色盤以便直觀選取顏色
    - 支援標準 Hex 格式（例如紅色 `"#FF0000"`、綠色 `"#00FF00"`）

- **line_thickness**（範圍 1-10，預設為 `2`）：邊界框外框的像素粗細度

    - 較小數值適用於精細微小的標註
    - 較大數值可帶來醒目突出的邊界框視覺

#### 標籤參數 (Label Parameters) {: #label-parameters }

- **show_labels**（布林值，預設為 `True`）：切換文字標籤的可見度

    - 設為 `True` 時於邊界框上方顯示標籤文字
    - 設為 `False` 時隱藏標籤，僅繪製矩形框

- **label_key**（字串範本 Template）：邊界框標籤的文字格式化範本

    - 預設值：`"{x}, {y}, width: {width}, height: {height}"`
    - 使用 `{key}` 語法動態插入對應邊界框字典中的數值
    - 範例：`"類別: {class}, 置信度: {confidence}"`
    - 範例：`"{x}, {y} - 尺寸: {width}x{height}"`
    - 若字典中不存在該鍵值，將原樣保留 `{key}` 文字輸出

### 輸出埠說明 {: #outputs }

- **output**：已繪製邊界框與文字標籤的完成影像產物

## 範例展示 {: #example }

典型的物件偵測視覺化工作流程：

1. 使用 LoadImage 節點載入待測影像
1. 執行目標偵測演算法（如 YOLO 或自訂偵測節點）
1. 將偵測結果封裝為字典清單：
    ```
    [
      {"x": 100, "y": 50, "width": 200, "height": 150, "class": "person", "confidence": 0.95},
      {"x": 350, "y": 120, "width": 180, "height": 200, "class": "car", "confidence": 0.87}
    ]
    ```
1. 將影像連線至 AddBoundingBoxes 的 "input_image"
1. 將偵測結果字典連線至 "bounding_boxes"
1. 設定標籤範本：`"Class: {class}, Conf: {confidence}"`
1. 選取邊界框色彩：`"#00FF00"`（綠色）
1. 將線條粗細 line_thickness 設為 `3` 提升可讀性
1. 將 "output" 引腳連線至 DisplayImage 節點檢視標註影像

## 重要注意事項 {: #important-notes }

- **座標系統規範**：邊界框採用左上角頂點作為 `(x, y)` 基準起點，`width` 向右延伸，`height` 向下延伸。
- **字串座標自動轉換**：若傳入的座標數值為字串型態，節點會自動嘗試將其轉換為整數（例如 `{"x": "100", "y": "50"}` 可正常運作）。
- **智慧標籤定位機制**：
    - 預設：放置於邊界框正上方，帶有半個標籤高度的間距
    - 靠近畫面上邊緣：自動自適應移至邊界框內部的頂端，防止被畫面裁切
- **動態字型縮放**：文字大小自動按影像總高度的 4% 進行等比縮放，確保在不同解析度下具備良好的可讀性。
- **嚴格邊界驗證**：座標必須為非負整數（`x ≥ 0`, `y ≥ 0`），寬度與高度必須大於零（`width > 0`, `height > 0`）。
- **RGBA 透明度保留**：若輸入為帶有透明通道的 RGBA 影像，繪製時將完整保留透明度通道。

## 常見問題排解 {: #common-issues }

- **缺少必要鍵值 (Missing Required Keys)**：請確保每個邊界框字典皆包含 `x`、`y`、`width`、`height`。
- **負數座標異常 (Negative Coordinates)**：邊界框頂點座標不能為負值。
- **零或負數尺寸 (Zero or Negative Dimensions)**：寬度與高度數值必須嚴格大於零。
- **標籤未顯示 (Labels Not Showing)**：檢查 `show_labels` 是否為 `True`，且範本中的 `{key}` 是否與字典中的鍵值大小寫完全一致。

## 技術實作細節 {: #technical-details }

節點執行以下處理管線：

1. **輸入合法性驗證**：檢查字典結構與必要座標欄位，並將合法字串安全轉為整數。
1. **色彩代碼解析**：將 Hex 十六進位色碼轉為 RGB 三元組色彩。
1. **動態字型載入**：根據影像高度動態配置比例字型大小。
1. **繪製邊界框**：利用 PIL (Pillow) ImageDraw 在畫布上精準繪製矩形外框。
1. **渲染文字標籤**：依據範本替換數值，繪製深色半透明背景框以增強文字對比度，並疊加清晰的文字。
