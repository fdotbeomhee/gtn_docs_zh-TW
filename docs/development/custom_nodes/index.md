# 節點開發指南 {: #developing-nodes }

本章節為開發 Griptape Nodes 自訂節點的開發者提供了完整詳盡的技術文件。

!!! tip "致 AI 助理與程式碼生成智慧體 (AI Coding Agents)"

    本章節中的所有技術文件皆提供專為 AI 程式碼輔助工具後處理的 Markdown 格式。本站點公開了完整的機器可讀介面；索引請參閱 [智慧體使用指南](../../for_agents.md)。

    - **快速上手**：[Markdown](https://docs.griptapenodes.com/en/stable/development/custom_nodes/getting_started/index.md)
    - **概述** (本頁面)：[Markdown](https://docs.griptapenodes.com/en/stable/development/custom_nodes/index.md)
    - **範例程式碼**：[檢視 Python 範例](https://raw.githubusercontent.com/griptape-ai/griptape-nodes/main/docs/development/custom_nodes/example_control_node.py)

    **調用語法：** 將您的 AI 助理指向這些 URL 並下達如下提示詞：
    `"請閱讀此節點開發指南：[URL]，並協助我建構自訂節點"`

## 簡介 {: #introduction }

Griptape Nodes 是一套模組化的工作流程組件系統，賦予使用者透過視覺化節點程式設計構建複雜 AI 流程的能力。本章節涵蓋了打造強健、易用的自訂節點所需的核心基礎概念與進階設計模式。

若您是首次開發節點，建議從 [快速上手指南](getting_started.md) 開始閱讀。該指南以初學者友善的方式介紹節點開發體系，並帶領您動手打造第一個節點。

所有節點皆繼承自 `BaseNode` 的專門衍生類別：

- **DataNode**：負責純數據處理與同步轉換任務
- **ControlNode**：具備 `exec_in` / `exec_out` 執行控制連線的流程控制節點
- **StartNode**：工作流程的起點節點
- **EndNode**：工作流程的終點節點

## 核心概念 {: #core-concepts }

### 基底類別 (Base Classes) {: #base-classes }

- **DataNode**：在不依賴顯式執行流程控制的情況下處理資料。適用於同步轉換或傳遞資料的節點——當所有必要輸入端皆連線滿足時，節點會立即進行計算解析。
- **ControlNode**：透過 `exec_in` / `exec_out` 連線嚴格管控執行時序。適用於發送外部 API 請求或執行耗時長任務的節點——可覆寫 `async def aprocess()` 處理非同步任務，或利用 `AsyncResult` 將阻斷性運算委派至背景執行緒。若您的節點需要調用遠端 API 並輪詢結果，則**必須**宣告為 ControlNode。
- **StartNode**：工作流程的執行入口。
- **EndNode**：工作流程的終結回報點。

### 參數 (Parameters) {: #parameters }

透過 `Parameter` 類別定義節點的輸入端、輸出端與自訂屬性。參數系統原生支援：

- 嚴格型別驗證
- UI 介面自訂呈現
- 拓撲連線約束規則
- 預設值設定
- 視覺化特徵標籤 (Traits，如 Options 下拉選單、Slider 滑桿、Button 按鈕、ColorPicker 顏色選擇器)

完整規格請參閱 [參數參考指南](parameters.md)。

### 運算方法 (Process Method) {: #process-method }

`process()` 方法封裝了節點的核心業務邏輯。運算完成後，請將輸出結果賦值至 `self.parameter_output_values` 字典中。針對非同步運算，請改為覆寫 `async def aprocess()`——詳見 [執行與生命週期](execution_and_lifecycle.md)。

### 節點生命週期狀態 (Node States) {: #node-states }

- **UNRESOLVED**：未解析初始狀態
- **RESOLVING**：運算處理中
- **RESOLVED**：解析運算圓滿完成

### 拓撲連線 (Connections) {: #connections }

透過生命週期回呼函式進行拓撲連線的即時驗證與動態響應。詳見 [執行與生命週期](execution_and_lifecycle.md)。

### 事件監聽 (Events) {: #events }

覆寫 `on_griptape_event` 可對工作流程運行過程中的廣播事件做出精確響應。

## 開發環境準備 {: #setting-up }

1. 安裝 `griptape-nodes` 套件。
1. 使用 Python 虛擬環境 (`venv`) 確保環境隔離。
1. 以簡潔清晰的目錄階層規劃專案組織。
1. 自 `griptape_nodes.exe_types.*` 與 `griptape_nodes_library.utils.*` 匯入核心型別。

## 建立新節點 {: #creating-a-node }

### 基礎節點架構範例 {: #basic-node-structure }

```python
from typing import Any
from griptape_nodes.exe_types.core_types import Parameter, ParameterMode
from griptape_nodes.exe_types.node_types import DataNode


class MyNode(DataNode):
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)
        self.category = "Category"
        self.description = "Description"

        self.add_parameter(Parameter(name="input", input_types=["str"], type="str", tooltip="輸入參數"))

        self.add_parameter(Parameter(name="output", output_type="str", tooltip="輸出參數"))

    def process(self) -> None:
        val = self.get_parameter_value("input").upper()
        self.parameter_output_values["output"] = val
```

## 章節架構導覽 {: #documentation-structure }

- **[快速上手](getting_started.md)** —— 初學者友善的手把手教學，帶您建構第一個節點
- **[參數](parameters.md)** —— 參數屬性、特徵標籤 (Traits)、輔助類別、容器型別與進階參數模式
- **[參數 UI 參考](parameter_ui_reference.md)** —— 參數型別至 UI 部件的對應關係、支援的 `ui_options` 設定鍵與特徵
- **[執行與生命週期](execution_and_lifecycle.md)** —— 生命週期回呼函式與非同步 API 整合模式
- **[專案系統](project_system.md)** —— 透過情境 (Situations)、巨集 (Macros) 與 `ProjectFileParameter` 進行檔案持久化儲存
- **[最佳實踐與錯誤處理](error_handling.md)** —— 憑據金鑰管理、動態匯入、參數負載體積限制、驗證、例外處理與記錄
- **[編寫節點程式庫](authoring_libraries.md)** —— 程式庫資訊清單 (Manifest)、型別宣告、依賴管理與貢獻至標準庫
- **[高級程式庫](advanced_libraries.md)** —— `AdvancedNodeLibrary` 生命週期掛鉤、程式庫自屬請求處理常式與動態節點註冊
- **[自訂部件 (Widgets)](custom_widgets.md)** —— 自訂 JavaScript 介面部件組件與部件測試環境 (Testbed)
- **[模式與範例](examples.md)** —— 生產級節點的進階設計架構與快速參考錦囊
- **[工作處理程序隔離](node_isolation_with_workers.md)** —— 在獨立 Worker 子處理程序中隔離執行節點程式庫
- **[嚴格模式參考](strict_mode.md)** —— 診斷跨處理程序隔離相容性問題的嚴格模式檢查規則
- **[範例控制節點](example_control_node.py)** —— 完整展示控制節點最佳實踐的參考實作範例

## 從範本存放庫開始 {: #start-from-the-template-repository }

邁向生產級節點程式庫最迅速的捷徑是使用官方範本存放庫：

[Griptape Nodes Library Template](https://github.com/griptape-ai/griptape-nodes-library-template/) ([讀我檔案](https://github.com/griptape-ai/griptape-nodes-library-template/blob/main/README.md))

該範本提供了一切開箱即用的基礎結構程式碼、單元測試框架以及文件組織範式。標準開發流程如下：

1. **套用範本存放庫**——從 GitHub 範本建立您專屬的專案庫。
1. **設定本機環境**——將專案 Clone 至您的 Griptape Nodes 工作區目錄中。
1. **配置程式庫元資料**——重新命名目錄結構並更新 `pyproject.toml` 中的套件描述。
1. **建立您的節點**——定義節點類別（繼承 ControlNode 或 DataNode）並定義相應參數。
1. **實作業務邏輯**——編寫核心 `process()` 方法與輔助輔助函式。
1. **配置程式庫中繼資料**——在 `library.json` 檔案中登錄節點型別與分類資訊。
1. **向引擎完成註冊**——在 Griptape Nodes 的設定介面中新增並啟用您的程式庫。
1. **測試與驗證**——在 Griptape Nodes 視覺化畫布中拖曳節點並組裝流程進行實測。

若想深入汲取節點設計靈感，歡迎探索標準庫所採用的架構模式：[標準節點參考指南](../../nodes/overview.md)。

## 快速連結 {: #quick-links }

- **自訂腳本**：[保留模式腳本撰寫](../retained_mode.md) —— 使用 Python 實現引擎自動化與批次操作
