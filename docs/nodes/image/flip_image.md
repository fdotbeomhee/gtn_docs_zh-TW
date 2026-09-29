# FlipImage {: #flipimage }

## 功能概述 {: #what-is-it }

FlipImage 節點允許您在水平方向、垂直方向或同時在兩個維度對影像執行鏡射翻轉 (Flipping)。它是製作鏡面反射特效、校正相機鏡頭方向偏差，或為現有素材創造完美對稱構圖的實用工具。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 製作鏡面鏡射效果 (Mirror Effects) 或水面倒影
- 校正上下顛倒或鏡像錯誤的拍攝素材
- 建立具有高度對稱美感的視覺藝術構圖
- 為既有圖片生成左右翻轉的視覺變體 (Variations)
- 基於設計排版考量調整主體的視線朝向
- 快速生成同圖素材的上下/左右對比樣式

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 FlipImage 節點新增至工作流程中
1. 將影像來源連線至 "input_image" 輸入埠
1. 選取您期望的翻轉方向 (direction)
1. 翻轉後的影像產物將於 "output" 輸出埠傳回

### 參數說明 {: #parameters }

#### 翻轉設定 (Flip Settings) {: #flip-settings }

- **direction**：影像翻轉的方向選項
    - **horizontal**：水平翻轉（左右對調，鏡面鏡像效果）
    - **vertical**：垂直翻轉（上下顛倒）
    - **both**：水平與垂直雙向翻轉（等效於 180 度中心點旋轉）

### 輸出埠說明 {: #outputs }

- **output**：翻轉後的完成影像產物
- **was_successful**：指示翻轉操作是否順利完成的布林值
- **result_details**：關於此次翻轉處理的詳細狀態或除錯資訊

## 範例展示 {: #example }

製作人物肖像的左右鏡像鏡頭：

1. 使用 LoadImage 載入照片
1. 將照片連線至 FlipImage 節點的 "input_image" 參數埠
1. 設定翻轉方向：
    - 選取 "horizontal" 建立水平鏡像畫面
    - 或選取 "vertical" 顛倒上下方向
    - 或選取 "both" 旋轉 180 度
1. 將 "output" 引腳連線至 DisplayImage 檢視翻轉成果

## 重要注意事項 {: #important-notes }

- **即時預覽 (Live Preview)**：調整方向選項時，畫面會即時重新計算並更新展示。
- **無損畫質保留**：翻轉屬於純像素矩陣重排，不會引入任何有損壓縮或重採樣模糊。
- **RGBA 透明度完整保留**：帶有透明通道的 RGBA 影像在翻轉時會完整保留 Alpha 邊界。
- **狀態追蹤反饋**：節點提供完整的成功狀態與執行細節回傳。

## 常見問題排解 {: #common-issues }

- **未產生任何輸出**：請檢查 "input_image" 輸入埠是否已正確接駁有效影像。
- **翻轉方向混淆**："horizontal" 為左右翻轉（鏡子效果），"vertical" 為上下顛倒。
- **雙向翻轉效果**："both" 會同時套用水平與垂直兩道翻轉，在幾何上完全等同於 180 度旋轉。

## 技術實作細節 {: #technical-details }

節點底層呼叫 PIL (Pillow) 核心影像處理模組：

- **水平翻轉**：呼叫 `ImageOps.mirror()` 執行高效左右鏡射。
- **垂直翻轉**：呼叫 `ImageOps.flip()` 執行高效上下對調。
- **雙向翻轉**：按順序串聯套用上述兩項操作。
