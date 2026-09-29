# ResizeVideo {: #resizevideo }

## 功能概述 {: #what-is-it }

ResizeVideo 節點透過 FFmpeg 底層濾鏡，支援運用多種縮放模式（寬度、高度、百分比或精確寬高）彈性調整視訊解析度。依據您選取的適配模式 (Fit Mode)，節點可在縮放過程中精確維持原始長寬比、自動添加指定背景色的留黑邊 (Letterboxing) 或裁切填滿畫面。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用 ResizeVideo 節點：

- 依照精確的等比百分比縮放視訊畫面
- 在維持原生長寬比的前提下，鎖定目標寬度或目標高度進行縮放
- 透過填色留邊或裁切填滿，將視訊強制作為特定固定幾何規格
- 在跨平台視訊工作流程中建立一致標準化的視訊規格控制

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 ResizeVideo 節點新增至工作流程中
1. 將視訊來源連線至 "video" 輸入埠
1. 挑選目標縮放模式 (resize_mode)
1. 配置該模式相對應的尺寸數值與參數
1. 執行工作流程以完成視訊解析度調整

### 參數說明 {: #parameters }

- **video**：欲縮放的目標視訊素材（相容 VideoArtifact 與 VideoUrlArtifact 格式）

- **resize_mode**：計算視訊縮放的基準模式

    - `width`：鎖定目標寬度縮放，高度自動等比計算
    - `height`：鎖定目標高度縮放，寬度自動等比計算
    - `percentage`：依照原生尺寸的等比百分比進行縮放
    - `width and height`：縮放至精確的寬高解析度，配合適配選項

- **target_size**（像素，範圍 1-8000）：在 width 或 height 模式下的目標邊長像素值

- **percentage**（縮放百分比，範圍 1-500，預設值：100）：在 percentage 模式下的百分比數值

- **target_width**（像素）：在 width and height 模式下的目標寬度

- **target_height**（像素）：在 width and height 模式下的目標高度

- **fit_mode**：在 target_width 與 target_height 模式下的幾何適配規則

    - `fit`：維持原生長寬比，不足部分自動以背景色填充黑邊 (Letterbox/Pillarbox)
    - `fill`：維持原生長寬比，置中裁切超出部分以完全填滿目標畫布
    - `stretch`：忽略原生長寬比，直接強制拉伸變形以契合目標尺寸

- **background_color**（Hex 色碼）：在 fit 模式下用於填充留白邊界的背景底色

- **scaling_algorithm**：執行影像重採樣的插值演算法（預設值："bicubic"）

    - `neighbor`（最鄰近插值，像素風格）、`bilinear`（雙線性插值）、`bicubic`（雙三次插值）、`lanczos`（極致清晰 Lanczos 濾波）

- **lanczos_parameter**（範圍 1.0-10.0，預設值：3.0）：Lanczos 濾波演算法的微調半徑係數

    僅當 `scaling_algorithm` 選定為 "lanczos" 時此參數才會生效。

### 輸出埠說明 {: #outputs }

- **resized_video**：調整解析度完成後的視訊產物，可供後續節點繼續引用

## 範例展示 {: #example }

將任意比例視訊縮放適配至標準 1280x720 HD 畫布，並以純黑留邊保護比例：

1. 將 **resize_mode** 設定為 `width and height`
1. 將 **target_width** 設為 `1280`
1. 將 **target_height** 設為 `720`
1. 將 **fit_mode** 選為 `fit`
1. 將 **background_color** 設為 `#000000`

## 重要注意事項 {: #important-notes }

- ResizeVideo 節點核心依賴 FFmpeg 工業級縮放濾波引擎。
- **偶數像素自動校正**：為了確保多數視訊編碼器（如 H.264/H.265）的底層相容性，輸出的幾何寬高像素會自動微調至能被 2 整除的偶數值。
- 廣泛相容主流視訊格式（mp4、avi、mov 等）。
- **音訊軌道完美保留**：縮放過程中原始音訊軌道會被完整保留且絕不失真。
- 支援輸出詳細的處理日誌以供除錯與效能診斷。
