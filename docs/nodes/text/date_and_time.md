# Date and Time {: #date-and-time }

## 功能概述 {: #what-is-it }

Date and Time 工具節點是一個實用工具，能協助您將日期與時間資訊格式化並操作轉換為純文字。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 將日期與時間格式化為特定的文字規格
- 在不同的日期/時間格式字串之間進行轉換
- 產生可在下游提示詞或文字中引用的時間戳記字串
- 構建基於時間維度的動態文字內容

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 Date and Time 工具節點新增至工作流程中
1. 設定日期與時間的相關參數
1. 將其輸出埠連線至需要已格式化日期/時間文字的下游節點

### 參數說明 {: #parameters }

- **prompt**：欲獲取日期的目標語意敘述（例如 "Christmas, 2027"、"Next friday"）
- **format**：日期/時間的目標文字格式（例如 `"%Y-%m-%d %H:%M:%S"`、`"Fri 25, March, 9pm"`）

### 輸出埠說明 {: #outputs }

- **output**：格式化後的日期/時間純文字輸出

## 範例展示 {: #example }

以自訂格式規範輸出目標日期字串：

1. 將 Date and Time 工具節點新增至工作流程中
1. 設定所需的日期時間格式字串
1. 將 "output" 引腳連線至需要該時間戳記的其他節點

## 實作細節 {: #implementation-details }

Date and Time 工具提供靈活的日期與時間文字格式化功能。它支援豐富的 strftime 格式規範與時區解析機制，精確產出符合預期的時間字串。
