# DisplayDictionary 節點 {: #displaydictionary-node }

## 功能概述 {: #what-is-it }

`DisplayDictionary` 節點是一個實用工具節點，專門用於在工作流程畫布中視覺化呈現字典 (Dictionary) 的內部內容。它提供直觀清晰的資料結構檢視面板，讓您在工作流程執行與除錯期間能即時檢查字典的具體數值。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用 DisplayDictionary 節點：

- 欲在工作流程畫布中即時檢查某個字典的具體內容
- 欲將複雜的巢狀資料結構視覺化以輔助除錯排錯
- 欲持續監控字典變數在工作流程各處理階段的動態演變
- 處理類似 JSON 的結構化數據並希望清晰確認其鍵值對層次

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 DisplayDictionary 節點新增至工作流程中
1. 將任何輸出字典的節點連線至此節點的 "dictionary" 輸入埠
1. 字典的結構化內容將會自動格式化並呈現於節點的介面檢視區塊中

### 參數說明 {: #parameters }

- **dictionary**：欲檢視與展示的來源字典物件（同時具備直通傳遞特性）
- **dictionary_display**：字典內容的排版字串表示形式（屬性參數，展示於 UI 介面中）

### 輸出埠說明 {: #outputs }

- **dictionary**：輸入的原始字典物件原樣直通輸出 (Pass-through)，便於繼續向下傳遞

## 範例展示 {: #example }

在使用者資料處理流程中進行中繼除錯：

1. 將 DisplayDictionary 節點新增至畫布中
1. 將產出使用者資料的節點輸出（例如 API 回應或結構化建構節點）連線至 "dictionary" 輸入埠
1. DisplayDictionary 節點將以易於閱讀的多行排版即時呈現該資料結構
1. 您可順勢將 "dictionary" 輸出埠連線至後續需要該資料的其他業務節點，毫不中斷資料流

## 重要注意事項 {: #important-notes }

- DisplayDictionary 節點完全不會更動字典內部資料，純粹作為直通與展示之用。
- 節點介面內建多行文字展示區域，即使面對深度巢狀的字典結構亦能清晰閱讀。
- 此節點是在工作流程建置期間除錯與觀測資料流變化的絕佳輔助利器。

## 常見問題排解 {: #common-issues }

- 無特殊已知錯誤。
