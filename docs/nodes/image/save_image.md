# SaveImage {: #saveimage }

## 功能概述 {: #what-is-it }

SaveImage 節點允許您將處理後的影像儲存至本機檔案系統中。它是保留影像處理管線運算成果不可或缺的基礎節點，支援以多種格式規格與壓縮畫質匯出影像。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 將處理完成的影像儲存至電腦硬碟
- 以特定檔案格式（PNG、JPEG 等）匯出影像
- 精細掌控影像壓縮品質與檔案體積
- 長久保留影像處理工作流程的成果產物
- 為關鍵影像建立本機備份
- 匯出影像以供其他第三方應用程式或管線載入

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 SaveImage 節點新增至工作流程中
1. 將影像來源連線至 "input_image" 輸入埠
1. 設定輸出檔案路徑與目標格式
1. 影像即會自動儲存至指定的本機儲存位置

### 參數說明 {: #parameters }

#### 影像輸入 (Image Input) {: #image-input }

- **input_image**：欲儲存的來源影像（可由任何影像處理節點連線輸入）

#### 輸出設定 (Output Settings) {: #output-settings }

- **output_path**：影像儲存的本機目標檔案路徑

    - 可指定為相對路徑（例如 `"output/image.png"`）
    - 可指定為絕對路徑（例如 `"/Users/username/Desktop/image.png"` 或 `"C:/Users/username/Desktop/image.png"`）
    - 檔案副檔名將輔助決定最終格式

- **image_format**：儲存影像的格式規格

    - **PNG**：無損壓縮格式，完整支援透明度 (Alpha 通道)
    - **JPEG**：失真壓縮格式，檔案體積小
    - **BMP**：無壓縮點陣圖格式
    - **TIFF**：專業出版等級高畫質格式

#### 畫質設定 (Quality Settings，僅限 JPEG) {: #quality-settings }

- **quality**（範圍 1-100，預設值：95）：JPEG 壓縮品質
    - 數值越高 = 畫質越精緻，檔案體積越大
    - 數值越低 = 畫質壓縮越明顯，檔案體積越小

### 輸出埠說明 {: #outputs }

- **saved_path**：影像成功儲存的實際路徑
- **file_size**：已儲存檔案的位元組大小 (Bytes)

## 範例展示 {: #example }

標準的影像儲存工作流程：

1. 使用 AdjustImageEQ 或其他處理節點修潤影像
1. 將修潤後的影像連線至 SaveImage 節點的 "input_image" 參數埠
1. 設定輸出參數：
    - 將 output_path 設為 `"processed_image.png"`
    - 將 image_format 選為 "PNG" 以獲取無損最佳畫質
1. 影像將自動寫入您指定的磁碟路徑
1. "saved_path" 輸出埠將回報最終檔案位置

## 重要注意事項 {: #important-notes }

- **目錄自動建立**：請確保輸出目錄存在，若不存在，節點會自動建立相應資料夾。
- **格式特性差異**：不同格式具備不同特性（例如只有 PNG/TIFF 支援透明度，JPEG 不支援透明通道）。
- **畫質參數範圍**：quality 畫質設定僅對 JPEG 格式有效。
- **檔案覆寫機制**：若目標路徑已存在同名檔案，節點將直接覆寫該檔案。

## 常見問題排解 {: #common-issues }

- **檔案未成功儲存**：請確認 output_path 是否合法，並檢查目前執行環境是否具備該目錄的寫入權限。
- **格式不符合預期**：請確保副檔名與 image_format 參數選擇互相一致。
- **影像壓縮噪點明顯**：針對 JPEG 格式，可提高 quality 數值或直接切換為無損 PNG 格式。

## 技術實作細節 {: #technical-details }

本節點核心運用 PIL (Pillow) 底層儲存引擎：

- **PNG**：無損壓縮，保留完整色彩空間與透明邊緣
- **JPEG**：支援量化表設定的離散餘弦轉換 (DCT) 失真壓縮
- **BMP**：標準無壓縮點陣圖矩陣保存
- **TIFF**：工業級標籤圖像格式，相容多種色彩空間規格
