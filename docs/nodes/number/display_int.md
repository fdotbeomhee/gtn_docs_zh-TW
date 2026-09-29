# DisplayInteger {: #displayinteger }

## 功能概述 {: #what-is-it }

DisplayInteger 節點專門用於在畫布中直接視覺化展示整數參數數值。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 當您希望在工作流程圖 (Graph) 中即時檢視或監控特定整數數值時

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 DisplayInteger 節點新增至工作流程中
1. 將任何其他上游節點的整數輸出引腳連線至此節點的輸入埠

### 參數說明 {: #parameters }

- **integer**：由本節點顯示的單一整數數值（預設值為 `0`）

### 輸出埠說明 {: #outputs }

- **無**（本節點為視覺展示端點節點，無輸出引腳）

## 範例展示 {: #example }

視覺化檢視工作流程中累計消耗的 Token 計數：

1. 將 DisplayInteger 節點新增至工作流程中
1. 將上游節點的 "token_count" 輸出引腳連線至 DisplayInteger 的輸入埠
1. DisplayInteger 節點將在圖形畫布中直觀呈現當前的 Token 計數整數值

## 重要注意事項 {: #important-notes }

- 本節點純粹用於視覺化監控與展示目的。
- 它不會對數值進行任何修改，僅忠實呈現接收到的內容。
- 非常適用於工作流程的除錯 (Debugging) 與即時狀態監控。

## 常見問題排解 {: #common-issues }

- 無特殊已知問題。
