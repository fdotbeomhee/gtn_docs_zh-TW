# ToolList {: #toollist }

## 功能概述 {: #what-is-it }

ToolList 節點將多個獨立的工具物件整合合併為單一工具清單。這使您能夠直接將多種多樣的工具打包並連線至接收 "tools" 參數的下游節點（例如 AI Agent）。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 將多個獨立工具組合成單一清單
- 構建能同時呼叫多維度專用工具的全能型 AI 代理工作流程
- 整合來自不同來源與模組的 Griptape 工具擴充

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 ToolList 節點新增至工作流程中
1. 將提供工具物件的各個上游節點連線至本節點的輸入埠
1. 將整合後的輸出引腳接駁至下游 Agent 節點

### 參數說明 {: #parameters }

- **tools**：欲合併為單一清單的工具物件集合

### 輸出埠說明 {: #outputs }

- **tool_list**：合併整合後的多工具物件清單

## 範例展示 {: #example }

為 Agent 同時掛載計算機、日期工具與網頁搜尋等多重工具：

1. 新增 Calculator、DateTimeTool 與 WebSearchTool 節點至工作流程
1. 新增 ToolList 節點，將上述三個工具節點的輸出分別連線至 ToolList
1. 將 ToolList 的 "tool_list" 輸出引腳連線至 Agent 節點的 "tools" 輸入埠
1. 該 Agent 即可在單一對話生命週期中自由切換運用多個工具

## 重要注意事項 {: #important-notes }

- 工作流程執行時，ToolList 必須接收到有效的輸入工具物件。
- 若傳入巢狀清單結構（Nested Lists），節點會自動將其展平 (Flattened) 為標準的單層工具清單。

## 常見問題排解 {: #common-issues }

- **無效的工具輸入 (Invalid Input Tools)**：請確保連線至本節點的物件為合法的 Griptape 工具實體。
- **巢狀清單行為 (Nested Lists)**：請注意傳入的任何多層清單結構都會被自動平鋪為單一層級。
