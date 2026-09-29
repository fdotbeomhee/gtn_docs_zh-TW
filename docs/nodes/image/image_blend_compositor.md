# ImageBlendCompositor {: #imageblendcompositor }

## 功能概述 {: #what-is-it }

ImageBlendCompositor 節點允許您使用各種類似專業修圖軟體圖層的混合模式 (Blend Modes)，將兩張影像深度合成在一起。它提供了對影像合成方式的極致精確控制，包括相對偏移位置、不透明度 (Opacity) 以及進階圖層合成選項。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 運用多種圖層混合模式將兩張影像融合
- 建立具備多圖層光影質感的複合影像
- 套用極具藝術創意的混合光學特效
- 以精準的像素偏移量疊加圖層
- 製作經典的雙重曝光 (Double Exposure) 攝影效果
- 將材質紋理 (Textures) 或重複圖樣融入底圖
- 構建精巧的數位拼貼與視覺藝術創作

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 ImageBlendCompositor 節點新增至工作流程中
1. 將兩個影像來源分別連線至 "input_image"（底圖背景層）與 "blend_image"（混合前景層）輸入埠
1. 挑選混合模式 (blend_mode) 並調整不透明度 (opacity) 滑桿
1. 最終合成的影像產物將於 "output" 輸出埠傳回

### 參數說明 {: #parameters }

#### 影像輸入 (Image Inputs) {: #image-inputs }

- **input_image**：底圖影像（背景圖層 Background Layer）
- **blend_image**：疊加在上方參與混合的影像（前景圖層 Foreground Layer）

#### 混合設定 (Blend Settings) {: #blend-settings }

- **blend_mode**：欲採用的圖層混合演算法模式

    - **normal**：標準正常圖層混合（基於 Alpha 透明度覆蓋）
    - **multiply**：色彩增值 / 正片疊底（使底圖變暗，常用於陰影合成）
    - **screen**：濾色（使底圖變亮，常用於光效合成）
    - **overlay**：覆蓋（結合色彩增值與濾色，增強明暗對比）
    - **soft_light**：柔光（溫和自然的光影明暗渲染）
    - **hard_light**：強光（強烈戲劇化的明暗反差疊加）
    - **color_dodge**：顏色減淡（極致提亮底圖高光）
    - **color_burn**：顏色加深（加深底圖陰影飽和度）
    - **darken**：變暗（僅保留較暗的像素）
    - **lighten**：變亮（僅保留較亮的像素）
    - **difference**：差集（像素色彩相減，產生反色與邊緣輪廓感）
    - **exclusion**：排除（類似差集但對比更柔和）

- **opacity**（範圍 0.0-1.0，預設值：1.0）：前景混合層的不透明度強度

    - 0.0 = 完全透明（毫無混合效果，只顯示底圖）
    - 1.0 = 完全不透明（100% 完整套用混合）

#### 位置偏移 (Positioning) {: #positioning }

- **x_offset**（像素，預設值：0）：前景混合圖在水平方向的像素偏移量
- **y_offset**（像素，預設值：0）：前景混合圖在垂直方向的像素偏移量

#### 進階選項 (Advanced Options) {: #advanced-options }

- **resize_blend_to_fit**（布林值，預設為 `false`）：自動將前景圖等比縮放以嚴格貼合底圖尺寸
- **preserve_alpha**（布林值，預設為 `true`）：在合成結果中完整保留底圖與前景的原生 Alpha 透明度
- **invert_blend**（布林值，預設為 `false`）：在進行混合運算前先將前景圖執行色彩反向 (Invert)

### 輸出埠說明 {: #outputs }

- **output**：兩張影像精確混合合成後的完成影像產物

## 範例展示 {: #example }

為照片疊加復古紙張紋理：

1. 使用 LoadImage 載入主體照片並連線至 "input_image"（底圖）
1. 使用另一個 LoadImage 載入紙張紋理並連線至 "blend_image"（前景圖）
1. 配置混合參數：
    - 將 blend_mode 選為 "multiply" 以保留紙張紋理的暗部褶皺
    - 將 opacity 不透明度調至 0.7 營造自然低調的紋理感
    - 勾選 resize_blend_to_fit 確保紋理完全覆蓋照片
1. 將 "output" 引腳連線至 DisplayImage 檢視成果

## 重要注意事項 {: #important-notes }

- **即時響應 (Live Preview)**：調整混合模式或位移時，畫布會即時重新運算渲染。
- **跨邊界擺放**：前景圖允許透過 x_offset 與 y_offset 偏移至底圖邊界之外，溢出部分會被安全裁剪。
- **RGBA 透明度完整相容**：節點支援全通道 Alpha 透明度，透空區域不會產生黑邊噪點。

## 常見問題排解 {: #common-issues }

- **看不到混合效果**：確認 opacity 大於 0，且若選取 normal 模式請留意前景是否完全遮蓋了底圖。
- **前景圖位置偏差**：檢查 x_offset 與 y_offset 是否將前景圖移出了可視邊界。
- **混合效果與預期不符**：不同混合模式的數學運算邏輯截然不同（例如 Screen 用於加光，Multiply 用於加深），建議切換嘗試。

## 技術實作細節 {: #technical-details }

節點全面實作了工業標準的數位影像合成公式：

- **代數疊加混合**：Multiply ($A \times B$)、Screen ($1 - (1-A)(1-B)$)、Overlay、Soft Light、Hard Light。
- **色彩空間極值運算**：Color Dodge、Color Burn、Difference ($|A - B|$)、Exclusion。
- **門檻選拔混合**：Darken ($\min(A, B)$)、Lighten ($\max(A, B)$)。
- **標準 Alpha 覆蓋**：遵循 Porter-Duff 影像合成規範。
