# AddTextToExistingImage {: #addtexttoexistingimage }

## 功能概述 {: #what-is-it }

AddTextToExistingImage 節點允許您在現有影像上疊加渲染文字。它支援帶有透明通道 (Alpha) 的文字與背景色彩配置、對齊位置控制，以及透過獨立傳入的字典將 `{key}` 範本佔位符號動態替換展開。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 在影像上方添加標題、圖說說明文字或浮水印標籤
- 在影像上壓印中繼資料（例如檔案名稱、時間戳記或專案識別碼）
- 產生帶有註釋說明圖層的影像產物以供下游流程引用
- 透過字典輸入動態展開 `{key}` 佔位符以生成客製化文字影像

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 AddTextToExistingImage 節點新增至工作流程中
1. 將來源影像連線至 **input_image** 輸入埠
1. 於 **text** 參數中輸入文字範本內容
1. （選填）將字典連線至 **template_values** 以動態展開 `{key}` 佔位符
1. 調整對齊方式、文字/背景色彩、邊距與字型大小
1. 執行節點以由 **output** 輸出埠取得完成影像

### 參數說明 {: #parameters }

#### 必要輸入 (Required Inputs) {: #required-inputs }

- **input_image**：欲在其上渲染文字的目標影像（ImageUrlArtifact / ImageArtifact / dict）

#### 文字參數 (Text Inputs) {: #text-inputs }

- **text**（字串 string）：欲渲染的文字範本

    - 支援類似 `{key_name}` 的動態佔位符號
    - 佔位符的展開僅套用於影像上所繪製的畫面；輸出的 `text` 參數依然保留原始範本字串

- **template_values**（字典 dict，選填）：用於替換展開 `text` 中佔位符的鍵值字典

    - 若佔位符對應的鍵不存在於字典中，該佔位符將原樣保留在渲染文字中
    - 遺失的鍵名亦會記錄於執行後的 `result_details` 資訊中

#### 樣式外觀參數 (Styling Parameters) {: #styling-parameters }

- **text_color**（含 Alpha 的 Hexa 色碼，預設為 `#ffffffff`）：包含透明度的文字色彩
- **text_background**（含 Alpha 的 Hexa 色碼，預設為 `#000000ff`）：文字底層背景矩形框的色彩（含透明度）
- **text_vertical_alignment**（垂直對齊：top 靠上 | center 居中 | bottom 靠下，預設為 top）
- **text_horizontal_alignment**（水平對齊：left 靠左 | center 居中 | right 靠右，預設為 left）
- **margin**（整數 int，預設為 10）：文字區塊與影像邊界之間的內縮邊距像素
- **font_size**（整數 int，預設為 36）：渲染文字所使用的字型大小

### 輸出埠說明 {: #outputs }

- **output**：已完成文字疊加的影像產物 (ImageUrlArtifact)
- **text**：原始文字範本字串（保留原始範本，非替換後的展開字串）
- **was_successful**：指出節點是否執行成功的布林值
- **result_details**：執行成功或失敗的詳細除錯資訊（包含未找到的範本鍵訊息）

## 範例展示 {: #example }

為照片壓印標題與編號中繼資料：

1. 使用 LoadImage 節點載入照片影像

1. 新增 AddTextToExistingImage 節點至工作流程

1. 將 **text** 範本設定為：

    `"相片: {name}  |  編號: #{index}"`

1. 提供 **template_values** 字典：

    ```
    {"name": "肖像攝影", "index": 7}
    ```

1. 將 `text_background` 設定為半透明黑色：`#00000080`

1. 將 `text_color` 設定為白色：`#ffffffff`

1. 執行節點並將 **output** 連線至 DisplayImage 節點檢視成果

## 重要注意事項 {: #important-notes }

- **鍵值缺失處理 (Missing keys)**：若 `{key}` 不存在於 `template_values` 中，佔位符將直接原樣印出，並於 `result_details` 中記錄類似 `key: key not found in dictionary input` 的提示。
- **透明度通道支援 (Alpha support)**：文字與背景色彩皆完整支援 8 位元 Hexa 色碼（`#RRGGBBAA`）。
- **影像發布機制**：節點在執行 `process()` 時會自動處理影像處理與發布產物。
