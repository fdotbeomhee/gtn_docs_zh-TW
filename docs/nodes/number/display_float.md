# DisplayFloat {: #displayfloat }

## 功能概述 {: #what-is-it }

DisplayFloat 節點專門用於在畫布中直接視覺化展示浮點數參數數值。

## 適用情境 {: #when-would-i-use-it }

當您希望在工作流程圖 (Graph) 中即時檢視或監控特定浮點數數值時使用。

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 DisplayFloat 節點新增至工作流程中
1. 將任何其他上游節點的浮點數輸出引腳連線至此節點的輸入埠

### 參數說明 {: #parameters }

- **float**：由本節點顯示的單一浮點數數值（預設值為 `0.0`）

### 輸出埠說明 {: #outputs }

- **無**（本節點為視覺展示端點節點，無輸出引腳）

## 範例展示 {: #example }

視覺化檢視工作流程中設定的採樣溫度 (Temperature) 數值：

1. 將 DisplayFloat 節點新增至工作流程中
1. 將上游節點的 "temperature" 輸出引腳連線至 DisplayFloat 的輸入埠
1. DisplayFloat 節點將在圖形畫布中直觀呈現當前的溫度浮點數數值

## 重要注意事項 {: #important-notes }

- 本節點純粹用於視覺化監控與展示目的。
- 它不會對數值進行任何修改，僅忠實呈現接收到的內容。
- 非常適用於工作流程的除錯 (Debugging) 與即時狀態監控。

## 常見問題排解 {: #common-issues }

- 無特殊已知問題。
