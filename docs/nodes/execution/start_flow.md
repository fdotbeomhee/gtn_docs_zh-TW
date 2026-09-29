# StartFlow {: #startflow }

## 功能概述 {: #what-is-it }

StartFlow 是一個特殊的基礎結構模組，專門用於標示工作流程的起始進入點。將它想像為一塊明確宣告「流程由此開始執行」的起跑標誌 (Go Sign)。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 欲為工作流程確立清晰、唯一的起點
- 欲啟動一連串循序相連的執行鏈
- 欲定義整個圖形工作流程的程式執行進入點 (Entry Point)

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 StartFlow 節點新增至工作流程畫布中
1. 將其連線至流程中的第一個具體動作節點

### 參數說明 {: #parameters }

*(本節點作為源頭進入點，無額外輸入參數)*

### 輸出埠說明 {: #outputs }

- **exec_out** — 控制流輸出引腳，用於將執行權限依序移交給流程中的下一個節點

## 範例展示 {: #example }

建立一個文字生成與自動儲存的工作流程：

1. 建立一個 StartFlow 節點
1. 將節點頂部白色的 "exec chain" 執行鏈引腳連線至負責生成文字的 Agent 節點
1. 將該 Agent 連線至 SaveText 節點以儲存文字成果
1. 將 SaveText 節點連線至 EndFlowNode 節點以完成閉環

StartFlow 明確告知引擎：「自此處開始，並循著執行鏈順序推進運算」。

## 重要注意事項 {: #important-notes }

- 每個連貫的工作流程通常需要且僅能具備一個 StartFlow 進入點。
- StartFlow 節點不接收任何前置輸入 — 它純粹作為執行序列的起點。
- 每個獨立的連續流程圖僅應配置一個 StartFlow。

## 常見問題排解 {: #common-issues }

- **流程未被觸發執行**：確認 StartFlow 的 `exec_out` 控制引腳已正確連線至下游的第一個活性節點。
- **存在多個起點**：請確認流程圖中沒有多餘的 StartFlow 造成調度邏輯混亂。
