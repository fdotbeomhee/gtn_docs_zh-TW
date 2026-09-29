# ApplyMask {: #applymask }

## 功能概述 {: #what-is-it }

ApplyMask 節點將遮罩 (Mask) 套用至輸入影像上以產生精確的透明度 (Transparency) 效果。它接收一張影像與一個遮罩，接著將該遮罩所指定的色彩通道作為最終影像的 Alpha 透明通道。您可以依據遮罩資料儲存的維度，自由指定要提取的通道（紅、綠、藍或 Alpha），藉此實現選擇性去背或保留影像的特定區域。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 運用遮罩為影像套用透明通道效果
- 為下游多圖層合成工作流程建立局部透明區域
- 配合 DisplayMask、PaintMask 或 InvertMask 所產生的遮罩修改透明度
- 快速摳圖 (Cutouts) 或消除影像背景
- 在影像處理工作流程中實作細緻的透明度混合遮罩
- 精準指定由遮罩影像的哪一個色彩通道驅動透明度

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 ApplyMask 節點新增至工作流程中
1. 將來源影像連線至 "input_image" 輸入埠
1. 將遮罩來源連線至 "input_mask" 輸入埠
1. 於 "channel" 參數中選取適當的通道來源
1. 處理完成的遮罩影像將於 "output" 輸出埠傳回

### 參數說明 {: #parameters }

- **input_image**：欲套用遮罩的來源影像（可由其他影像節點連線輸入）

- **input_mask**：欲套用的遮罩來源（可由 DisplayMask、PaintMask 或 InvertMask 等節點連線輸入）

- **channel**：從遮罩影像中提取作為透明度的指定通道：

    - **red**：使用紅色通道（預設值，與 PaintMask 輸出完全相容）
    - **green**：使用綠色通道
    - **blue**：使用藍色通道
    - **alpha**：使用 Alpha 通道（若來源遮罩包含 Alpha 資訊）

### 輸出埠說明 {: #outputs }

- **output**：套用遮罩透明度後的最終完成影像（透明通道依據遮罩資訊決定）

## 範例展示 {: #example }

常見的遮罩合成工作流程：

1. 使用 GenerateImage 生成影像或使用 LoadImage 載入照片
1. 使用 DisplayMask 或 PaintMask 繪製並生成遮罩
1. 將影像連線至 ApplyMask 節點的 "input_image" 參數埠
1. 將遮罩連線至 ApplyMask 節點的 "input_mask" 參數埠
1. 選擇相符的通道（若為 PaintMask 輸出通常選 "red"，若為 RGBA 遮罩選 "alpha"）
1. ApplyMask 節點將自動進行像素對齊與 Alpha 通道置換
1. 將 "output" 引腳連線至 DisplayImage 檢視帶有透明度的輸出成果

## 重要注意事項 {: #important-notes }

- **通道選取建議 (Channel Selection)**：
    - **PaintMask 節點輸出**：通常選用 "red" 通道
    - **具備透明度的 RGBA 遮罩**：選用 "alpha" 通道
    - **自訂特殊遮罩**：可切換不同通道檢視何者效果最佳
- **自動尺寸縮放 (Mask Processing)**：若遮罩解析度與來源影像不同，節點會自動等比重置遮罩尺寸以匹配來源影像。
- **RGBA 格式輸出**：最終影像一律統一轉換為 RGBA 格式以完整支援透明度。
- **透明度無損儲存**：帶有透明度的產物自動以無損 PNG 格式保存。

## 常見問題排解 {: #common-issues }

- **未產生任何輸出**：請檢查 "input_image" 與 "input_mask" 是否皆已正確接駁。
- **透明度顛倒非預期**：請記住：遮罩中**白色區域保持不透明，黑色區域完全變為透明**。若相反，請搭配使用 InvertMask 節點反轉。
- **通道不匹配**：若遮罩效果未生效，請嘗試切換 channel 為 red、green、blue 或 alpha。

## 技術實作細節 {: #technical-details }

節點內部依序執行以下處理步驟：

1. **影像與遮罩載入**：分別自記憶體或 URL 載入來源影像與遮罩產物。
1. **通道精準提取 (Channel Extraction)**：
    - RGB 影像：支援提取紅、綠、藍三通道中的任意一個。
    - RGBA 影像：支援提取紅、綠、藍或原生 Alpha 通道。
    - L 灰階影像：直接採用單通道灰階亮度。
1. **解析度匹配縮放**：將提取出的單通道陣列等比縮放至與原始輸入影像嚴格一致的像素寬高。
1. **透明通道置換**：運用 PIL 的 `putalpha()` 函式將單通道灰階資料寫入為目標影像的 Alpha 通道。
1. **產物發布**：將最終生成的 RGBA 影像發布為新產物輸出。
