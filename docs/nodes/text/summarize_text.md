# Summarize Text {: #summarize-text }

## 功能概述 {: #what-is-it }

Summarize Text Task 工具節點是一個實用工具，專門用於為您自動提煉並摘要長篇文字的核心內容。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 建立摘要長篇文字內容的任務
- 定義具體的文字摘要需求與限制參數
- 建置結構化的文件自動摘要工作流程
- 管控大篇幅文字的重點凝練處理程序

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 Summarize Text Task 工具節點新增至工作流程中
1. 配置摘要相關參數
1. 將其輸出引腳連線至需要摘要成果的下游節點

### 參數說明 {: #parameters }

- **text**：欲進行提煉摘要的長篇文字內容

### 輸出埠說明 {: #outputs }

- **output**：產出之提煉文字重點摘要

## 範例展示 {: #example }

為一篇長篇文稿建立自動摘要任務：

1. 將 Summarize Text Task 工具節點新增至工作流程中
1. 將來源長篇文字連線至 "text" 輸入埠
1. 將 "output" 引腳連線至接收摘要結果的下游處理節點

## 實作細節 {: #implementation-details }

Summarize Text Task 工具能協助您快速提煉大量長篇文字的精華，顯著節省閱讀與資訊過濾時間。
