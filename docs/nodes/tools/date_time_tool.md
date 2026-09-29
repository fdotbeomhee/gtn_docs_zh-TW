# Date Time {: #date-time }

## 功能概述 {: #what-is-it }

Date Time 工具節點是一個時間處理工具，可掛載指派給 AI Agent 代理，協助模型精準取得當前時間、進行時間跨度換算及日期運算。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 賦予 AI Agent 認知與操作日期時間的能力
- 執行複雜的日期/時間計算與排程推演
- 取得系統當前的精確時間資訊
- 支援以多種格式解析與轉換日期時間字串

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 Date Time 工具節點新增至工作流程中
1. 將其輸出引腳連線至需要時間感知與處理能力的節點（例如 Agent 節點的 tools 輸入埠）

### 輸出埠說明 {: #outputs }

- **tool**：配置完成的日期時間工具物件 (Tool Object)，可直接供其他節點引用

## 範例展示 {: #example }

建立一個能處理排程預約與時間計算的智慧 Agent：

1. 將 Date Time 工具節點新增至工作流程中
1. 將 "tool" 輸出埠連線至 Agent 節點的 "tools" 輸入埠
1. 該 Agent 隨即具備在與使用者對話時自主查詢目前時間與推算未來日期的能力

## 實作細節 {: #implementation-details }

Date Time 工具底層基於 Griptape 的 `DateTimeTool` 類別封裝，提供全方位的時間運算與時區解析介面。此工具專為 AI 代理設計，能精準管理時間序列資料並解決時間推理問題。
